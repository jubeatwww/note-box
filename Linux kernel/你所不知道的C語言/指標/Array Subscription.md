- C 提供操作多維陣列的機制 (C99 [6.5.2.1] **_Array subscripting_**)，但實際上只有一維陣列的資料存取
    
    - 對應到線性記憶體
    - Successive subscript operators designate an element of a multidimensional array object. If E is an n-dimensional array (n ≥ 2) with dimensions i × j × … × k, then E (used as other than an lvalue) is converted to a pointer to an (n − 1)-dimensional array with dimensions j × … × k. If the unary * operator is applied to this pointer explicitly, or implicitly as a result of subscripting, the result is the pointed-to (n − 1)-dimensional array, which itself is converted into a pointer if used as other than an lvalue. It follows from this that arrays are stored in row-major order (last subscript varies fastest)
    - Consider the array object defined by the declaration `int x[3][5];` Here x is a 3 × 5 array of ints; more precisely, x is an array of three element objects, each of which is an array of five ints. In the expression `x[i]`, which is equivalent to `(*((x)+(i)))`, x is first converted to a pointer to the initial array of five ints. Then i is adjusted according to the type of x, which conceptually entails multiplying i by the size of the object to which the pointer points, namely an array of five int objects. The results are added and indirection is applied to yield an array of five ints. When used in the expression `x[i][j]`, that array is in turn converted to a pointer to the first of the ints, so `x[i][j]` yields an int.

- array subscripting 在**編譯時期**只確認以下兩件事:
    - 得知 size
    - 取得陣列第 0 個 (即最初的一項) 元素的指標
- 前兩者以外的操作，都透過 pointer
    - array subscripting => syntax sugar

#C語言 #Pointer #Array 