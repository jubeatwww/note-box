在程式執行過程中，用來儲存函式呼叫的相關資訊的資料結構
- 當函式被呼叫時，一個新的stack frame會被push到call stack的頂部
- 函式結束return時，該stack frame會被pop掉

- 典型的stack frame包含以下
	1. **返回地址 (Return Address)**: 當函式呼叫結束後，程式應該返回到哪裡繼續執行。這通常是呼叫該函式的下一條指令的地址
	2. **局部變量 (Local Variables)**: 在函式內部宣告的變量
	3. **傳入參數 (Passed Parameters)**: 呼叫該函式時傳入的參數值
	4. **保存的暫存器 (Saved Registers)**: 許多架構的指令集會在呼叫函式之前保存某些暫存器的值，並在返回之後恢復它們
	5. **動態連結資訊 (Dynamic Link)**: 指向前一個堆疊框架的指針，這有助於當函式結束時找到前一個堆疊框架的位置
