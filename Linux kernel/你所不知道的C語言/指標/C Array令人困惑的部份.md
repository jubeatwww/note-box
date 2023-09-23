- 根據你使用的地方不同，Array 會有不同的意義
	- 在expression內，array永遠會被轉換為pointer
	- function argument以外的declaration還是一個array，而且==不能==改寫成pointer
	- function argument中array會被轉成pointer


若現在這裡有一個全域變數 (global)

```c
char a[10];
```
> 在另一個檔案，用extern char \*a來操作是不允許的
> 因為對應到的指令(instruction)不同

- 在function argument中
	-  `void function(char a[])` 與 `void function(char * const a)` 是等價的
	- 真實的型態會是pointer

- 因此不能用sizeof來抓他的大小(因為是pointer)，這也是[[main function]]內需要一個argc來判斷argv長度的原因
- array是 unmodifiable [[lvalue]]，除了被轉成pointer，他還會是不能更改的指標，因此需要`const`修飾
- 在取值的時候 array與pointer幾乎雷同，但
	- array為兩步取值: array address -> offset
	- pointer為三步: 載入pointer address -> pointer value當作address -> offset取值

#C語言 #Pointer #Array 