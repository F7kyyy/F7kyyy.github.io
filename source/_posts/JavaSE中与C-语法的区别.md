---
title: JavaSE中与C++语法的区别
date: 2023-05-16 00:21:50
tags:
    - Java
    - 编程语言
    - 工作
categories: Java
math: true
index_img: /img/article/JavaSE.jpg
---

## 1. 数据类型

```java
boolean a=true
```

```c++
bool a=true
```

## 2. 控制语句

```java
outer:
for (int i = 1; i < 4; ++i) { // 在循环语句前，添加 标签: 来进行标记
    inner:
    for (int j = 1; j < 4; ++j) {
        if (i == j)
            break outer; // break后紧跟要结束的循环标记，当i == j时终止外层循环
        System.out.println(i + ", " + j);

    }
}
```

## 3. 声明对象

```java
Person p1=new Person();
Person p2=p1;
// 允许，运算符'=='可以比较
System.out.println(p1==p2);
```

```c++
Person p1=Person();
Person p2=p1;
// 不允许，运算符'=='可以比较
cout << (p1 == p2) << "\n";

Person *p3 = new Person();
Person *p4 = p3;
// 允许，比较两个指针的地址
cout << (p3 == p4) << "\n";
```

