---
界: Java
门: Basic
纲: 
tags: ["#Java/#Basic","#BQ"]
aliases:
  - 
date: 2021-09-06
---
#String 

1. 使用 StringBuilder 或者 stringBuffer 的 reverse() 方法

```java
	StringBuffer stringBuffer = new StringBuffer();
    stringBuffer.append("abcdefg");
    System.out.println(stringBuffer.reverse()); // gfedcba
    StringBuilder stringBuilder = new StringBuilder();
    stringBuilder.append("abcdefg");
    System.out.println(stringBuilder.reverse()); // gfedcba 
```