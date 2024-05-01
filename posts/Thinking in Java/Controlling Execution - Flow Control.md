---
sidebar_position: "41"
tags:
  - Java
title: Controlling Execution - Flow Control
slug: /thinking-in-java/controlling-execution-flow-control
---
## true and false

- 所有的條件語句都利用條件表達式的 true 或 false 來決定執行路徑
- 在 Java 中，不允許使用數字作為 **boolean** 值:
    - 相較於其他一些語言，例如 C 語言，其中 `false` 被定義為 0，`true` 為任何非 0 數值
    - 若要在 Java 中使用非 **boolean** 值作為條件，如 `if(a)`，則必須通過條件表達式將其轉換為 **boolean**，例如 `if(a != 0)`

## if-else

以下是 Java 中使用 `if-else` 結構的基本語法：

```java
if (BooleanExpression)
  statement
// 或
if (BooleanExpression)
    statement
else
    statement
```
> `statement` 這裡指的是任何有效的 Java 單行執行語句


實際應用示例：
```java
public class IfElse {
  static int result = 0;
  static void test(int testval, int target) {
    if(testval > target)
      result = +1;
    else if(testval < target)
      result = -1;
    else
      result = 0; // Match
  }
  public static void main(String[] args) {
    test(10, 5);
    System.out.println(result);
    test(5, 10);
    System.out.println(result);
    test(5, 5);
    System.out.println(result);
  }
} /* Output:
1
-1
0
*/
```


## 迭代 (Iteration)

迭代指的是在程式中重複執行某些語句，直到控制條件的 _Boolean Expression_ 為 **false** 時停止：

```
while(Boolean-expression)
  statement
```

以下是一個使用 `while` 迴圈的實際範例：
```java
public class WhileTest {
  static boolean condition() {
    boolean result = Math.random() < 0.99;
    System.out.print(result + ", ");
    return result;
  }
  public static void main(String[] args) {
    while(condition())
      System.out.println("Inside 'while'");
    System.out.println("Exited 'while'");
  }
}
```

- `condition()` 函數會產生一個隨機數並檢查是否小於 0.99。如果是，則返回 `true`，迴圈繼續執行，印出 "Inside 'while'"
 - 當隨機數終於達到或超過 0.99，`condition()` 返回 `false`，迴圈結束，然後印出 "Exited 'while'"
- **while** 表達式的意思為: “只要 `condition()` 回傳 **true**，就重複執行“

### do-while

`do-while` 迴圈是一種後測迴圈，即使條件初始不成立，迴圈內的語句也會至少被執行一次
```java
do
  statement
while (BooleanExpression);
```

實例示例：
```java
public class DoWhileExample {
	public static void main(String[] args) {
		int count = 0;
		do {
			System.out.println("Count is: " + count);
			count++;
		} while (count < 5);
	}
}
```

- 在這個範例中，`do` 區塊內的語句首先執行一次，然後根據 `while` 條件表達式（`count < 5`）檢查是否需要繼續迴圈
	- 因為 `count` 從 0 開始，迴圈會執行五次，每次都會印出 `count` 的值
	- 然後 `count` 進行加一操作，直到其值達到 5 為止

### for

- `for` 迴圈是最常用於迭代的結構：
```java
for (initialization; BooleanExpression; step)
  statement
```

- 解釋
    - initialization: 初始化表達式，設定迴圈起始的狀態
    - BooleanExpression: 控制迴圈繼續的 **boolean** 條件表達式
    - step: 在每次迴圈結束時執行的更新表達式，用於修改迴圈變數，推動迴圈向前進
:::info
任何這三個部分都可以留空，這在需要的情況下提供了靈活性
:::

```java
public class ListCharacters {
  public static void main(String[] args) {
    for(char c = 0; c < 128; c++)
      if(Character.isLowerCase(c))
        System.out.println("value: " + (int)c + " character: " + c);
  }
} /* Output:
value: 97 character: a
value: 98 character: b
value: 99 character: c
value: 100 character: d
value: 101 character: e
value: 102 character: f
value: 103 character: g
value: 104 character: h
value: 105 character: i
value: 106 character: j
...
*/
```
> 這段程式碼示範了遍歷 ASCII 字符集中的小寫字母，並輸出其整數值和字符形式

#### 逗號操作符 (The comma operator)

- Java 內唯一用到 comma operator的地方只有 **for**
- `for` 迴圈的三個部分（初始化、條件、步進）可以使用逗號操作符來包含多個語句，這些語句會按順序執行，但彼此間不互相影響

```java
public class CommaOperator {
  public static void main(String[] args) {
    for(int i = 1, j = i + 10; i < 5; i++, j = i * 2) {
      System.out.println("i = " + i + " j = " + j);
    }
  }
} /* Output:
i = 1 j = 11
i = 2 j = 4
i = 3 j = 6
i = 4 j = 8
*/
```
- 迴圈內同時追蹤兩個變數 `i` 和 `j`
	- `i` 每次迴圈遞增
	- `j` 則是在每次迴圈結束時更新為 `i` 的兩倍

## foreach

- Java SE5 引入了一種更簡潔的 `for` 語法，專為 array 和 collection 設計，稱為 `foreach` 迴圈
- 使用此語法時，無需手動建立和管理索引變數，迴圈會自動遍歷所有元素

```java
public class ForEachFloat {
  public static void main(String[] args) {
    Random rand = new Random(47);
    float f[] = new float[10];
    for(int i = 0; i < 10; i++)
      f[i] = rand.nextFloat();
    for(float x : f)
      System.out.println(x);
  }
} /* Output:
0.72711575
0.39982635
0.5309454
0.0534122
0.16020656
0.57799757
0.18847865
0.4170137
0.51660204
0.73734957
*/
```

- 在 `foreach` 迴圈中，迴圈變數 `x` 會自動被 assign 數組 `f` 中的每一個元素。迴圈每進行一次，`x` 就更新為數組的下一個元素。
- foreach 可以用於任何 **Iterable** object