---
sidebar_position: "32"
tags:
  - Java
title: Operators - Arithmetic and Relations
slug: thinking-in-java/operators-arithmetic-and-relations
---
## 算數操作符 (Mathematical operators)

- 操作上基本上與大部分語言，尤其是 C/C++ 一致
```java
public class MathOps {
  public static void main(String[] args) {
    Random rand = new Random(47); // 創建一個隨機數生成器
    int i, j, k;
    // 從1到100隨機選取數值:
    j = rand.nextInt(100) + 1;
    print("j : " + j);
    k = rand.nextInt(100) + 1;
    print("k : " + k);
    i = j + k;
    print("j + k : " + i);
    i = j - k;
    print("j - k : " + i);
    i = k / j;
    print("k / j : " + i);
    i = k * j;
    print("k * j : " + i);
    i = k % j;
    print("k % j : " + i);
    j %= k;
    print("j %= k : " + j);

	// 浮點數運算測試:
    float u, v, w; // Applies to doubles, too
    v = rand.nextFloat();
    print("v : " + v);
    w = rand.nextFloat();
    print("w : " + w);
    u = v + w;
    print("v + w : " + u);
    u = v - w;
    print("v - w : " + u);
    u = v * w;
    print("v * w : " + u);
    u = v / w;
    print("v / w : " + u);
    
    // The following also works for char,
    // byte, short, int, long, and double:
    u += v;
    print("u += v : " + u);
    u -= v;
    print("u -= v : " + u);
    u *= v;
    print("u *= v : " + u);
    u /= v;
    print("u /= v : " + u);
  }
} /* Output:
j : 59
k : 56
j + k : 115
j - k : 3
k / j : 0
k * j : 3304
k % j : 56
j %= k : 3
v : 0.5309454
w : 0.0534122
v + w : 0.5843576
v - w : 0.47753322
v * w : 0.028358962
v / w : 9.940527
u += v : 10.471473
u -= v : 9.940527
u *= v : 5.2778773
u /= v : 9.940527
*/
```

### 一元加、減 operator (Unary minus and plus operators)

- 一元加號和減號運算符是基本的算術運算符，在Java中具有特定的用途
	- **一元減號** (`-`): 用於將數字的正負號反轉，例如，如果 `a = 5`，則 `-a` 會給出 `-5`
	- **一元加號** (`+`): 雖然大部分時間沒有顯著影響，它可以用來將較小的數據類型（如 `byte` 或 `short`）提升到 `int`
```java
x = -a;
x = a * -b;
x = a * (-b);
```

## 自動遞增、遞減(Auto increment and decrement)

- `a++`等價於 `a = a + 1`，`a--` 則等價於 `a = a - 1`
- 這個 operator 放在不同位置通常被稱為 **prefix** 、 **postfix**
    - **Preincrement** (`++number`): 這表示運算符位於變量前，操作是在使用變數值之前進行。這意味著在這個表達式中，`number`首先被增加1，然後新的值被用於表達式的其他部分
    - **Post-increment** (`number++`): 這表示運算符位於變量後，操作是在使用變數值之後進行。在這個表達式中，`number`的原始值首先被用於表達式的其他部分，然後`number`被增加1

```java
public class AutoInc {
  public static void main(String[] args) {
    int i = 1;
    print("i : " + i);
    print("++i : " + ++i); // Pre-increment
    print("i++ : " + i++); // Post-increment
    print("i : " + i);
    print("--i : " + --i); // Pre-decrement
    print("i-- : " + i--); // Post-decrement
    print("i : " + i);
  }
} /* Output:
i : 1
++i : 2
i++ : 2
i : 3
--i : 2
i-- : 2
i : 1
*/
```
> `++`、`--` 是除了 assignment 以外唯一具有 “副作用” 的 operator，會改變 operand 而不僅是使用自己的值


### 關係操作符 (Relational operators)

- relational operators 生成的是一個 **boolean** 結果，計算 operand 之間的關係

```java
public class Equivalence {
  public static void main(String[] args) {
    Integer n1 = Integer.valueOf(47);
    Integer n2 = Integer.valueOf(47);
    System.out.println(n1 == n2);
    System.out.println(n1 != n2);
  }
} /* Output:
false
true
*/
```
> 從 Java 9 開始，建議使用 `Integer.valueOf(47)` 來代替 `new Integer(47)` 來獲得 Integer 實例。這是因為 `Integer.valueOf()` 可以利用 Integer 快取來提高效能和降低內存使用
- 在比較兩個 Integer 物件時，若使用 `==` 和 `!=` 操作符，會比較兩個物件的 reference，而不是它們的數值
- 對於 primitives 可以直接使用 `==` 和 `!=`


```java
public class EqualsMethod {
  public static void main(String[] args) {
    Integer n1 = Integer.valueOf(47);
    Integer n2 = Integer.valueOf(47);
    System.out.println(n1.equals(n2));
  }
} /* Output:
true
*/
```
> 比較兩個 object 是否相同，要使用特殊的 method **equals()**


```java
class Value {
  int i;
}

public class EqualsMethod2 {
  public static void main(String[] args) {
    Value v1 = new Value();
    Value v2 = new Value();
    v1.i = v2.i = 100;
    System.out.println(v1.equals(v2));
  }
} /* Output:
false
*/
```
> 對於自訂的 class，**equals()** 的預設行為仍舊是比較 reference
