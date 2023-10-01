
# 數學上的function定義

[數學定義的 Function](https://www.cs.colorado.edu/~srirams/courses/csci2824-spr14/functionsCompositionAndInverse-17.html) (==函數==)

![[Function.png]]
- 函數 (function) f : $A→B$ 是一個對應
	- 滿足:  對所有 $a \in A$，存在惟一 $b \in B$，使得 $f$ 將 $a$ 對應到 $b$
	- 即 
		- $\forall a \in A$，對所有a
		- $\exists! b \in B$，唯一存在一個b
		- 使得 $f(a)=b$
- $A$ 稱為 $f$ 的定義域 (domain)，$B$ 稱為 $f$ 的對應域 (codomain): $f(a)={f(a)|a \in A} \subset B$ 稱為 $f$ 的值域 (range)
- [Function Composition](https://en.wikipedia.org/wiki/Function_composition) 本身可以組合，例如 $g \circ f(x)=g(f(x))$
![[Function Notation.png]]

## [[Parameter v.s. Argument]]


# 數學上的函數與程式語言上的差異

在 C 語言中，“function” 其實是特化的形式，並非數學意義上的函數，而隱含一個狀態到另一個狀態的關聯。 (==函式==)

摘自 [What is the difference between functions in math and functions in programming?](https://stackoverflow.com/questions/3605383/what-is-the-difference-between-functions-in-math-and-functions-in-programming) 的討論

> In functional programming you have [Referential Transparency](http://en.wikipedia.org/wiki/Referential_transparency_(computer_science)), which means that you can replace a function with its value without altering the program. _This is **true**_ in Math too, but _this is **not always true**_ in Imperative languages

函數的返回值只依賴於其輸入值，這種特性就稱為 [Referential Transparency](https://en.wikipedia.org/wiki/Referential_transparency)

> The main difference, is, then, that if you call `f(x)` in math, you will **ALWAYS** get the same answer, but if you call `f'(x)` in C, the answer may not be the same - the same arguments **don't always** return the same output. A function in non-functional languages may not depend solely on the arguments you give them, but on other things in the program (enclosing scope, globals, builtins, state if you're using object orientation).

- 在數學的定義中，$y = f(x)$，當$x$相同則$y$必定得到相同的結果
	- 例如 $f(x) = xsin\pi$，不論何時回傳值皆為$0$
	- 在C函式中則除了傳入值外還受到其他東西影響
		- 全域變數
		- 記憶體內容
		- 已開啟的檔案
		- 其他變數、環境
	- 例如 `getpid`，雖然在同樣的process中必定得到相同的值，但在不同的process會得到不同的結果
	- 或考慮以下程式，不依賴任何傳入值但每次都會得到不同結果
```c
static int counter = 0;
int count() { return ++counter; }
```



#C語言 #Function 