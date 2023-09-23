`int main(int argc, char *argv[], char *envp[])`

```c
#include <stdio.h>
int main(int argc, char (*argv)[0])
{
  puts(((char **) argv)[0]);
  return 0;
}
```
> `argv[0]` 就是執行檔的名字


利用[[GDB]]來看這段程式

```c
(gdb) b main
(gdb) r
(gdb) print *((char **) argv)
$1 = 0x7fffffffe7c9 "/tmp/x"
```


```c
(gdb) x/4s (char **) argv
0x7fffffffe558: "\311\347\377\377\377\177"
0x7fffffffe55f: ""
0x7fffffffe560: ""
0x7fffffffe561: ""
```

換個方法表示 `(char **) argv` => `((char **) argv)[0]`
```c
(gdb) x/4s ((char **) argv)[0]
0x7fffffffe7c9: "/tmp/x"
0x7fffffffe7d0: "LC_PAPER=zh_TW"
0x7fffffffe7df: "XDG_SESSION_ID=91"
0x7fffffffe7f1: "LC_ADDRESS=zh_TW"
```
> argc紀錄argv有多大
> ==main在執行時其實是shell提供 argv做為參數傳進去==

---
- 最後一項envp (environment variable) 為環境變數

#C語言 #Pointer #Array 