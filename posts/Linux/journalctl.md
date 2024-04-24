---
tags:
  - linux
---
> [journalctl(1)](https://man7.org/linux/man-pages/man1/journalctl.1.html) @ Linux manual page

---

**`journalctl`** 是一個用於查看和操作 systemd 日誌的強大命令行工具，主要用於查看、分析和調試系統上的日誌資訊

## **基本概念**

在 Linux 中，**`systemd`** 系統和服務管理器使用日誌來收集和管理從啟動到關機的所有日誌**`journalctl`** 命令提供了一個介面來檢索和查看這些日誌資訊

## **常用命令及參數**

### **1\. 查看所有日誌**

```bash
journalctl
```

### **2\. 反向顯示日誌（最新的日誌條目先顯示）**

```bash
journalctl -r
```

### **3\. 查看特定服務的日誌**

```bash
journalctl -u nginx.service
```

### **4\. 查看指定時間的日誌**

```bash
journalctl --since "2021-05-01" --until "2021-05-02"
```

### **5\. 實時跟踪最新日誌**

```bash
journalctl -f
```

### **6\. 查看 kernel 日誌**

```bash
journalctl -k
```

### **7\. 按頁顯示日誌條目**

```bash
journalctl --no-pager
```

預設情況下，**`journalctl`** 輸出會通過分頁 process（如 less）來顯示

使用 **`--no-pager`** 參數可以直接在 terminal 輸出所有日誌，不進行分頁

## 進階用法

### **壓縮舊日誌**

為了管理空間，**`journalctl`** 允許壓縮舊日誌：

```bash
journalctl --vacuum-size=1G
```

刪除舊的日誌，直到日誌文件大小被壓縮至1GB

### **日誌持久性**

預設情況下，系統可能只存儲當前會話的日誌。要配置日誌的持久存儲，需要創建一個特定的目錄：

```bash
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
```