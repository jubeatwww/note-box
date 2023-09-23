- C99 [6.2.5] _**Types**_

# Describe pointer type

^3a031b

- 
    - A pointer type may be derived from a [[Function Pointer#^818b62|function type]], an ==object type==, or an ==incomplete type==, called the referenced type. A pointer type describes an object whose value provides a reference to an entity of the referenced type. A pointer type derived from the referenced type T is sometimes called ‘‘pointer to T’’. The construction of a pointer type from a referenced type is called ‘‘pointer type derivation’’.
	    - 在C語言中 object type就是佔有一段記憶體空間的就叫[[Object]]，與其他語言的物件不是相同概念
    
    > 注意到術語！這是 C 語言只有 call-by-value 的實證，函式的傳遞都涉及到數值  
	> 這裡的 “incomplete type” 要注意看，稍後會解釋。要區分 `char []` 和 `char *`

# What is scalar type

^965799

- 
    - Arithmetic types and pointer types are collectively called ==scalar types==. Array and structure types are collectively called aggregate types.
	    - pointer type使用 `*` operator會報錯，但 `ptrA + ((int) ptrA * 1)` 卻是沒問題的
    > 注意 “scalar type” 這個術語，日後我們看到 `++`, `--`, `+=`, `-=` 等操作，都是對 scalar (純量)
    
    > [[來源](http://www.cyut.edu.tw/~cpyu/oldphweb/chapter3/page3.htm)] 純量只有大小，它們可用數目及單位來表示(例如溫度 = 30oC)。純量遵守算數和普通的代數法則。注意：純量有「單位」(可用 `sizeof` 操作子得知單位的「大小」)，假設 `ptr` 是個 pointer type，對 `ptr++` 來說，並不是單純 `ptr = ptr + 1`，而是遞增或遞移 1 個「單位」

# Describe ==incomplete type==

^913b49

- 
    - An array type of unknown size is an incomplete type. It is completed, for an identifier of that type, by specifying the size in a later declaration (with internal or external linkage). A structure or union type of unknown content is an incomplete type. It is completed, for all declarations of that type, by declaring the same structure or union tag with its defining content later in the same scope.
    
    > 這是 C/C++ 常見的 forward declaration 技巧的原理，比方說我們可以在標頭檔宣告 `struct GraphicsObject;` (不用給細部定義) 然後 `struct GraphicsObject *initGraphics(int width, int height);` 是合法的，但 `struct GraphicsObject obj;` 不合法
    

==因為沒有細部定義，所以是Incomplete Type，沒辦法建立實體，卻可以用指標==

# What array, function, and pointer types are collectively called?

^b7ace2

- Array, function, and pointer types are collectively called derived ==declarator types==. A declarator type derivation from a type T is the construction of a derived declarator type from T by the application of an array-type, a function-type, or a pointer-type derivation to T.

> 這句話很重要，貌似三個不相關的術語「陣列」、「函式」，及「指標」都歸類為 derived declarator types，讀到此處會感到驚訝者，表示不夠理解 C 語言

==array function pointer其實是一樣的，重點都在位址==


## 練習題
### 設定絕對地址為 `0x67a9` 的 32-bit 整數變數的值為 `0xaa6`，該如何寫？

^e8988b

```
*(int32_t * const) (0x67a9) = 0xaa6;
/* Lvalue */
```
==要先把0x67a9轉型成指標，再用\*取值 做更改==

## [[void pointer|void*]] 跟 `char*` 的關係

^bef57a

- A pointer to void shall have the same representation and alignment requirements as a pointer to a character type.
    
    > 關鍵描述！規範[[void pointer|void*]] 和 `char *` 彼此可互換的表示法

```
void *memcpy(void *dest, const void *src, size_t n);
```

==明明是處理子串的函式，去看規格書的原型卻是用void==
- [[Array Declaration#將純量轉換為 void*]]


#C語言 #Pointer #Types #C語言規格書 
