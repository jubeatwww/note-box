---
sidebar_position: "3"
tags:
  - Java
title: Everything Is an Object - Class
---
## Java Class Fundamentals

- class的含意: 我準備告訴你一種新類型的 object 看起來像什麼樣子 (所以中國都會直翻為 _類_ )
- 基本定義方法:
    ```
    class ATypeName { /* Class body goes here */ }
    ```

- 基本使用方法:
    ```
    ATypeName a = new ATypeName();
    ```

### Fields and methods

- 一旦定義了 class，就能對它設置兩種元素
    - field: 台灣一般稱欄位、中國稱為字段
    - methods: 方法 (成員函數)
- field 可以是任何類型的 object 或基本類型
    - 如果 field 是某個 object 的reference 就需要初始化
    - 每個 object 都有自己的儲存 field 的空間，不能在 object 間共享

```java
class DataOnly {
  int i;
  double d;
  boolean b;
}

DataOnly data = new DataOnly();
```

- 可以對 field 賦值，而需要先知道如何取用成員 object 的 reference
    - 在 object reference 的名稱之後接一個句點
```java
objectReference.memeber

// 賦值
data.i = 47;
data.d = 1.1;
data.b = false;
```

- object 包含其他 object 時，只要連續使用句點即可
```java
myPlane.leftTank.capacity = 100;
```

### 基本成員 default 值

- 若基本成員沒有被初始化，Java 會確保它獲得 default 值來初始化

| primitive | default         |
| --------- | --------------- |
| boolean   | false           |
| char      | ‘\u0000’ (null) |
| byte      | (byte) 0        |
| short     | (short) 0       |
| int       | 0               |
| long      | 0L              |
| float     | 0.0f            |
| double    | 0.0d            |
- 非 class 內的 local variable 並不適用 class 的初始化，編譯時會出現錯誤


## Methods, Arguments, and Return Values

- 許多語言例如 C/C++ 使用 _函數_ 來命名子程序，而 Java 常用 _方法_ 來表示
- method 基本組成包含
    - name
    - 參數 (argument): 要傳給方法的訊息的類型和名稱
    - 回傳型別 (return type): 調用方法後回傳的值的型別
    - body
```java
ReturnType methodName(arg1, arg2, arg3); 
```
- 方法名和參數列表合起來稱為方法的 **signature**，是方法的唯一標識
- Java 中的 method 只能做為 class 的一部分建立，通過 object 才能調用
- 調用 class 上不具備的方法會得到錯誤消息
- 調用方法為列出對象名，接著句點，後面接上方法名和參數列表
```java
objectName.methodName(arg1, arg2, arg3);
```
- 假設有方法 **f()** ，不帶任何參數，回傳類型是 **int**，有個名為 **a** 的 object
```java
int x = a.f();
// 返回值的類型要與 x 的類型兼容
```
- 這種調用方法通常被稱為 **發送消息給對象**，上述的例子中消息是 **f()**，對象是 **a**
### 參數列表

- 參數列表指定要傳遞給 method 怎樣的訊息，跟其他 Java 的訊息一樣，都採用 object 形式
- 參數列表中必須指定每個要傳遞的 object 的類型跟名字
- 這邊的資料傳遞實際上也都是 reference (primitives 為例外)
- 傳遞的訊息引用的類型必須正確，參數設為 **String**，就必須傳遞 **String**，否則編譯器會報錯

#### Example
```java
int storage(String s) {
  return s.length() * 2;
}
```
- 參數類型為 **String** ，參數名為 **s**
- 將 **s** 傳遞給這個 method 之後就可以當成 object 一樣進行處理，在這邊 **s** 的 **length()**方法被調用
- **return**
    - 代表 “已經做完，離開此方法“
    - 如果此方法產生了一個值，這個值要放在 **return** 後面，在這個例子透過回傳值透過計算 `s.length() * 2` 得到
- 不想回傳任何值可以回傳 **void (空)**
```java
Boolean flag() { return true; }
double naturalLogBase() { return 2.718; }
void nothing() { return; }
void nothing() {}
```
- 如果回傳型別為 **void**，**return** 關鍵字只是用來退出 method
- 如果回傳型別不為 **void**，則編譯器會強制回傳正確類型的值

