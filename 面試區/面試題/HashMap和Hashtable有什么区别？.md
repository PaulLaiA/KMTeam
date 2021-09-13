---
界: Java
门: Container
纲: 
tags: ["#Java","#Container","#BQ"]
aliases:
  - 
date: 2021-09-13
---

## 共同性

HashMap和Hashtable都实现了Map接口

## 不同性

| 不同點      | HashMap                 | HashTable                          |
| ----------- | ----------------------- | ---------------------------------- |
| Key & Value | 可以為Null              | 不可以為null                       |
| 線程安全    | 非同步,適合單線程       | 同步,適合多線程                    |
| 遍歷順序    | 有序                    | 無序                               |
| Fail-fast   | 提供Key set迭代所以支持 | 提供key Enumeration迭代,所以不支持 |


### 參考
[[HashMap為什麼有序？]]

HashTable被認為是一個遺留的類別

如果需要尋求在迭代時還能夠修改的Map

可以推薦使用 `ConcurrentMap`