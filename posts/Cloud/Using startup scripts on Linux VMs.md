---
tags:
  - gcp
  - cloud
slug: /cloud/using-startup-scripts-on-linux-vms
---
在 Google Cloud Platform (GCP) 上，使用 Linux 虛擬機（VM）的啟動腳本可以自動執行各種任務，如軟件安裝、環境設置等，從而簡化和自動化 VM 的配置過程

## Startup Script 的定義與功能

啟動腳本是在虛擬機實例啟動過程中自動執行的一系列指令。這些腳本通常是以 shell 腳本的形式存在，可以進行軟件安裝、執行腳本、修改配置文件等任務。其主要的優勢在於能夠在 VM 啟動時自動化設置過程，節省手動配置的時間與精力

## Startup Script 的執行條件

- **網絡連接**：需要在 VM 的網絡接口可用時執行，這是因為許多腳本任務，如軟件包更新或安裝，需訪問外部網絡資源
- **元數據覆蓋**：在 GCP 中，啟動腳本可以配置在 project 級別或 VM 級別。當兩者都被設置時，VM 級別的 metadata 會覆蓋項目級別的設置，使得在具體 VM 上可以有針對性的不同配置

## Startup Script 的設置方法

### Passing a Linux startup script directly

在使用 `gcloud compute instances create` 命令創建 VM 時直接指定啟動腳本
這個方法適合在自動化腳本或手動創建單個實例時使用
```bash
gcloud compute instances create [INSTANCE_NAME] \
  --metadata=startup-script='#!/bin/bash
  echo "Running startup script."
  sudo apt-get update && sudo apt-get install -y nginx'
```

### Passing a Linux startup script from a local file

若要從本地文件傳遞 Linux 啟動腳本到 Google Cloud VM，可以運用 `gcloud` 命令行工具來實現這一過程，這種方式非常適合已經有一個預先編寫好的腳本文件，希望在創建 VM 時自動執行它

1. **準備 startup script  文件**
```bash
#!/bin/bash

apt-get update
apt-get install -y nginx

systemctl enable nginx
systemctl start nginx
```

2. 使用 gcloud 命令中的 `--metadata-from-file` 上傳腳本
```bash
gcloud compute instances create "my-vm" \
	--image-project="debian-cloud" \
	--image-family="debian-10" \
	--machine-type="n1-standard-1" \
	--metadata-from-file startup-script=startup-script.sh
```

### Passing a Linux startup script from Cloud Storage

除了直接在 cli 中插入或從 local file 傳遞啟動腳本，還可以使用來自 Google Cloud Storage(GCS) 的startup script 來初始化 Linux VM
- 這種方法特別適合於需要大量自動化部署的情境
- 集中管理腳本並確保所有 VM 使用的是最新的配置

1. 準備 script
```bash
#! /bin/bash

apt-get update
apt-get install -y nginx

echo "<html><body><h1>Welcome to My Server on GCP!</h1></body></html>" > /var/www/html/index.html

```
2. 將 script 上傳至 GCS
```bash
gsutil cp startup-script.sh gs://your-bucket-name/
```

3. 在創建 VM 時指定啟動腳本 URL
```bash
gcloud compute instances create "my-vm" \
  --image-family="debian-10" \
  --image-project="debian-cloud" \
  --machine-type="n1-standard-1" \
  --metadata startup-script-url=gs://your-bucket-name/startup-script.sh

```

