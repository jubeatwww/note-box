- 指標本身不可變更 (Constant pointer to variable): `const` 在 `*` 之後
>` char * const pContent;`

- 指標所指向的內容不可變更 (Pointer to constant): `const` 在 `*` 之前
> `const char * pContent;  `
> `char const * pContent;`

- 兩者都不可變更
>` const char * const pContent;`


#C語言 #Pointer 