- `gcc -o <output> -Og -g <filename>`
	- `-Og`: 對debugger最佳化
	- `-g`: 加入debugger看得懂的訊息
- `gdb -q <binary>`  => 載入


- `p`: 
	- 印出結果
	- 可以拿來變更記憶體內容 => `p (&b[0])->v = {1, 2, 3}`
	- 執行期可以呼叫函式，改變執行順序、記憶體內容
```c
(gdb) p calendar
$19 = {{0 <repeats 31 times>} <repeats 12 times>}
(gdb) p memcpy(calendar, b, sizeof(b[0]))
$20 = 6296224
(gdb) p calendar
$21 = {{0, 1072693248, 0, 1073741824, 0, 1074266112, 0 <repeats 25 times>}, {0 <repeats 31 times>} <repeats 11 times>}
```

- `whatis`: 分析型態
- `disas`: 反組譯

#C語言 