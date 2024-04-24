---
tags:
  - cloud
  - gcp
  - terraform
slug: /Cloud/gce-cos-terraform-part2
title: 在Google Compute Engine上用 container 部署Elasticsearch (下)
---
## 前言

在先前的文章中，我們已經詳細介紹了如何在Google Compute Engine（GCE）上使用Container-Optimized OS和Terraform部署Elasticsearch
我們成功配置了虛擬機和相關資源，並探討了基本的設置過程
然而，實際運營中經常會遇到一些技術挑戰，尤其是與外部硬碟掛載和權限管理相關的問題

在這篇文章中，我們將深入探討在使用Container-Optimized OS進行Elasticsearch部署時遇到的一個具體問題：磁碟權限被不當覆蓋

## 遇到的問題與初步解決嘗試

- 在將terraform 的設定 apply 到雲端環境後，Elasticsearch在啟動階段的最後會出現無法寫入數據的錯誤訊息導致 container 無法正確啟動
	- 既然 disk 是額外掛載的方式掛到機器上，因此高機率可以從 `/mnt` 找到硬碟來排查錯誤
	- 查看後知道 Container-Optmized OS 在指定外接硬碟作為 volume 的時候會將硬碟 mount 至 `/mnt/disks/gce-containers-mounts/gce-persistent-disks/${disk_name}`
- 空白磁碟被掛載到VM上時是root權限
	- Elasticsearch的image已經寫死用戶是`elasticsearch`（UID為1000）
	- 在Container-Optimized OS中，UID 1000已被其他用戶占用，且磁碟的預設權限為755，這讓Elasticsearch無法正常寫入數據
- 在思考解決辦法的途中，找到了GCE提供了撰寫[啟動腳本](https://cloud.google.com/compute/docs/instances/startup-scripts/linux#passing-directly)的辦法
	- 首先嘗試使用GCE的startup script來進行`chown`，但發現這個操作會在Elasticsearch容器啟動過程中被覆蓋。於是我採用了`chmod a+w`來嘗試賦予更廣泛的寫入權限，但這同樣被後續操作覆蓋
	- 由於權限會被固定在 owner 為 `root` 且權限為 `755` 的情形下，無法透過將 UID 1000 的使用者加入 `group` 來解決

## 深入系統日誌與源碼分析

- 要解決Elasticsearch的部署問題，一個關鍵步驟是理解Volume掛載過程及其權限如何被設置
	- 因此需要深入系統的日誌和相關服務的 source code，才能確認何處出了問題以及如何修復
### 日誌分析：確認Startup Script的執行者

- 透過`journalctl`檢視系統日誌
	- 發現日誌中有名為 `cloud-init` 的 service unit，從名稱上來看就跟雲端系統啟動有關
	- 推測startup script應該是由`cloud-init`這個service執行的
	- 為進一步驗證這一點，我執行了以下命令以列出所有服務
		- `sudo systemctl list-units --type="service"`

```
cloud-config.service     loaded active exited  Apply the settings specified in cloud-config
cloud-final.service      loaded active exited  Execute cloud user/final scripts
cloud-init-local.service loaded active exited  Initial cloud-init job (pre-networking)
cloud-init.service       loaded active exited  Initial cloud-init job (metadata service crawler)
containerd.service       loaded active running containerd container runtime
dbus.service             loaded active running D-Bus System Message Bus
```

找到了`cloud-final.service`，其狀態顯示為`loaded active exited`，並且描述為`Execute cloud user/final scripts`
這確認了我的猜測：startup script是由此service負責執行的

### 從日誌追蹤到Konlet的行動

接著，我仔細查看了`cloud-final`執行完成後的日誌記錄：
```
Apr 21 09:19:44 shareclass-tw-elasticsearch-dev systemd[1]: Finished cloud-config.service.
Apr 21 09:19:44 shareclass-tw-elasticsearch-dev systemd[1]: Starting cloud-final.service...
Apr 21 09:19:45 shareclass-tw-elasticsearch-dev konlet-startup[694]: d2519f41f710: Pull complete
Apr 21 09:19:45 shareclass-tw-elasticsearch-dev cloud-init[742]: Cloud-init v. 23.2.1 running 'modules:final' at Sun, 21 Apr 2024 09:19:45 +0000. Up 14.74 seconds.
Apr 21 09:19:45 shareclass-tw-elasticsearch-dev cloud-init[742]: Cloud-init v. 23.2.1 finished at Sun, 21 Apr 2024 09:19:45 +0000. Datasource DataSourceGCE.  Up 15.05 seconds
# Skip...
Apr 21 09:20:23 shareclass-tw-elasticsearch-dev konlet-startup[694]: 2024/04/21 09:20:17 Creating directory /mnt/disks/gce-containers-mounts/gce-persistent-disks/volume-shareclass-tw-elasticsearc>
Apr 21 09:20:23 shareclass-tw-elasticsearch-dev konlet-startup[694]: 2024/04/21 09:20:17 Attempting to mount device /dev/disk/by-id/google-volume-shareclass-tw-elasticsearch-dev at /mnt/disks/gce>
Apr 21 09:20:23 shareclass-tw-elasticsearch-dev konlet-startup[694]: 2024/04/21 09:20:17 Created a container with name 'klt-shareclass-tw-elasticsearch-dev-bqub' and ID: c03e7f32d59e88148f0359580>
Apr 21 09:20:23 shareclass-tw-elasticsearch-dev konlet-startup[694]: 2024/04/21 09:20:17 Starting a container with ID: c03e7f32d59e88148f035958082beb25946b7c4d9a9c2c36cbdf9ff72f300902
Apr 21 09:20:23 shareclass-tw-elasticsearch-dev konlet-startup[694]: 2024/04/21 09:20:17 Saving welcome script to profile.d
```

- 日誌中的`Creating directory /mnt/disks/gce-containers-mounts/gce-persistent-disks/volume-shareclass-tw-elasticsearch`可以看到，`konlet-startup`是處理掛載邏輯的主要元件
- 透過 `sudo systemctl status konlet-startup.service`，追查到 service 文件位置
```
○ konlet-startup.service - Containers on GCE Setup
     Loaded: loaded (/lib/systemd/system/konlet-startup.service; disabled; preset: disabled)
     Active: inactive (dead) since Sun 2024-04-21 09:20:17 UTC; 2 days ago
   Duration: 37.532s
    Process: 674 ExecStart=/usr/share/gce-containers/konlet-startup (code=exited, status=0/SUCCESS)
   Main PID: 674 (code=exited, status=0/SUCCESS)
        CPU: 151ms
```
- 由 `/usr/share/gce-containers/konlet-startup`
	- 這個service會啟動 `gcr.io/gce-containers/konlet` 的 container
	- 啟動參數 `-v=/var/run/docker.sock:/var/run/docker.sock` 代表這個 service 需要在 docker 環境內取得 docker 的控制權，推測這個 service 會負責啟動 container 的所有相關事宜

### 查看 source code：確定掛載和權限設置

在`konlet`的[GitHub](https://github.com/GoogleCloudPlatform/konlet/blob/master/gce-containers-startup/volumes/volumes.go)中，我找到了以下重要的代碼片段，這幫助我了解到Volume掛載點的創建和權限設置是如何進行的

```go
func (env Env) createNewMountPath(volumeFamily string, volumeName string) (string, error) {
    path := fmt.Sprintf("%s/%ss/%s", *mountedVolumesPathPrefixFlag, volumeFamily, volumeName)
    log.Printf("Creating directory %s as a mount point for volume %s.", path, volumeName)
    if err := env.OsCommandRunner.MkdirAll(path, 0755); err != nil {
        return "", fmt.Errorf("Failed to create directory %s: %s", path, err)
    } else {
        return path, nil
    }
}
```
- 這段代碼揭示了Volume掛載點是如何被創建的
- 並且確認了每次掛載時都會設置0755的權限，這是我之前設置權限被覆蓋的原因

## 最終解決方案

為確保權限設置不被覆蓋，我設計了一個新的service
- 在startup script中實作
- 在 `cloud-final` 階段建立一個 service 的必要檔案，並啟動
- 這個service會等待 volume 成功掛載後，立即調整必要的權限
:::info  
由於 Container-Optimized OS，會將 `/usr/local` 設為 **read-only**，因此使用 `/etc`作為路徑
:::

```bash
#!/bin/bash
# 創建執行腳本
mkdir -p /etc/bin
echo "#!/bin/bash

while ! findmnt -M \"$MOUNT_PATH\" >/dev/null; do
  echo \$(date) - Waiting for $MOUNT_PATH to be mounted...
  sleep 2
done

echo \$(date) - Mount detected, changing permissions...
chmod a+w \"$MOUNT_PATH\"
echo \$(date) - Permissions changed.

echo \$(date) - Script completed.
exit 0
" > /etc/bin/startup_script.sh
chmod +x /etc/bin/startup_script.sh

# 創建systemd service文件
echo "[Unit]
Description=Wait for disk mount and adjust permissions
After=network-online.target

[Service]
Type=simple
ExecStart=/etc/bin/startup_script.sh
Restart=on-failure
RestartSec=30
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
" > /etc/systemd/system/wait-and-chmod.service

# 啟動並啟用service
systemctl enable wait-and-chmod.service
systemctl start wait-and-chmod.service
```
這樣，當磁碟被掛載且Docker容器準備就緒時，權限會被正確設定，允許Elasticsearch順利啟動並運行

## 結語

透過這次經驗，不僅解決了具體的技術問題，也對GCE的運作機制有了更深的理解
希望這篇分享能有所幫助，尤其是在處理雲端服務時遇到類似挑戰的情況
