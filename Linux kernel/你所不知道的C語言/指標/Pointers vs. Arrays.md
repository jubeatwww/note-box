
- in declaration
	- extern, 如 `extern char x[];` → 不能變更為 pointer 的形式
	- definition/statement, 如 `char x[10]` → 不能變更為 pointer 的形式
		- `char[]` 便可以與pointer互換，因為他是 [[Types|incomplete type]]
		- 有明確的空間就不行
	- parameter of function, 如 `func(char x[])` → 可變更為 pointer 的形式 → `func(char *x)`
- in expression
	- array 與 pointer 可互換
```
int main() {
    int x[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    printf("%d %d %d %d\n", x[4], *(x + 4), *(4 + x), 4[x]);
}
```


在 [The C Programming Language](http://www.amazon.com/The-Programming-Language-Brian-Kernighan/dp/0131103628) 第 2 版，Page 99 寫道:

> As formal parameters in a function definition,

該書 Page 100 則寫:

> `char s[]`; and `char *s` are equivalent.
> 前提是 ==As formal parameters in a function definition==
- `extern`及 `char x[10]` 這兩個形式是不可互換的


> `x[i]` 總是被編譯成 `*(x + i)*` \<- in expression

- 所以從上述我們可以得知\[\] 只是語法糖，真正的操作還是記憶體位址
- 二維陣列也是語法糖，最後是一維的pointer做操作
	- C 提供操作多維陣列的機制 (C99 [6.5.2.1] [[Array Subscription|Array subscripting]])

- 指標的各種情境
	- [[Array Declaration]]
	- [[指標指向陣列元素]]
	- [[pointer與字串的處理]]
	- [[main function]]
	- [[C Array令人困惑的部份]]


延伸閱讀

- [C 語言雜談：指標和陣列 (上)](https://www.cnblogs.com/baochuan/archive/2012/03/22/2410821.html)
- [C 語言雜談：指標和陣列 (下)](https://www.cnblogs.com/baochuan/archive/2012/03/26/2414062.html)


#C語言 #Pointer #Array