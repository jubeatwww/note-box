# 規格書

- C99 [ 6.5.3.2 ]
	- `&` 所能操作的 operand 只能是：
		- function designator - 基本上就是 function name
		- `[]` or `*` 的操作結果：跟這兩個作用時，基本上就是相消
			- `*` - operand 本身
			-  `[]` - `&` 會消失，而 `[]` 會被轉換成只剩 `+` (註：原本 `[]` 會是 `+` 搭配 `*`)
			    - 例如: `&(a[5]) == a + 5`
				    - `&(*(a + 5))`
		- 一個指向非 bit-field or register storage-class specifier 的 object 的 [[lvalue]]
    > [bit-field](https://hackmd.io/@sysprog/c-bitfield)：一種在 struct 或 union 中使用用來節省記憶體空間的物件;  
    > 特別的用途：沒有名稱的 bit-field 可以做為 padding
    
- 除了遇到 `[]` 或 `*` 外，使用 `&` 的結果基本上都是得到 pointer to the object 或是 function 的 address


# 範例

```c
char str[123];
```

- `str == &str`
	- 型態不同，但值相同
	- 左邊的為**pointer to char**  `char *`
		- C99 [6.3.2.1]：
			- Except when it is the operand of the sizeof operator or the unary & operator, or is a string literal used to initialize an array, an expression that has type ‘‘array of type’’ is converted to an expression with type ‘‘pointer to type’’ that points to the initial element of the array object and is not an [[lvalue]].
		- 除非遇到 sizeof 或是 & 之外，array of type (在這就是指 str) 都會被直接解讀成 pointer to type (在這就是 pointer to char)，而這個 type 是根據 array 的第一個元素來決定的
	- 右邊的則是 **pointer to an array**： `char (*)[123]`
		- 遇到 & 時，str 不會被解讀為 pointer to type，而是做為原本的 object，在這就是 array object，而 address of array object 也就是這個 array object 的起始位址，當然也就會跟第一個元素的位址相同
	- C99 [6.5.9]
		- Two pointers compare equal if and only if both are null pointers, both are pointers to the same object (including a pointer to an object and a subobject at its beginning) or function


#C語言 #Pointer #Array 