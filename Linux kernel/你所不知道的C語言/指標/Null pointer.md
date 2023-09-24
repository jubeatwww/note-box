

考慮以下程式 (null.c)
```c
int main() { return (NULL == 0); }
```

用 gcc-7.2 編譯:
```c
null.c:1:22: 'NULL' undeclared (first use in this function)
int main() { return (NULL == 0); }
                     ^~~~
null.c:1:22: note: each undeclared identifier is reported only once for each function it appears in
```

表示 NULL 需要定義，我們加上 `#include <stddef.h>` 即可通過編譯

# 定義

- 依據 C99 規格 [6.3.2.3]
	- nothing requires the NULL **pointer** to have the numeric value of zero;
	- 這點導致實作上的歧異
- C99 規格 [6.3.2.3 - 3]
	- A “null **pointer** constant” does not have to be a “null **pointer**”
- 依據 C99 / C11 規格 **6.3.2.3** - 3
	- "An integer constant expression with the value 0, or such an expression cast to type void *, is called a null pointer constant.
	- If a null pointer constant is converted to a pointer type, the resulting pointer, called a null pointer, is guaranteed to compare unequal to a pointer to any object or function."
	- 在 caller 端轉換數值為 pointer type 時，就會變為 null pointer。
	- [空指標常數(null pointer constant)](https://www.ptt.cc/bbs/C_and_CPP/M.1461840781.A.192.html)
- 延伸 C99/C11 規格 **6.3.2.3** - 4
	- Conversion of a null pointer to another pointer type yields a null pointer of that type. Any two null pointers shall compare equal.
	- (void*)(uintptr_t) 0 本質上是個 null pointer，而 NULL 是 null ptr，以 C99 的觀點
- C99 規格 6.7.8 Initialization
	- 當透過 malloc() 建立的物件經由 free() 釋放後，應該將指標設定為 NULL
	- 避免 doubly-free
- `free()` 傳入 NULL 是安全的

# 兩個NULL pointer是否能比較
- 依據 C99 規格 6.3.2.3 - 3
	- “An integer constant expression with the value 0, or such an expression cast to type `void*`, is called a null pointer constant. If a null pointer constant is converted to a pointer type, the resulting pointer, called a null pointer, is guaranteed to compare **unequal** to a pointer to any object or function.”
		- **_compare unequal_** 意味著 “a == b is false”，換句話說：“a != b is true”
- 接下來 C99 規格在 6.3.2.3 又說:
	- Conversion of a null pointer to another pointer type yields a null pointer of that type. Any two null pointers shall compare equal.
	- 對 pointer type 作比較，又是另外的狀況
- 依據 C99 規格 6.5.8 - 5
	- “Comparison of **pointers** that do not point to the same object or to members of the same aggregate is undefined.”
	- 需要注意的是 condition (傳入 assert() 的參數也是) 依據 C99 規範是 “a scalar type”，而pointers 也是 scalar

# C 語言 offsetof 巨集的定義

`<stddef.h>` 定義了 [offsetof 巨集](https://en.wikipedia.org/wiki/Offsetof)，取得 struct 特定成員 (member) 的位移量。傳統的實作方式如下:
```c
#define offsetof(st, m) ((size_t)&(((st *)0)->m))
```

> 根據C99規格，這個其實是undefined behavior
> 後來便改用builtin functions

```c
#define offsetof(st, m) __builtin_offsetof(st, m)
```

Linux 核心延伸 offsetof，提供 container_of 巨集，作為反向的操作，也就是給予成員位址和型態，反過來找 struct 開頭位址：
```c
#define container_of(ptr, type, member) ({ \
    const typeof(((type *) 0)->member) *__mptr = (ptr); \
    (type *) ((char *)__mptr - offsetof(type,member) );})
```

應用方式: [[container_of]]

延伸閱讀
- [C++11- NULLPTR != NULL](https://cppturkiye.wordpress.com/2016/02/05/c11-nullptr-null/)
- [Null Pointer Dereferencing Causes Undefined Behavior](https://pvs-studio.com/en/blog/posts/cpp/0306/)



#C語言 #Pointer 