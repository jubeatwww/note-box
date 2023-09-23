- `void`最早在C語言中不存在，function若沒有標註return type就一率是`int(0)`
- `void main() {}` 可以編譯過
	- `main() {}`，也可以編譯，但會有warning提示 `return type defaults to 'int'`
- `void*`必須==explicit(顯式)==或強制轉型才能存取object，否則會報錯
	- 我們無法直接對 `void *` 做數值操作
```
void *p = ...;
void *p2 = p + 1; /* what exactly is the size of void? */
```
- 換句話說， `void *` 存在的目的就是為了強迫使用者使用 ==顯式轉型== 或是 ==強制轉型==，以避免 Undefined behavior 產生

```
int main() {
	char *X = 0;
	void *Y = 0;
	char c = *X;
	char d = *Y; // compile error

	char d = *(char *) Y; // ok
	char d = *(double *) Y; // 也可以編譯過，但alignment會不一致
}
```
- [[void pointer的問題]]


#C語言 #Pointer #void