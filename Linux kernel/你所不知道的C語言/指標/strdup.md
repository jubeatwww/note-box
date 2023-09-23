依據 man page [strdup(3)](http://linux.die.net/man/3/strdup) :

> The strdup() function returns a pointer to a new string which is a duplicate of the string s. Memory for the new string is obtained with [malloc(3)](http://linux.die.net/man/3/malloc), and can be freed with [free(3)](http://linux.die.net/man/3/free).


- `strdup`會呼叫 [[pointer與字串的處理#^bff8bd|`malloc`]]，來配置字串的新空間
> 需要特別注意記憶體分配帶來的問題

```c

char *p, *q;
p = strdup("xyz");

q = p;
q[1] = 'A'; // p would point to "xAz" too
```


#C語言 #Pointer #Array #String 