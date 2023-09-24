# 定義

在規格書的 Rationale 將 lvalue 定義為 object ==locator== (C99)

- [C99 6.3.2.1] footnote
    > The name “lvalue” comes originally from the assignment expression E1 = E2, in which the left operand E1 is required to be a (modifiable) lvalue. It is perhaps better considered as representing an object “locator value”. What is sometimes called “rvalue” is in this International Standard described as the “value of an expression”. An obvious example of an lvalue is an identifier of an object. As a further example, if E is a unary expression that is a pointer to an object, *E is an lvalue that designates the object to which E points.


# 範例

```c
int a = 10;
int *E = &a;
++(*E);  // a = a + 1
++(a++); // error
```

> E 就是上面 C99 註腳提及的 “a pointer to an object”

## E的identifier所代表的身分

- object就是a占用的記憶體空間
	- `E`
		- object: 儲存address of int object的區域
		- lvalue: E object的位置，E本身區域的address
	- `*E`
		- lvalue: 對E deference得到int object的位置，就是==lvalue that designates the object which E points==

## 錯誤

使用gcc會得到
> error: **lvalue** required as increment operand

- `a++`會回傳a的value，而這個暫存值也就是個non-lvalue
	- `++()`需要是一個lvalue
	- 要寫回data，就必須有地方(locator)寫入

- 延伸閱讀:
	- [Understanding lvalues and rvalues in C and C++](http://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c)
	- 透過 Lvalue 解釋 compound literal 效果的[案例](http://stackoverflow.com/questions/14105645/is-it-possible-to-create-an-anonymous-initializer-in-c99/14105698#14105698)


#C語言 