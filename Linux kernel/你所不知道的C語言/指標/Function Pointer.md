
```c
int main() { return (********puts)("Hello"); }
```
> 為何可以運作?


# 規格書摘要

根據 C99 規格書 ^818b62

- C99 [ 6.3.2.1 ] A function designator is an expression that has function type
    - **Except** when it is the operand of the **sizeof** operator or the unary **&** operator, a **function designator** with type ‘‘function returning type’’ **is converted to** an expression that has type ‘‘**pointer to function returning type’**’.
	    - 若非一元 `&` 或 `sizeof` 運算，則 function designator 會認定為指向函式的指標型態
	    - 假設你有一個函數 `int foo(void);`。當你僅提到 `foo`（不帶括號）時，它實際上會被視為指向該函數的指針，除非它是 `sizeof` 或 `&` 的操作元
    - `*` is Right associative operator
	    - 這告訴我們 `*` 運算符（當用作解參考運算符時）是右關聯的。這意味著當多個 `*` 運算符連續出現時，它們將從右至左地評估
	    - 例如，在指針到指針的表達式 `**p` 中，首先對最右邊的 `*` 進行評估，然後再對左邊的 `*` 進行評估

- C99 [ 6.5.1 ] It is an lvalue, a function designator, or a void expression if the unparenthesized expression is, respectively, an lvalue, a function designator, or a void expression.
> 如果你有一個表達式，而這個表達式沒有被括弧包圍，並且它是一個 lvalue、function designator 或 void expression，那麼即使你將其放入括弧中，它仍然會保持其原本的性質
1. 假設 `x` 是一個整數變量，那麼 `x` 是一個 lvalue。如果我們用括弧包裹它，`(x)` 仍然是一個 lvalue。
2. 假設 `f` 是一個函數，那麼 `f` 是一個 function designator。用括弧包裹它，`(f)` 仍然是一個 function designator。
3. 考慮一個如 `printf("Hello, world!");` 的表達式，它是一個返回 void 的函數呼叫，因此它是一個 void expression。用括弧包裹它 `(printf("Hello, world!"))` 仍然是一個 void expression


- C99 [ 6.5.3.2-4 ] The unary * operator denotes indirection. **If the operand points to a function, the result is a function designator**
	- 如果存取的東西是function designator，不管做幾次操作都還是function designator

```c
int foo(void) { return 42; }

int (*ptr_to_foo)(void) = foo; // ptr_to_foo is a pointer to the function foo
```
> 在上述代碼中，`foo` 被隱式轉換為一個指向函數 `foo` 的指針

```c
int result = (*ptr_to_foo)();
```
> 這裡的 `*ptr_to_foo` 通過解參考得到函數設計符，這意味著你再次得到函數 `foo` 本身。然後，通過在其後面添加 `()`，你可以呼叫該函數。
> 值得注意的是，在這種情境下，由於函數指針的語法和語義，你通常可以直接使用 `ptr_to_foo()` 來呼叫該函數，無需明確地解參考它。


# 回到 `********puts`

- 程式碼 `(********puts)` 
	- C99[6.5.1] : 可看作 `(*(*(*(*(*(*(*(*puts))))))))`
	- C99[ 6.5.3.2-4 ] : 最裡面的括號 `(*puts)`可知它是一個 function designator
	- C99[ 6.3.2.1 ] : 
		- `*puts` 則會解參考該指針，得到原始的函數設計符
		- 往外延伸一個括號 `(*(*puts))`
		- 再多一個 * operator 它還是一個 function designator，最後也會被轉為 pointer to function returning type
	- 依此類推最後 `(********puts)` 仍會是一個 function designator


# 另一個範例

```c
void (*fptr)();
void test();
fptr = test;
```

- [6.3.2.1] : test: void (), 會被轉成 function pointer: `void (*) ()`
- fptr is a pointer to function with returning type
- `(*fptr)` is a function designator, a function designator will be converted to pointer.
- `type of (*fptr): void ()`


```c
(gdb) print test
$1 = {void ()} 0x400526 <test>
(gdb) print test
$2 = {void ()} 0x400526 <test>
(gdb) print fptr
$3 = (void (*)()) 0x400526 <test>
(gdb) print (*fptr)
$4 = {void ()} 0x400526 <test>
(gdb) print (**fptr)
$5 = {void ()} 0x400526 <test>
```



#C語言 #Pointer #Function
