---
title: 在Google Compute Engine上用 container 部署Elasticsearch (上)
slug: /Cloud/gce-cos-terraform-part1
tags:
  - cloud
  - gcp
  - terraform
---
當需要在雲端快速部署可擴展的應用時，Google Compute Engine（GCE）配合Container-Optimized OS提供了一個高效的選擇
下面將利用Terraform來部署一個Elasticsearch容器實例，並結合外接空白硬碟以優化數據遷移和存儲

## 配置Terraform
### 定義資源：靜態內部IP

首先，定義一個內部IP地址，這個地址將被用作我們虛擬機的網絡接口
(由於這邊使用情境為純內部 service，因此不設定外部IP)

```hcl
resource "google_compute_address" "internal_ip" {
  name         = var.service_name
  address_type = "INTERNAL"
  purpose      = "GCE_ENDPOINT"
  project      = var.project_id
  region       = var.project_region
}
```

### 設置虛擬機實例

接下來，設定GCE虛擬機實例
使用Container-Optimized OS作為啟動盤的映像，並將Docker容器的規格通過metadata傳遞給GCE，這裡使用的是類似於Kubernetes的配置方式
也可以利用GCE的 UI 介面設定好之後畫面上會有等效的程式碼可以照搬

```hcl
resource "google_compute_instance" "instance" {
  boot_disk {
    initialize_params {
	    /* 省略了部分初始化和配置代碼...*/
		image = "projects/cos-cloud/global/images/cos-stable-109-17800-66-78"
    }
  }
	/* 省略了部分初始化和配置代碼...*/

  metadata = merge(
    {
      gce-container-declaration = var.gce_container_declaration
    },
    var.startup_script != null ? {
      startup-script = var.startup_script
    } : {}
  )

  network_interface {
    network_ip  = google_compute_address.internal_ip.address
    subnetwork  = "projects/shareclass-tw/regions/asia-east1/subnetworks/default"
  }

  dynamic "attached_disk" {
    for_each = var.attached_disk
    content {
      source      = attached_disk.value.source
      device_name = attached_disk.value.device_name
      mode        = attached_disk.value.mode
    }
  }
  depends_on = [google_compute_address.internal_ip]
}

```

在這個`google_compute_instance`資源定義中，我們設置了一台虛擬機來運行Elasticsearch容器。關鍵配置包括
- 指定操作系統映像為Container-Optimized OS的最新穩定版本
- 通過`metadata`傳遞Docker容器的配置（這里用了`gce-container-declaration`）
這種方法類似於Kubernetes的pod定義，允許細粒度地控制容器的運行環境

### 配置外接硬碟

```
resource "google_compute_disk" "elasticsearch" {
  name = "volume-${var.service_name}"
  type = "pd-balanced"
  zone = var.project_zone
  size = var.disk_size
}
```
這部分代碼創建了一個外接硬碟，將用於Elasticsearch數據的存儲

### 定義容器規格

```
data "template_file" "gce_container_declaration" {
  template = file("${path.module}/gce_container_declaration.tpl")

  vars = {
    service_name = local.service_name
    image        = local.image
    disk_name    = google_compute_disk.elasticsearch.name
  }
}
```
- `template_file`數據源用於生成容器的配置
	- 讀取一個模板文件，並替換模板中的變量以生成最終的容器配置
	- 這個過程讓配置更加模塊化和可重用，便於管理大型部署
- 若使用GCE的等效程式碼，只能看到 `spec:\n  containers:\n  - name: instance....` 這種單行設定，不利於以後維護

## 定義模組

將所有配置集成到一個模組中，方便管理和重用
```
module "elastic_search" {
  source         = "../compute_container"
  project_id     = var.project_id
  project_region = var.project_region
  project_zone   = var.project_zone
  service_name   = var.service_name
  environment    = var.environment
  machine_type   = var.machine_type
  attached_disk  = [
    {
      device_name = google_compute_disk.elasticsearch.name
      mode        = "READ_WRITE"
      source      = google_compute_disk.elasticsearch.self_link
    }
  ]
  gce_container_declaration = data.template_file.gce_container_declaration.rendered

  depends_on = [
    google_compute_disk.elasticsearch,
    data.template_file.gce_container_declaration
  ]
}
```

最後，所有這些資源和配置被封裝進一個模組。這使得整個Elasticsearch部署可以作為一個單元被重用和管理，無論是在當前項目還是跨項目都能輕鬆配置和更新

## 結語

通過上述步驟，我們已經成功配置了一個在Google Compute Engine上運行Elasticsearch的環境
這個環境利用了Container-Optimized OS的穩定性和安全性，以及Terraform的自動化部署能力
通過細致的配置和模組化的設計，我們確保了系統的可擴展性和易管理性

後半段將更深入地探討在此架構中運行時遇到的特定技術問題，特別是之前提及的磁碟權限問題，這是一個在使用外接硬碟和Container-Optimized OS時常見的挑戰
將詳細討論問題的根源——如何在容器的啟動和運行過程中保護好磁碟的權限不被覆蓋，以及如何利用startup script中創建的專用service來動態調整和修正權限
