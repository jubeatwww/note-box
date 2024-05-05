---
sidebar_position: "1"
tags:
  - Java
slug: /thinking-in-java/Initialization-n-cleanup-constructing-objects
---
## 用 constructor 確保初始化
### 基本概念
- 當你想要初始化 class，你可能需要在每個class中定義一個假設的`initialize()`方法。但這要求每個用戶都記得手動調用這個方法，容易出錯
- 通過使用**constructor**，我們可以確保每個對象在創建時自動初始化
- Java在建立對象時會自動調用與class名稱相同的constructor，這樣編譯器便知道如何進行初始化
- constructor的名稱必須與class名完全一致，並且不適用通常方法名首字母小寫的命名風格
### 範例說明
#### Default Constructor 範例
```java
class Rock {
  Rock() { // This is the constructor
    System.out.print("Rock ");
  }
}

public class SimpleConstructor {
  public static void main(String[] args) {
    for(int i = 0; i < 10; i++)
      new Rock();
  }
} /* Output:
Rock Rock Rock Rock Rock Rock Rock Rock Rock Rock
*/
```

- 使用`new Rock();`創建 object 時，Java會為對象分配記憶體並調用其 **constructor**
- 未接收任何參數的 **constructor** 稱為 **default constructor**。在Java文件中，這類 **constructor** 常被稱為 **no-arg constructor**
#### Parameterized Constructor 範例
- 通過對constructor稍加修改，我們可以使其接受參數，從而根據這些參數來進行更為定制化的初始化操作
```java
class Rock2 {
  Rock2(int i) {
    System.out.print("Rock " + i + " ");
  }
}

public class SimpleConstructor2 {
  public static void main(String[] args) {
    for(int i = 0; i < 8; i++)
      new Rock2(i);
  }
} /* Output:
Rock 0 Rock 1 Rock 2 Rock 3 Rock 4 Rock 5 Rock 6 Rock 7
*/
```
- **constructor**不允許有任何返回類型，甚至包括void。這是因為constructor的主要目的是初始化對象，而不是計算值或返回數據。如果在constructor定義中加入返回類型，會導致編譯錯誤

## Method overloading
- 在Java中，相同名稱的方法可以有不同的實現，這種設計主要是為了減少冗餘性。例如，"洗"這個動作可以用於洗衣服、洗車或洗狗。若每種情況都設立一個獨立的方法名稱，將會非常冗餘。
- **方法多載 (method overloading)** 允許相同的方法名稱在同一個類別中以不同的參數列表存在，這樣可以根據傳入參數的不同調用相對應的方法
```java
class Tree {
  int height;
  Tree() {
    print("Planting a seedling");
    height = 0;
  }
  Tree(int initialHeight) {
    height = initialHeight;
    print("Creating new Tree that is " +
      height + " feet tall");
  }	
  void info() {
    print("Tree is " + height + " feet tall");
  }
  void info(String s) {
    print(s + ": Tree is " + height + " feet tall");
  }
}

public class Overloading {
  public static void main(String[] args) {
    for(int i = 0; i < 5; i++) {
      Tree t = new Tree(i);
      t.info();
      t.info("overloaded method");
    }
    // Overloaded constructor:
    new Tree();
  }	
} /* Output:
Creating new Tree that is 0 feet tall
Tree is 0 feet tall
overloaded method: Tree is 0 feet tall
Creating new Tree that is 1 feet tall
Tree is 1 feet tall
overloaded method: Tree is 1 feet tall
Creating new Tree that is 2 feet tall
Tree is 2 feet tall
overloaded method: Tree is 2 feet tall
Creating new Tree that is 3 feet tall
Tree is 3 feet tall
overloaded method: Tree is 3 feet tall
Creating new Tree that is 4 feet tall
Tree is 4 feet tall
overloaded method: Tree is 4 feet tall
Planting a seedling
*/
```

- 此範例中，**Tree** 類別提供了兩種constructor：一種不接受參數，一種接受一個整數參數
- 對明顯相同的方法採用兩個名字肯定會讓人疑惑，有了多載，可以使用相同的名字
	- 例如範例的 **info()** 方法：一種不接受任何參數，另一種接受一個字符串參數，
#### 區分 overloaded methods
- 每個多載的方法必須具有獨一無二的參數列表，包括參數的數量、類型或參數的排列順序
- 參數順序的變更雖然技術上可行以區分多載方法，但可能造成代碼難以維護
#### Overloading with primitives
- 數值常數（如5）預設被處理為 **int** 類型
- 若傳入數據類型小於方法所需的參數類型，則進行類型提升；例如，**char** 類型若無匹配方法會提升至 **int**
- 如果傳入數值的類型比方法參數大，不透過類型轉換進行 narrowing，否則會導致編譯錯誤
#### 根據 return value 的多載
- Java不支持僅憑返回值不同來多載方法。因為在呼叫時，單憑方法名和參數無法確定具體調用哪個方法，特別是在不關心返回值的情況下
	- 有時可能為了副作用調用 method，而不關心 return value，例如只調用 `f();`，縱使方法定義了回傳值，編譯器也無法判別要使用哪個 overloading method

## Default Constructors
### **基本概念**
- 在Java中，如果一個類中沒有明確定義任何constructor，則編譯器會自動為該類創建一個無參數的default constructor。這個自動生成的constructor不會執行任何特殊操作，僅僅是允許創建該類的實例
- 這個自動產生的constructor通常被稱為**無參數構造函數**（no-arg constructor）
### **條件限制**
- 如果類中已經明確定義了至少一個constructor（無論該constructor是否接受參數），則編譯器不會自動生成default constructor
- 這意味著如果你已經定義了一個或多個帶參數的constructors而沒有定義無參數的constructor，嘗試無參數創建實例（即使用**`new ClassName();`**）將導致編譯錯誤