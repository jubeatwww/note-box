
# 基本概念
- [The Internals of “Hello World” Program](http://www.slideshare.net/jserv/helloworld-internals)
![[Pasted image 20230930170026.png]]


# Process所看到的Memory

- [Virtual Memory 與 C 語言角度的 memory](http://www.study-area.org/cyril/opentools/opentools/x909.html)
- [Process 角度的 Memory](https://manybutfinite.com/post/anatomy-of-a-program-in-memory/)
> ==在 ELF 裡頭為 section, 進入到 memory 後則以 process 的 segment 去看==
> 此處皆為 virtual memory address (VMA)
> [VMA與ELF對應](https://www.jollen.org/blog/2007/01/process_vma.html)

## Segments

- [[Stack]] : 由高位址長至低位址，儲存函式呼叫時個別 stack frame 的 local variables 與 return address 等
- Heap : 由低位址長至高位址，動態記憶體配置
> 此處以x86的記憶體為例，不同的處理器架構高低位配置可能有所不同
- BSS segement: Block Started by Symbol ，尚未初始化的變數
	- 沒有指定初始值，因此不需要特別放入執行檔，但是需要配置空間
- Data segement : 已經被初始化後的變數
	- 宣告的變數會存在 **data segment** ，例如: `int content = 10` 
	- 宣告在此的 pointer 所指向的內容則不會，也就是說 `gonzo` 的內容 `God's own prototype` 會是放在 text segment 裡，只有 pointer 所存的**位址**會在 data segment
- Text segement : 存放使用者程式的 binary code
![[Pasted image 20230930173345.png]]

- instructions: 自 object file (ELF) 映射 (map) 到 process 的 program code (機械碼)
- static data: 靜態初始化的變數
- BSS: 全名已 ==不可考==，一般認定為 "Block Started by Symbol”，未初始化的變數或資料
    - 可用 `size` 命令來觀察
- Heap 或 data segment: 執行時期才動態配置的空間
    - sbrk 系統呼叫 (sbrk = set break)
    - malloc/free 實際的實作透過 sbrk 系統呼叫

## [[ELF segment & section]]

## [[回傳值]]

#C語言 #Function 