
```c
int a[3];
struct { double v[3]; double length; } b[17];
int calendar[12][31];
```

找出
```c
sizeof(calendar) = ? sizeof(b) = ?
```


- 透過[[GDB]] debug
> 注意 `sizeof` 在C裡面是一個operator非function

```c
(gdb) p sizeof(calendar)
$1 = 1488
(gdb) print 1488 / 12 / 31
$2 = 4
(gdb) p sizeof(b)
$3 = 544
```

- `$2 == sizeof(int)`
- `$3 / 17 / 4 == 8 == sizeof(double)`

---
- 型態分析
```c
(gdb) whatis calendar
type = int [12][31]
(gdb) whatis b[0]
type = struct {...}
(gdb) whatis &b[0]
type = struct {...} *
```

---
- 觀察記憶體
```c
(gdb) x/4 b
0x601060 <b>: 0x00000000 0x00000000 0x00000000 0x00000000
(gdb) p b
$4 = {{
  v = {0, 0, 0},
  length = 0
} <repeats 17 times>}
```

---
- 實驗
```c
(gdb) p &b
$5 = (struct {...} (*)[17]) 0x601060 <b>
(gdb) p &b+1
$6 = (struct {...} (*)[17]) 0x601280 <a>
```
- `&b + 1`不是加一個byte而加入了一整個struct array的大小
	- `sizeof(b[0])  == 32`
	- `sizeof(b[0]) * 17 == 544`
	- 這邊的 `+1`取決於所屬的型態
- `p &(b[0]) + 1`  => `(struct {...} *) 0x601080 <b+32>`
	- 只加了一個element的大小
	- `&b[0] + 1`的 `+1` 是遷移一個 `b[0]`占用的空間
---

- 修改b的內容
```c
(gdb) p &b[0]
$10 = (struct {...} *) 0x601060 <b>
(gdb) p (&b[0])->v
$11 = {0, 0, 0}
```

```c
(gdb) p (&b[0])->v = {1, 2, 3}
$12 = {1, 2, 3}
(gdb) p b[0]
$13 = {
  v = {1, 2, 3},
  length = 0
}
```
- 此處 `x/4 b`，會是一串看不懂的數字
	- 從這邊理解`(float) 7.0` 和 `(int) 7`的不同之處
	-  `whatis (&b[0])->v[0]` => `type = double`
	- `p sizeof (&b[0])->v[0]` => 8

---
```c
(gdb) p &(&b[0])->v[0]
$15 = (double *) 0x601060 <b>
(gdb) p (int *) &(&b[0])->v[0]
$16 = (int *) 0x601060 <b>
(gdb) p *(int *) &(&b[0])->v[0]
$17 = 0
(gdb) p *(double *) &(&b[0])->v[0]
$18 = 1
```
- Linux x86_64 採用 [LP64 data model](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models)，`double` 依據 C 語言規範，至少要有 64-bit 長度
	- 轉型成 `int*`時取值變會得到0
> 只有windows使用LLP64，long int是32位

```c
(gdb) x/4 (int *) &(&b[0])->v[0]
0x601060 <b>: 0x00000000 0x3ff00000 0x00000000 0x40000000
```

---

## 將純量轉換為 void*
```c
(gdb) p calendar
$19 = {{0 <repeats 31 times>} <repeats 12 times>}
(gdb) p memcpy(calendar, b, sizeof(b[0]))
$20 = 6296224
(gdb) p calendar
$21 = {{0, 1072693248, 0, 1073741824, 0, 1074266112, 0 <repeats 25 times>}, {0 <repeats 31 times>} <repeats 11 times>}
```

```c
(gdb) p (void *) 6296224
$22 = (void *) 0x6012a0 <calendar>
```
> 把純量轉型成pointer type，gdb就知道要去找他指的位置是什麼

```c
(gdb) p &calendar
$23 = (int(*)[12][31]) 0x6012a0 <caleandar>
```

## 從 calendar 把 `{1, 2, 3}` 內容取出

```c
(gdb) p *(double *) &calendar[0][0]
$23 = 1
(gdb) p *(double *) &calendar[0][2]
$24 = 2
(gdb) p *(double *) &calendar[0][4]
$25 = 3
```
> 如果直接 `p calendar[0][0]` 會看到0，因為calendat本身是 `int`, 只取前四個bytes就會是0
> 用不同的pointer type會有==不同的移動量==



#C語言 #Pointer #Array 