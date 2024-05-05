---
tags:
  - Java
sidebar_position: "6"
slug: /thinking-in-java/Initialization-n-cleanup-variable-argument-lists
---

- 利用大括號建立 array 的形式提供一種方便的語法來創建 `Object[]` 的實例，以獲得與 C 的可變參數列表一樣的效果 (C 中稱為 `varargs`)
- 可以創建以 object array 為參數的方法

```java
class A {}

public class VarArgs {
  static void printArray(Object[] args) {
    for(Object obj : args)
      System.out.print(obj + " ");
    System.out.println();
  }
  public static void main(String[] args) {
    printArray(new Object[]{
      new Integer(47), new Float(3.14), new Double(11.11)
    });
    printArray(new Object[]{"one", "two", "three" });
    printArray(new Object[]{new A(), new A(), new A()});
  }
} /* Output: (Sample)
47 3.14 11.11
one two three
A@1a46e30 A@3e25a5 A@19821f
*/
```

- 可以看到 `print()` 使用了 `Object` 的 array 作為參數，並使用 foreach 語法遍歷
- 標準 Java library 中的 class 能輸出有意義的內容，但這裡建立的 object 只會印出 class 名稱跟著 `@` 及多個16進位的數字，如果沒有定義 `toString()` 方法，預設就會印出這樣的輸出

## varargs的導入

- Java SE5 之後，可變參數列表的特性才加進來
- varargs 特性允許方法直接接收不定數量的參數，無需顯式創建 array。這使得調用方法時更加靈活和簡潔

```java
public class NewVarArgs {
  static void printArray(Object... args) {
    for(Object obj : args)
      System.out.print(obj + " ");
    System.out.println();
  }
  public static void main(String[] args) {
    // Can take individual elements:
    printArray(new Integer(47), new Float(3.14),
      new Double(11.11));
    printArray(47, 3.14F, 11.11);
    printArray("one", "two", "three");
    printArray(new A(), new A(), new A());
    // Or an array:
    printArray((Object[])new Integer[]{ 1, 2, 3, 4 });
    printArray(); // Empty list is OK
  }
} /* Output: (75% match)
47 3.14 11.11
47 3.14 11.11
one two three
A@1bab50a A@c3c749 A@150bd4d
1 2 3 4
*/
```
- 有了可變參數就不用顯式的寫 array 語法了，當指定參數時編譯器會自動幫你填充 array
- 你得到的還是一個 array，所以 `print()`才可以用 foreach 來迭代
- 程式中的 `Integer` array (使用自動包裝建立的)，被轉型成 `Object` array，以便移除編譯器警告，編譯器會發現它是一個 array，所以不會在其上做任何轉換。所以如果有一組事物，可以當列表傳遞，而如果已經有一個 array，可以當可變參數列表來接受

```java
public class OptionalTrailingArguments {
  static void f(int required, String... trailing) {
    System.out.print("required: " + required + " ");
    for(String s : trailing)
      System.out.print(s + " ");
    System.out.println();
  }
  public static void main(String[] args) {
    f(1, "one");
    f(2, "two", "three");
    f(0);
  }
} /* Output:
required: 1 one
required: 2 two three
required: 0
*/
```
- 將 0 個參數傳遞給可變參數列表是可接受的
- 也可以使用具有 `Object` 之外類型的可變參數列表
- 可變參數列表可使用任何類型，包括基本類型

```java
public class VarargType {
  static void f(Character... args) {
    System.out.print(args.getClass());
    System.out.println(" length " + args.length);
  }
  static void g(int... args) {
    System.out.print(args.getClass());
    System.out.println(" length " + args.length);
  }
  public static void main(String[] args) {
    f('a');
    f();
    g(1);
    g();
    System.out.println("int[]: " + new int[0].getClass());
  }
} /* Output:
class [Ljava.lang.Character; length 1
class [Ljava.lang.Character; length 0
class [I length 1
class [I length 0
int[]: class [I
*/
```

### 自動包裝與可變參數
- 可變參數列表與自動包裝機制可以和諧共處
```java
public class AutoboxingVarargs {
  public static void f(Integer... args) {
    for(Integer i : args)
      System.out.print(i + " ");
    System.out.println();
  }
  public static void main(String[] args) {
    f(new Integer(1), new Integer(2));
    f(4, 5, 6, 7, 8, 9);
    f(10, new Integer(11), 12);
  }
} /* Output:
1 2
4 5 6 7 8 9
10 11 12
*/
```
- 可以將單一的參數列表中將類型混和在一起，而自動包裝機制會將 `int` 提升為 `Integer`
### 可變參數與 method overloading
- 可變參數列表會讓 overloading 的情況變複雜
```java
public class OverloadingVarargs {
  static void f(Character... args) {
    System.out.print("first");
    for(Character c : args)
      System.out.print(" " + c);
    System.out.println();
  }
  static void f(Integer... args) {
    System.out.print("second");
    for(Integer i : args)
      System.out.print(" " + i);
    System.out.println();
  }
  static void f(Long... args) {
    System.out.println("third");
  }
  public static void main(String[] args) {
    f('a', 'b', 'c');
    f(1);
    f(2, 1);
    f(0);
    f(0L);
    //! f(); // Won't compile -- ambiguous
  }
} /* Output:
first a b c
second 1
second 2 1
second 0
third
*/
```
- 在每一種情況編譯器都會使用自動包裝機制來匹配 overloading methods，然後調用最明確匹配到的 method，但在不使用參數時，編譯器無法知道該調用哪一個
- 透過在某個方法中增加一個非可變參數來解決問題
```java
// {CompileTimeError} (Won't compile)

public class OverloadingVarargs2 {
  static void f(float i, Character... args) {
    System.out.println("first");
  }
  static void f(Character... args) {
    System.out.print("second");
  }
  public static void main(String[] args) {
    f(1, 'a');
    f('a', 'b');
  }
}
```
- 這實際上還是會出問題，手動編譯會得到錯誤訊息
> _**reference to f is ambiguous, both method f(float,java.lang.Character...) in OverloadingVarargs2 and method f(java.lang.Character...) in OverloadingVarargs2 match**_

- 幫兩個都加上不可變參數就可以了
```java
public class OverloadingVarargs3 {
  static void f(float i, Character... args) {
    System.out.println("first");
  }
  static void f(char c, Character... args) {
    System.out.println("second");
  }
  public static void main(String[] args) {
    f(1, 'a');
    f('a', 'b');
  }
} /* Output:
first
second
*/
```