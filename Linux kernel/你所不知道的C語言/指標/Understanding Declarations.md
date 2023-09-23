
## 解釋以下程式

取自 [C Traps and Pitfalls](http://www.literateprogramming.com/ctraps.pdf) 的案例 “Understanding Declarations”:
```
(*(void(*)())0)();
```
^6e23fa



![[Understanding Declarations.jpg]]
對應的組合語言，搭配 `-Os` (空間最佳化)

```
main:
    pushq       %rax
    xorl        %eax, %eax
    call        *%rax
    xorl        %eax, %eax
    popq        %rdx
    ret
```

- `xorl %eax. %eax` ，對自己XOR必定是0
- 呼叫一個函式，其所在的位置是0

### 執行結果?

^c171e4

`0x0`這個地址是沒辦法在使用者層級呼叫的，因此會得到segmentation fault

##  科技公司面試題:

```
void **(*d) (int &, char **(*)(char *, char **));
```

^b6c600


![[科技公司面試題.jpg]]
### 此類function pointer的用途?

^445c42

- [How to read this prototype?](http://stackoverflow.com/questions/15739500/how-to-read-this-prototype)
- `void ( *signal(int sig, void (*handler)(int) ) (int)`


#C語言 #Pointer