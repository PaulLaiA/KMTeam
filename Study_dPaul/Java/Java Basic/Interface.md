---
分类: Interface
tags:
  - Java
  - Interface
  - Note
date: 2021-09-07T00:00:00.000Z
date updated: 2021-10-15 20:54

---

# Java 介面

介面（英文：Interface），在JAVA程式語言中是一個抽像型別，是抽像方法的集合，介面通常以interface來聲明。一個類通過繼承介面的方式，從而來繼承介面的抽像方法。

介面並不是類，編寫介面的方式和類很相似，但是它們屬於不同的概念。類描述對象的屬性和方法。介面則包含類要實現的方法。

除非實現介面的類是抽像類，否則該類要定義介面中的所有方法。

介面無法被實例化，但是可以被實現。一個實現介面的類，必須實現介面內所描述的所有方法，否則就必須聲明為抽像類。另外，在 Java 中，介面型別可用來聲明一個變數，他們可以成為一個空指針，或是被繫結在一個以此介面實現的對象。

### 介面與類相似點：

- 一個介面可以有多個方法。
- 介面檔案儲存在 .java 結尾的檔案中，檔名使用介面名。
- 介面的位元組碼檔案儲存在 .class 結尾的檔案中。
- 介面相應的位元組碼檔案必須在與包名稱相匹配的目錄結構中。

### 介面與類的區別：

- 介面不能用於實例化對象。
- 介面沒有構造方法。
- 介面中所有的方法必須是抽像方法，Java 8 之後 介面中可以使用 default 關鍵字修飾的非抽像方法。
- 介面不能包含成員變數，除了 static 和 final 變數。
- 介面不是被類繼承了，而是要被類實現。
- 介面支援多繼承。

### 介面特性

- 介面中每一個方法也是隱式抽像的,介面中的方法會被隱式的指定為 **public abstract**（只能是 public abstract，其他修飾符都會報錯）。
- 介面中可以含有變數，但是介面中的變數會被隱式的指定為 **public static final** 變數（並且只能是 public，用 private 修飾會報編譯錯誤）。
- 介面中的方法是不能在介面中實現的，只能由實現介面的類來實現介面中的方法。

### 抽像類和介面的區別

1. 抽像類中的方法可以有方法體，就是能實現方法的具體功能，但是介面中的方法不行。
2. 抽像類中的成員變數可以是各種型別的，而介面中的成員變數只能是 **public static final** 型別的。
3. 介面中不能含有靜態程式碼塊以及靜態方法(用 static 修飾的方法)，而抽像類是可以有靜態程式碼塊和靜態方法。
4. 一個類只能繼承一個抽像類，而一個類卻可以實現多個介面。

> **注**：JDK 1.8 以後，介面里可以有靜態方法和方法體了。
>
> **注**：JDK 1.8 以後，介面允許包含具體實現的方法，該方法稱為"預設方法"，預設方法使用 default 關鍵字修飾。更多內容可參考 [Java 8 預設方法](https://www.runoob.com/java/java8-default-methods.html)。
>
> **注**：JDK 1.9 以後，允許將方法定義為 private，使得某些複用的程式碼不會把方法暴露出去。更多內容可參考 [Java 9 私有介面方法](https://www.runoob.com/java/java9-private-interface-methods.html)。