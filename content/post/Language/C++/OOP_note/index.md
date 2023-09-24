---
title: C++ OOP - 笔记
description: "Notes about C++ OOP"
slug: C-plus-plus-OOP
date: 2022-04-21 00:00:00+0000
categories:
    - C++
tags:
    - C++
    - OOP
    - CS
    - Programming languages
weight: 1
---

# C++面向对象

## basic

```C++
class xx
{
public:
    xxx

private: // private可省略
    xxx
}
```

核心思想`数据隐藏`

外接可通过友元函数访问私有成员，但一般只能通过`public`函数调用，`public`函数可直接使用私有成员

类成员可以为数据或函数

类设计尽可能将公共接口和实现细节分开

不必在类声明中使用关键字`private`，因为这是类对象的默认访问控制

实现类成员函数：
- 定义成员函数时，使用作用域解析运算符`(::)`来标识函数所属的类；
- 类方法可以访问类的`private`组件；

成员函数函数头使用作用域运算符解析`(::)`来指出函数所属的类

类声明常将短小的成员函数作为内联函数，声明使用`inline`关键字，作用是直接替换到函数位置

调用成员函数时，它将使用被用来调用它的对象的数据成员

最好在创建对象时对它初始化，类构造函数

