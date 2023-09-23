[規格書](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf)


## `&`在C語言中該怎麼唸?

- `&` 不要都念成 and，涉及指標操作的時候，要讀為 “address of”
    
    - C99 標準 [6.5.3.2] Address and indirection operators 提到 ‘==&==’ address-of operator
# 簡述C語言中的Object

^207827

- C99 [3.14]
	- - region of data storage in the execution environment, the contents of which can represent values
	- 在 C 語言的物件就指在執行時期，==資料==儲存的區域，可以明確表示數值的內容
	- 很多人誤認在 C 語言程式中，(int) 7 和 (float) 7.0 是等價的，其實以資料表示的角度來看，這兩者截然不同，前者對應到二進位的 “111”，而後者以 IEEE 754 表示則大異於 “111”

# 簡述Storage durations of objects

^7ab677

- C99 [6.2.4]
    
    - An object has a storage duration that determines its lifetime. There are three storage durations: static, automatic, and allocated.
    
    > 注意生命週期 (lifetime) 的概念，中文講「初始化」時，感覺像是「盤古開天」，很容易令人誤解。其實 initialize 的[英文意義](http://dictionary.reference.com/browse/initialize)很狹隘： “to set (variables, counters, switches, etc.) to their starting values at the beginning of a program or subprogram.”
    
    - The lifetime of an object is the portion of program execution during which storage is guaranteed to be reserved for it. An object exists, has a constant address and retains its last-stored value throughout its lifetime. If an object is referred to outside of its lifetime, the behavior is undefined.
    
    > 在 object 的生命週期以內，其存在就意味著有對應的常數記憶體位址。注意，C 語言永遠只有 call-by-value
    
    - The value of a pointer becomes indeterminate when the object it points to reaches the end of its lifetime.
    
    > 作為 object 操作的「代名詞」(alias) 的 pointer，倘若要在 object 生命週期以外的時機，去取出 pointer 所指向的 object 內含值，是未知的。考慮先做 `ptr = malloc(size); free(ptr);` 倘若之後做 `*ptr`，這個 allocated storage 已經超出原本的生命週期
    
    - An object whose identifier is declared with no linkage and without the storage-class specifier static has automatic storage duration.


#C語言  #Pointer #Object