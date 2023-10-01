- 每次function call中，其中的一層稱為[[stack frame]]，和兩個暫存器有密切的關係
	- stack pointer
	- frame pointer
# Stack名詞解釋

> x86_64 暫存器為例
- rip (instruction pointer): 記錄下個要執行的指令位址(或有些地方稱pointer counter)
- rsp (stack pointer): 指向 stack 頂端
- rbp (base pointer, frame pointer): 指向 stack 底部
![[Pasted image 20231001151955.png]]

# [[追蹤Stack執行]]




#C語言 #Function 