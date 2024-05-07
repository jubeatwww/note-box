---
tags:
  - http
title: application/octet-stream
slug: /networking/application-octet-stream
description: 探索 application/octet-stream 的應用，瞭解其與 Content-Disposition 的互動，並探討如何安全地處理未知類型文件的下載和顯示。
keywords:
  - application/octet-stream
  - Content-Disposition
  - MIME type
  - RFC 2046
  - RFC 2616
  - web security
---

> [application/octet-stream](https://mimetype.io/application/octet-stream) @ mimetype.io

> [学习笔记：application/octet-stream](https://www.cnblogs.com/RioTian/p/17592907.html) @ Koshkaaa

> [Do I need Content-Type: application/octet-stream for file download?](https://stackoverflow.com/questions/20508788/do-i-need-content-type-application-octet-stream-for-file-download) @ stackoverflow
---
在上傳檔案到 `S3` 等儲存空間時，若沒有指定 MIME type，就會被預設被設置為 `application/octet-stream`
- 這個類型在 RFC 2046 中被定義為“任意的二進制數據”，常被用於那些其唯一目的是被保存到磁碟上，之後與網絡無關的實體
## **瀏覽器的反應**
當 HTTP header 中的 **`Content-Type`** 設置為 **`application/octet-stream`** 時，瀏覽器的默認行為是觸發下載過程，而不是嘗試解析或執行文件。這種行為對於保護用戶安全非常重要，因為它防止了潛在的危險執行檔自動運行，進而降低了網絡安全風險
## **實際應用**
1. **電子郵件附件**：如果電子郵件客戶端不能確定附件的正確類型，它可能會將附件設定為 **`application/octet-stream`**，從而將如何處理文件的決定權交給接收者。
2. **文件下載**：網站和網絡服務器對於類型不明的文件經常使用此 MIME 類型，以確保這些文件被當作一般的下載對待，而不是被瀏覽器嘗試打開或執行。
## **與 Content-Disposition 的運用**
- **`Content-Disposition`** 可以與其他類型的 **`Content-Type`** 結合使用
    - 比如 **`image/png`** 或甚至是 **`text/html`**，來指明你希望文件被保存而非顯示
    - RFC 2616 還提到了擴展標記的可能性，如今大多數瀏覽器識別 **`inline`** 意味著如果可能的話希望顯示實體（即，如果是瀏覽器能夠顯示的類型，否則它別無選擇）
```
Content-Type: application/octet-stream
Content-Disposition: attachment; filename="picture.png"
```
> 意味著：“我不知道這是什麼。請將其保存為文件，最好命名為 picture.png。”


```
Content-Type: image/png
Content-Disposition: attachment; filename="picture.png"
```
> 意味著：“這是一個 PNG 圖片。請將其保存為文件，最好命名為 picture.png。”


```
Content-Type: image/png
Content-Disposition: inline; filename="picture.png"
```
> 意味著：“這是一個 PNG 圖片。請顯示它，除非你不知道如何顯示 PNG 圖片。否則，或如果用戶選擇保存它，我們建議使用 picture.png 作為文件的名稱。”