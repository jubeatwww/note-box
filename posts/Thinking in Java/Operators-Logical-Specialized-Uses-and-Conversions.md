---
sidebar_position: "34"
tags:
  - Java
title: Operators - Specialized Uses and Conversions
slug: /thinking-in-java/operators-logical-specialized-uses-and-conversions
---
## 三元操作符 (Ternary if-else operator)

```
booleanExp ? value0 : value1
```
- 當 `booleanExp` 為 **true** 時，計算 `value0`，否則計算 `value1`

```java
public class TernaryIfElse {
  static int calculateUsingTernary(int i) {
    return i < 10 ? i * 100 : i * 10;
  }
  static int calculateUsingIfElse(int i) {
    if(i < 10)
      return i * 100;
    else
      return i * 10;
  }
  public static void main(String[] args) {
    print(calculateUsingTernary(9));  // 使用三元操作符計算，輸出: 900
    print(calculateUsingTernary(10)); // 使用三元操作符計算，輸出: 100
    print(calculateUsingIfElse(9));   // 使用標準if-else計算，輸出: 900
    print(calculateUsingIfElse(10));  // 使用標準if-else計算，輸出: 100
  }
} /* Output:
900
100
900
100
*/
```

:::warning
過度使用三元操作符雖然可以簡化代碼，但可能會降低代碼的可讀性，尤其在涉及復雜邏輯時更應避免使用
:::
- `calculateUsingTernary` 和 `calculateUsingIfElse` 兩個函數展示了三元操作符和標準 if-else 的等價性

## 字串操作符 + 和 += (String operator + and +=)

- 在 Java 中有特殊的用途: 連接不同的 String

```java
public class StringOperators {
  public static void main(String[] args) {
    int x = 0, y = 1, z = 2;
    String s = "x, y, z ";
    System.out.println(s + x + y + z);
    System.out.println(x + " " + s); // Converts x to a String
    s += "(summed) = "; // Concatenation operator
    System.out.println(s + (x + y + z));
    System.out.println("" + x); // Shorthand for Integer.toString()
  }
} /* Output:
x, y, z 012
0 x, y, z
x, y, z (summed) = 3
0
*/
```

:::info
輸出 `012` 而非 `3`，因為它是作為字符串依次加起來的結果。這也展示了如何利用字符串操作來省略显式的使用 `toString()` 方法
:::
- 字符串操作符 `+` 用於連接字符串和其他類型的數據，如果操作的第一個元素是字符串，則後續的數字會被自動轉換為字符串然後進行連接
- 當 `+` 操作符首個操作數為非字符串類型時，則按照操作數從左到右的順序進行評估和轉換
- `+=` 是連接操作符，它會改變原有字符串的內容，將指定內容追加到原字符串之後
- 編譯器自動將雙引號內的字串轉換為 String 對象

## 類型轉換操作符 (Casting operators)

- 在適當的時候 Java 會將類型轉換成另一種
```java
public class Casting {
  public static void main(String[] args) {
    int i = 200;
    long lng = (long)i;    // 無需顯式轉換，因為 int 到 long 是 Widening
    lng = i;               // "Widening," so cast not really required
    long lng2 = (long)200; // 直接將字面量轉為 long
    lng2 = 200;            // 擴展轉換，自動處理
    // A "narrowing conversion":
    i = (int)lng2; // Cast required，因為 long 到 int 可能導致資料丟失
  }
} 
```

- _窄化轉換(narrowing conversion)_ 可能會造成資料丟失，因此編譯器要求顯式轉換
- _擴展轉換(widening conversion)_，則不需要，因為肯定可以容納新類型而不會造成數據丟失
- Java 允許所有基本類型轉換，除了 **boolean**
### Truncation and rounding
- 進行 narrowing conversion 時必須注意 truncation 和 rounding 問題
	- 如從浮點數到整數，會發生截斷
```java
public class CastingNumbers {
  public static void main(String[] args) {
	double above = 0.7, below = 0.4;
	float fabove = 0.7f, fbelow = 0.4f;
	print("(int)above: " + (int)above);
	print("(int)below: " + (int)below);
	print("(int)fabove: " + (int)fabove);
	print("(int)fbelow: " + (int)fbelow);
  }
} /* Output:
(int)above: 0
(int)below: 0
(int)fabove: 0
(int)fbelow: 0
*/
```
- 浮點數轉整數時通常發生截斷(truncation)，要進位則需使用 `Math.round()`

```java
public class RoundingNumbers {
  public static void main(String[] args) {
	double above = 0.7, below = 0.4;
	float fabove = 0.7f, fbelow = 0.4f;
	print("Math.round(above): " + Math.round(above));
	print("Math.round(below): " + Math.round(below));
	print("Math.round(fabove): " + Math.round(fabove));
	print("Math.round(fbelow): " + Math.round(fbelow));
  }
} /* Output:
Math.round(above): 1
Math.round(below): 0
Math.round(fabove): 1
Math.round(fbelow): 0
*/
```
### 提升 (Promotion)

- 表達式中不同數據類型的操作會導致數據類型的提升
	- 通常表達式中最大的數據類型會決定最終結果的數據類型
    - `float * double ⇒ double`
    - `long * int ⇒ long`
- 要將結果分配給較小的類型，則需要顯式進行類型轉換

## Java 沒有 sizeof

- 在 C/C++ 中，`sizeof()` 函數的主要用途是為了支持不同位元的系統間的程式碼移植。它能提供在特定平台上各種數據類型的大小，這對於直接管理記憶體和優化儲存非常重要
- 相對而言，Java 的數據類型在所有機器中都是相同的大小
	- 這是因為 Java 被設計成一種可攜式語言，其中 JVM 負責抽象硬件的具體細節
	- 在 Java 中不需要 `sizeof()` 函數，因為數據類型的大小並不會隨著運行的系統而改變，這減少了需要考慮的平台相關問題，使得 Java 應用更容易跨平台運行