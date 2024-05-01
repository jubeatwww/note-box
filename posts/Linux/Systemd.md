---
tags:
  - linux
slug: /linux/systemd
---
> [systemd.unit(5)](https://man7.org/linux/man-pages/man5/systemd.unit.5.html) => Linux manual page
> [systemd.service(5)](https://man7.org/linux/man-pages/man5/systemd.service.5.html) ⇒ Linux manual page

---

**`systemd`** 是 Linux 系統上的一個初始化系統和系統管理守護程序 (daemon)，它被設計來用作系統和服務管理器

systemd的 process id 為 1

## **主要功能**

- **服務管理**: systemd 使得啟動、停止、重啟和管理系統服務變得更加容易

- **並行啟動**: 它能夠並行地啟動服務，加快啟動時間

- **依賴性管理**: 自動解析服務之間的依賴關係，只有在必要的前置條件滿足後，才會啟動某個服務

- **日誌管理**: 通過 journalctl 工具集成日誌管理，提供一個統一的日誌管理解決方案

## 建立範例 service

```plain
[Unit]
Description=Example Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /path/to/your/script.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 常用 Units 選項

### **\[Unit\] 選項**

用來描述如何啟動和管理服務、裝置、掛載點等

- **Description**: 簡短描述該 unit 的功能

- **Documentation**: 提供該 unit 相關文檔的 url

- **Wants**: 弱依賴，當該 unit 被啟動時，列出的 units 也應該被啟動，但是當列出的 units 沒有成功啟動也不影響此 unit

- **Requires**: 強依賴，如果任一所需的 unit 沒有啟動，則此 unit 也不會啟動

- **BindsTo**: 比 Requires 更強的依賴關係。如果被綁定的 unit 失效，這個 unit 也會被停止

- **After**: 確保此 unit 在指定的 units 之後啟動

- **Before**: 確保此 unit 在指定的 units 之前啟動

- **OnFailure**: 指定當此 unit 啟動失敗時應該啟動的單元

### \[Service\] 選項

專用於描述如何管理和運行服務的 units

- **Type**: 定義啟動類型（如 **`simple`**, **`forking`**, **`oneshot`**, **`dbus`**, **`notify`**, **`idle`**）

- **ExecStart**: 指定啟動服務時執行的命令

- **ExecStartPre** / **ExecStartPost**: 分別在 **`ExecStart`** 命令之前和之後執行的命令

- **ExecStop**: 指定停止服務時執行的命令

- **ExecReload**: 指定重載服務配置時執行的命令

- **Restart**: 定義在什麼情況下應自動重啟服務（如 **`always`**, **`on-success`**, **`on-failure`**, **`on-abnormal`**, **`on-watchdog`**, **`on-abort`**, **`never`**）

- **TimeoutStartSec**: 從開始啟動到發出超時信號之間的時間

- **TimeoutStopSec**: 從開始停止到發出超時信號之間的時間

- **Environment**: 設置在啟動服務時應用的環境變量

- **WorkingDirectory**: 指定服務的工作目錄

- **User**, **Group**: 指定運行服務的用戶和群組

- **RestartSec**: 定義重啟之間的等待時間

### \[Install\] 選項

- **WantedBy**: 定義哪個目標（target）在被啟用時應該包含此 unit

- **RequiredBy**: 類似於 WantedBy，但創建更強的依賴關係

- **Alias**: 為此 unit 定義別名

## Service 類型

**`Service`** 單元可以有幾種不同的類型，這些類型定義了 systemd 如何管理和啟動這些服務。這個設置是在服務單元文件的 **`[Service]`** 部分中的 **`Type=`** 關鍵字下設定的。每種類型都有其特定的用途和行為

### **1\. simple**

**`simple`** 是最常見的類型，用於那些一旦啟動就會持續運行的 process

```plain
[Service]
Type=simple
ExecStart=/usr/bin/python3 /path/to/your/server.py
```

### **2\. forking**

**`forking`** 類型用於那些會產生（fork）一個 child process 服務，常見於傳統的 UNIX 服務。在這種模式下，啟動程序會退出，而 child process 會繼續運行作為服務的主進程

```plain
[Service]
Type=forking
ExecStart=/usr/sbin/apache2ctl start
ExecStop=/usr/sbin/apache2ctl stop
```

### **3\. oneshot**

**`oneshot`** 類型用於那些只執行一次任務且結束的腳本或程序，通常用於初始化、單次任務和其他一次性操作，此類型的服務在完成 `ExecStart` 後會被視為已完成

```plain
[Service]
Type=oneshot
ExecStart=/bin/run_once
RemainAfterExit=yes
```

### **4\. dbus**

類似 `simple`，但 units 只在 main process 取得 D-Bus 名的時候將其視為啟動

```plain
[Service]
Type=dbus
BusName=org.example.MyApp
ExecStart=/usr/bin/myapp
```

### **5\. notify**

**`notify`** 類型用於那些會向 systemd 發送通知訊號來表示服務已經準備好的程式

這樣允許複雜的異步初始化流程，來讓系統在收到通知前不要視為啟動

### **6\. idle**

這類型的服務延遲執行直到其他啟動任務都完成

通常用在非關鍵的 background job，不希望影響啟動時間



## systemd 操作

### Reload systemd 配置

創建或修改了 service 文件後，需要讓 systemd 重新讀取

```bash
sudo systemctl daemon-reload
```

### 啟動及啟用 service

```bash
sudo systemctl start example.service
sudo systemctl enable example.service
```

### 狀態檢查

```bash
$ sudo systemctl status docker.service
```

```bash
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; preset: disabled)
     Active: active (running) since Sun 2024-04-21 09:19:38 UTC; 2 days ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 408 (dockerd)
      Tasks: 9
     Memory: 50.5M
        CPU: 1min 13.567s
     CGroup: /system.slice/docker.service
             └─408 /usr/bin/dockerd --registry-mirror=https://mirror.gcr.io --host=fd:// --containerd=/var/run/containerd/containerd.sock

Apr 21 09:19:36 shareclass-tw-elasticsearch-dev systemd[1]: Starting docker.service...
Apr 21 09:19:38 shareclass-tw-elasticsearch-dev dockerd[408]: time="2024-04-21T09:19:38.008550852Z" level=info msg="Starting up"
Apr 21 09:19:38 shareclass-tw-elasticsearch-dev dockerd[408]: time="2024-04-21T09:19:38.127032296Z" level=info msg="[graphdriver] trying configured driver: overlay2"
```

- **`Loaded`** 欄位描述了 unit 文件的載入狀態，包括其位置和配置狀態：

   - **loaded**: 表示相關的 unit 文件已成功加載到 systemd 中

   - **not-found**: 表示未找到指定的 unit 文件

   - **error**: 表示在嘗試加載 unit 文件時發生錯誤

   - **masked**: 表示 unit 文件被屏蔽了，屏蔽等同於禁用，但是更為徹底，因為即便嘗試啟動該服務也會被阻止

- **`Active`** 欄位描述了服務的運行狀態：

   - **active (running)**: 服務已經啟動並正在運行

   - **active (exited)**: 服務已經執行完成其任務並正常退出

   - **active (waiting)**: 服務正在等待某個條件被滿足，通常用於服務連接或初始化

   - **active (activating)**: 服務正在啟動過程中

   - **inactive**: 服務當前未運行

   - **activating**: 服務正在啟動

   - **deactivating**: 服務正在停止

   - **failed**: 服務嘗試啟動但失敗

## 列出 service 單位

**`systemctl`** 命令中，**`list-units`** 子命令用於列出當前系統上的 unit 文件的狀態

```bash
sudo systemctl list-units --type=service
```

### 選項說明

- **`list-units`**: 這個參數指示 **`systemctl`** 列出正在管理的 units

- **`--type=service`**: 這個選項限制輸出只顯示類型為 service 的 units。不加此選項則會顯示所有類型的 units，包括 service, socket, device 等

```bash
UNIT FILE            LOAD   ACTIVE SUB     DESCRIPTION
atd.service          loaded active running Deferred execution scheduler
cron.service         loaded active running Regular background program processing daemon
dbus.service         loaded active running D-Bus System Message Bus
docker.service       loaded active running Docker Application Container Engine
nginx.service        loaded active running A high performance web server and a reverse proxy server
sshd.service         loaded active running OpenBSD Secure Shell server
init-dm-devs.service loaded active exited  Initialize device mapper devices configured on kernel command line

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
```

- **UNIT**: 列出了服務的名稱

- **LOAD**: 顯示服務的加載狀態，如 **`loaded`** 表示服務已經正確加載

- **ACTIVE**: 顯示服務的活躍狀態，如 **`active`** 表示正在運行

- **SUB**: 提供更具體的服務狀態，對於服務來說通常是 **`running`**

- **DESCRIPTION**: 簡要描述了服務的用途或功能