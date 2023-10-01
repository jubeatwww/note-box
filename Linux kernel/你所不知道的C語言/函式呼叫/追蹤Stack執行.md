```c
int funcB(int a) {
    return a + 1;
}

int funcA(int b) {
    return funcB(b);
}

int main() {
    int a = funcA(1);
    return 0;
}
```

編譯時加上 `-g` 以利後續 [[GDB]] 追蹤
```shell
$ gcc -o stack -g -no-pie stack.c
```
> `-no-pie` 編譯選項是抑制 [Position Independent Executables (PIE)](https://www.redhat.com/en/blog/position-independent-executables-pie)，便於後續分析。若你的 gcc 版本較舊，可能沒有該編譯選項，可自行移去
> PIE 是啟用 [address space layout randomization](https://en.wikipedia.org/wiki/Address_space_layout_randomization) (ASLR) 的預備動作，用以強化核心載入程式時，確保虛擬記憶體的排列不會總是一樣。

```shell
$ gdb -q stack
```

在 GDB 中使用 `disas` 命令將其反組譯，預設是 AT&T 語法，我們可改為 Intel 語法，得到更簡潔的輸出:
```c
(gdb) set disassembly-flavor intel
```


```c
(gdb) disas main
Dump of assembler code for function main:
   0x0000000000400501 <+0>:     push   rbp
   0x0000000000400502 <+1>:     mov    rbp,rsp
   0x0000000000400505 <+4>:     sub    rsp,0x10
   0x0000000000400509 <+8>:     mov    edi,0x1
   0x000000000040050e <+13>:    call   0x4004d6 <funcA> # 注意到此處，讀者可先抄寫本地址
   0x0000000000400513 <+18>:    mov    DWORD PTR [rbp-0x4],eax # 這段地址也可抄下，函式呼叫的返回地址
   0x0000000000400516 <+21>:    mov    eax,0x0
   0x000000000040051b <+26>:    leave
   0x000000000040051c <+27>:    ret
End of assembler dump.

(gdb) disas funcA
Dump of assembler code for function funcA:
   0x00000000004004d6 <+0>:     push   rbp
   0x00000000004004d7 <+1>:     mov    rbp,rsp
   0x00000000004004da <+4>:     sub    rsp,0x10
   0x00000000004004de <+8>:     mov    DWORD PTR [rbp-0x4],edi
   0x00000000004004e1 <+11>:    mov    eax,DWORD PTR [rbp-0x4]
   0x00000000004004e4 <+14>:    mov    edi,eax
   0x00000000004004e6 <+16>:    mov    eax,0x0
   0x00000000004004eb <+21>:    call   0x4004f2 <funcB>
   0x00000000004004f0 <+26>:    leave
   0x00000000004004f1 <+27>:    ret
End of assembler dump.

(gdb) disas funcB
Dump of assembler code for function funcB:
   0x00000000004004f2 <+0>:     push   rbp
   0x00000000004004f3 <+1>:     mov    rbp,rsp
   0x00000000004004f6 <+4>:     mov    DWORD PTR [rbp-0x4],edi
   0x00000000004004f9 <+7>:     mov    eax,DWORD PTR [rbp-0x4]
   0x00000000004004fc <+10>:    add    eax,0x1
   0x00000000004004ff <+13>:    pop    rbp
   0x0000000000400500 <+14>:    ret
End of assembler dump.
```

---

# 觀察

## 起始
- 在進入 `funcA()`之前中斷
```c
(gdb) b *0x000000000040050e
Breakpoint 1 at 0x4004ec: file stack.c, line 10.
(gdb) r
```
- 中斷後印出`rbp` `rsp`
```c
(gdb) p $rbp
$1 = (void *) 0x7fffffffe480

(gdb) p $rsp
$2 = (void *) 0x7fffffffe470
```

![[Pasted image 20231001154004.png]]

---

## `call funcA`

- 執行 `call funcA` 之後
	- `call`指令會 push ==next== instruction address
		- 使用下一條指令才能在return後執行下一行
	- 這個地址就是回到`main`的返回地址
```c
(gdb) x $rsp
0x7fffffffe480: 00400513
```
![[Pasted image 20231001155645.png]]

- 進入`funcA`之後會執行以下
```c
Dump of assembler code for function funcA:
   0x00000000004004d6 <+0>:     push   rbp
   0x00000000004004d7 <+1>:     mov    rbp,rsp
   0x00000000004004da <+4>:     sub    rsp,0x10
   0x00000000004004de <+8>:     mov    DWORD PTR [rbp-0x4],edi
   0x00000000004004e1 <+11>:    mov    eax,DWORD PTR [rbp-0x4]
   0x00000000004004e4 <+14>:    mov    edi,eax
   0x00000000004004e6 <+16>:    mov    eax,0x0
   0x00000000004004eb <+21>:    call   0x4004f2 <funcB>
   0x00000000004004f0 <+26>:    leave
   0x00000000004004f1 <+27>:    ret
End of assembler dump.
```
- `push rbp`
![[Pasted image 20231001160000.png]]

- `mov rbp, rsp`
![[Pasted image 20231001160028.png]]
- `sub rsp,0x10`
![[Pasted image 20231001160132.png]]

---

## `leave`
- 執行完`funcA()`後會碰到 `leave`，等同於
```c
mov rsp, rbp
pop rbp
```

- `mov rsp, rbp`
	- 將`rsp`移回stack frame頂端
	- 相當於跟將整個stack frame pop出來的前置作業
![[Pasted image 20231001160747.png]]

- `pop rbp`
	- 在`call funcA`的時候執行過的`push rbp`，儲存的上一個stack frame的頂端位址，現在pop回來將`rbp`移回去
![[Pasted image 20231001160815.png]]
- `ret`
	- 這時 `rsp`已經指向回到`main`的地址，呼叫`ret`就會將`rip`指向返回地址，stack frame就會恢復成在main的stack frame
![[Pasted image 20231001163826.png]]

---

## 比較`funcA` `funcB`

```c
(gdb) disas funcA
Dump of assembler code for function funcA:
   0x00000000004004d6 <+0>:     push   rbp
   0x00000000004004d7 <+1>:     mov    rbp,rsp
   0x00000000004004da <+4>:     sub    rsp,0x10
   0x00000000004004de <+8>:     mov    DWORD PTR [rbp-0x4],edi
   0x00000000004004e1 <+11>:    mov    eax,DWORD PTR [rbp-0x4]
   0x00000000004004e4 <+14>:    mov    edi,eax
   0x00000000004004e6 <+16>:    mov    eax,0x0
   0x00000000004004eb <+21>:    call   0x4004f2 <funcB>
   0x00000000004004f0 <+26>:    leave
   0x00000000004004f1 <+27>:    ret
End of assembler dump.

(gdb) disas funcB
Dump of assembler code for function funcB:
   0x00000000004004f2 <+0>:     push   rbp
   0x00000000004004f3 <+1>:     mov    rbp,rsp
   0x00000000004004f6 <+4>:     mov    DWORD PTR [rbp-0x4],edi
   0x00000000004004f9 <+7>:     mov    eax,DWORD PTR [rbp-0x4]
   0x00000000004004fc <+10>:    add    eax,0x1
   0x00000000004004ff <+13>:    pop    rbp
   0x0000000000400500 <+14>:    ret
End of assembler dump.
```

- `funcB`沒有`sub rsp,0x10`
	- **保存暫存器**：即使 `funcA` 在邏輯上只是調用了 `funcB` 而已，編譯器可能還是會選擇在調用其他函數之前保留某些暫存器的值。在這裡，它將 `edi` 的值保存到了堆疊上的位置 `[rbp-0x4]`
	- **堆疊對齊(alignment)**：在某些系統和應用中，堆疊的對齊是很重要的，特別是在x86-64架構上
		- 為了確保性能和某些指令的正確操作，堆疊地址可能需要按照16位元組邊界對齊
		- 即使 `funcA` 在邏輯上不需要這麼多的空間，編譯器可能還是會生成 `sub rsp, 0x10` 指令以確保堆疊的對齊
	- **函式調用**：`funcA` 中調用了 `funcB`，為了確保調用過程中堆疊的穩定，可能需要分配一些額外的空間
		- **返回地址**：確保這個地址不被誤刪或覆蓋
		- **局部變數和臨時數據**：在函式內，可能會在stack上分配空間存儲局部變數或臨時數據確保這些數據的空間不會互相干擾、或與其他stack內容相互干擾
		- **參數傳遞**：特定的呼叫約定可能會要求某些參數通過堆疊傳遞
		- **呼叫其他函式**：在函式內部，當其他函式被調用時，當前的函式可能還需要在堆疊上保留自己的狀態（局部變數、返回地址等），以確保在返回時能恢復正確的狀態
- 編譯器已知 `funcB` 之後，就不會再呼叫別的函式，也沒有 `push`, `pop` 等操作，因此 `rsp` 也不需要特別保留一段空間給 `funcB`

# Stack frame定義

stack frame 之範圍於 [System V Application Binary Interface AMD64 Architecture Processor Supplement](https://github.com/hjl-tools/x86-psABI/wiki/x86-64-psABI-1.0.pdf) 中定義為
![[Pasted image 20231001164700.png]]

#C語言 #Function 