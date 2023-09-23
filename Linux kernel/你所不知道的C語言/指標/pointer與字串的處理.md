考慮以下情境
```c
char *r;
strcpy(r, s);
strcat(r, t);
```
> 由於r並未指向任何位置，str操作找不到 `\0`的狀況下這段程式不會運作

```c
char r[100];  //編譯器配置的其實是至少100個，可能會多給
strcpy(r, s);
strcat(r, t);
```
- 當s, t的長度合理的時候就可以正常運作
- ==超出r配置的空間會影響資料安全==
	- 對於C來說不會特別做保護，超出宣告的範圍會有覆蓋到其他空間的可能性

- C99後支援動態宣告
```c
char r[strlen(s) + strlen(t)];
strcpy(r, s); strcat(r, t);
```
或利用malloc
```c
char *r = malloc(strlen(s) + strlen(t));
strcpy(r, s); strcat(r, t);
```
- 
- 
其實malloc是有可能失敗的
	- size是0的話，malloc會給你一個null
	- malloc後要很好的去free
	- 有可能記憶體空間不足以配置 ^bff8bd

確保分配的長度不是0
```c
char *r = malloc(strlen(s) + strlen(t) + 1); // use sbrk; change program break
if (!r) exit(1); /* print some error and exit */
strcpy(r, s); strcat(r, t);
```



#C語言 #Pointer #Array #String