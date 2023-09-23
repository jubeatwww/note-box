
# 指標解說

- [C 語言: 超好懂的指標](https://kopu.chat/2017/05/15/c%E8%AA%9E%E8%A8%80-%E8%B6%85%E5%A5%BD%E6%87%82%E7%9A%84%E6%8C%87%E6%A8%99%EF%BC%8C%E5%88%9D%E5%AD%B8%E8%80%85%E8%AB%8B%E9%80%B2%EF%BD%9E/) ^3dbf74
- [Everything you need to know about pointers in C](https://boredzo.org/pointers/)

# 指標使用上的疑惑

### 該如何解釋 [qsort(3)](https://linux.die.net/man/3/qsort) 的參數和設計考量呢？

```
void qsort(void *base, size_t nmemb, size_t size,
           int (*compar)(const void *, const void *));
```


### 為何我看到的程式碼往往寫成類似下面這樣？

```
struct list **lpp;
for (lpp = &list; *lpp != NULL; lpp = &(*lpp)->next)
```


#

#C語言 #Pointer