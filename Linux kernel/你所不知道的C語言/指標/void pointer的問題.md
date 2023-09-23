依據 C99 規格 6.3.2.3:8 [ Pointers ]

> A pointer to a function of one type may be converted to a pointer to a function of another type and back again; the result shall compare equal to the original pointer. If a converted pointer is used to call a function whose type is not compatible with the pointed-to type, the behavior is undefined.

換言之，C99 不保證 pointers to data (in the standard, “objects or incomplete types” e.g. `char *` or `void *`) 和 pointers to functions 之間相互轉換是正確的

- 可能會招致 undefined behavior (UB)
- 注意：C99 規範中，存在一系列的 UB，詳見 [未定義行為篇](https://hackmd.io/@sysprog/c-undefined-behavior)

> 延伸閱讀: [The Lost Art of Structure Packing](http://www.catb.org/esr/structure-packing/)

#C語言 #Pointer #void 