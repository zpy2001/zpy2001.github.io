---
title: Rust - 笔记
description: "Notes about Rust"
slug: language-rust
date: 2023-07-01 00:00:00+0000
categories:
    - Rust
tags:
    - Rust
    - CS
    - Programming languages
weight: 1
---

# Rust

## 变量与可变性

变量默认是不可改变的，可变变量声明时加`mut`

Rust对常量命名的约定是在单词之间使用全大写加下划线

变量隐藏可在同一作用域，可不同类型

## 数据类型

标量可为符合类型，形式为tuple，长度固定

Rust数组长度固定

## 函数d

使用`fn`关键字声明，函数与变量名为snake case

```rust
fn [func_name](arg: type) -> ret_type
```

返回值不使用return声明，为最后表达式的值，可使用元组返回多一个值

函数声明顺序可任意

## 控制流

`if`无`()`，`if`表达式可为右值：

```rust
let number = if condition { 5 } else { 6 };
```

`loop`始终循环，`break`返回值

循环标签：

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```

`while`循环无括号

- `for element in a`
- `for num in (1..4).rev()`

## 所有权

规则：

- Rust中每个值都有一个所有者
- 值在任一时刻有且只有一个所有者
- 当所有者离开作用域，值被丢弃

`drop`在`}`结尾自动调用

资源获取即初始化(RAII)

### 变量与数据交互方式

#### 变量移动

堆上数据赋值后原值不再有效，栈上数据仍然有效

如果一个类型实现了 `Copy` trait，那么一个旧的变量在将其赋值给其他变量后仍然可用

Rust 不允许自身或其任何部分实现了 `Drop` trait 的类型使用 `Copy` trait

任何一组简单标量值的组合都可以实现 `Copy`，任何不需要分配内存或某种形式资源的类型都可以实现 `Copy`

#### 克隆

`clone`函数，复制堆上数据

函数传参意味着堆上数据离开作用域，利用返回值转移所有权

### 引用与借用

使用`&`表示与传递引用，引用无法被修改，使用可变引用

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

如果你有一个对该变量的可变引用，你就不能在作用域内再创建对该变量的可变引用，两个引用同时有效时避免数据竞争

也不能在拥有不可变引用的同时拥有可变引用

一个引用的作用域从声明的地方开始一直持续到最后一次使用为止

引用不能作为返回值，否则会由于自动销毁造成悬垂指针引用

-   在任意给定时间，**要么** 只能有一个可变引用，**要么** 只能有多个不可变引用。
-   引用必须总是有效的。

### Slice

`&s[0..5]`
`&s[..]`

slice可以作为返回值，类型是`&str`