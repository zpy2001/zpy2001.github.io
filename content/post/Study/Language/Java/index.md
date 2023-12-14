---
title: Java - 笔记
description: "Notes about Java"
slug: language-java
date: 2021-08-09 00:00:00+0000
categories:
    - Java
tags:
    - study
    - Java
    - OOP
    - CS
    - Programming languages
weight: 1
---

# JAVA

## 面向对象

### 对象与类

1. 万物皆为对象

2. 程序是对象的集合，通过发送消息告知彼此所需信息

3. 可通过创建包含现有对象的包，来创建新类型对象

4. 每个对象都有其类型

5. 某特定类型的所有对象都可以接受相同消息

对象具有状态、行为、标识，即内部数据、方法、唯一地址

接口：对某特定对象能够发出的请求

类创建者与客户端程序员

三个关键字在类的内部设定边界：

- `public`:对任何人可用

- `private`:除了类创建者和类内部方法外都不能访问

- `protected`:类似`private`，但继承的类可以访问`protected`成员

代码复用
创建新类使用其他类中对象

### 继承

创建和父类相似的子类时，可使用继承，新类包含现有类型的所有成员，且复制了基类的接口，因此发送给基类消息同时发送给子类对象，父类与子类具有相同的类型

在父子类中产生差异：

1. 在子类添加新方法

2. 覆盖，改变子类中现有基类方法，在子类重新定义方法

java 单根继承，所有类最终继承自Object

### 对象创建与生命周期

- C++: 追求效率，对象存储空间与生命周期在程序编译时确定，对象存储位置位于堆栈或静态存储区，牺牲了灵活性

- JAVA: 在称为堆的内存池中动态创建对象，程序在运行时明确对象情况，因为动态管理，时间消耗大

C++中必须通过编程直接销毁对象，容易因遗忘而导致内存泄漏

java使用垃圾回收器，根据判断对象何时不再使用进行自动释放内存操作

### Java and Internet

#### Web

信息存储池（数据库）、分发信息的软件、所在机器集群 称为服务器

用户机器 客户机

中间件，将负载分散给服务器端机器

CGI：通用网关接口，表单数据传递给Web服务器

客户端编程：

1. 浏览器插件

2. 脚本语言，客户端程序源码潜入Html页面

3. java，通过applet（只在web浏览器运行的小程序）

.NET ~ JVM虚拟机
C# ~ java

### 编程约定

- 类名的首字母大写
- 类名由几个单词构成直接拼在一起（无_），每个内部单词首字母大写

驼峰风格

标识符的第一个字母小写

## 对象

### 创建

安全做法：创建引用同时进行初始化：`String s = "java";`，利用java语言特性：字符串可用带引号的文本初始化

引用与新对象关联，使用`new`操作符，因此上例可写作：`String s = new String("java");`

#### 存储地

1. 寄存器
2. 堆栈，位于RAM中，通过堆栈指针从处理器获得直接支持，java对象引用可存储

3. 堆，通用内存池，位于RAM，存放所有java对象

4. 常量存储，常量值直接存储在程序代码内部，嵌入式中可选地放在ROM

5. 非RAM存储流对象，对象转化为字节流；持久化对象位于磁盘上，程序终止仍然保存状态

#### 基本类型

![types](types.png)

包装器类提供方法使用原始数据类型作为对象

有时必须使用包装器类，例如在使用ArrayList对象集合时，其中不能使用原始类型（列表只能存储对象）

```java
ArrayList<int> myNum = new ArrayList<int>(); // invalid
ArrayList<Integer> myNum = new ArrayList<Integer>(); // valid
```

自动包装可自动转换基本类型为包装器类型

高精度数字

高精度计算的类：`BigInteger`和`BigDecimal`，虽然属于包装器类，但没有对应的基本类型，分别支持任意精度整数和定点数

速度换精度

### 无需销毁

#### 作用域

作用域由`{}`位置决定，定义的变量可用于作用域结束前

java自由格式，空白符无影响

**java中大作用域变量不可隐藏**，在作用域内（包括其内部小作用域）变量只能声明一次

java用new生命的对象只要需要会一直保存，而他的引用在作用域结束即消失，垃圾回收器监视new的对象，释放不被使用的对象内存空间

### 创建类

创建新类型名称：`class ATypeName{/* class body */}`

创建这种类型的对象：`ATypeName a = new ATypeName();`

#### 字段和方法

可在类中设置：字段（数据成员）和方法（成员函数）

字段为任何类型的对象，可通过其引用与其通信，也可以为基本类型的一种

字段若为某个对象的引用，则必须初始化该引用（使用new实现）

类似`struct`，饮用对象成员：`objectReference.member`

基本类型若未初始化，默认值为对应数据类型的0

未初始化局部变量在java中被视为错误

### 方法、参数、返回值

```java
ReturnType methodName (/* argument list */){
    /* Method body */
}
```

调用方法：`ReturnType res = objectName.methodName(/* arglist */)`

发送消息给对象

### 名称管理

反转域名

### 其他构件

`import`
具体的包，可使用通配符*

### static

创建类时只是在描述，执行new创建对象时数据空间被分配

在创建类时，可能会出现需求：

1. 只为某特定域分配单一存储空间，不考虑创建多少对象

2. 希望某个方法与类任何对象有关，即使没有创建对象也能够调用方法

使用`static`关键字，使声明的事物不会与累的任何对象实例联系在一起，可以直接调用类的static方法访问static域（相当于起到全局变量作用）

类方法、类数据，上述对象只针对整个类，而非类中的特定对象

引用`static`变量可通过对象去定位，也可以直接使用类名称

## 操作符

几乎所有的操作符都只能操作基本类型，例外的操作符为`=`,`==`,`!=`，这些操作符能操作所有对象，String类支持`+`和`+=`

对象进行赋值操作时，真正操作的是对象的引用，于是两个引用指向同一对象

判断对象内容是否相同使用`equals()`方法

java没有`sizeof()`，因为数据类型大小规定死

大整数溢出编译器无警告/报错

## 控制

Foreach

索引数组中每一个元素`for(TypeName x : ArrayList){}`

java没有`goto`，但在循环内`continue label`和`break label`就会中断循环（多个），知道标签所在的地方，相当于使用`goto`

1. 一般的`continue`会退回最内层循环的开头，继续执行

2. 带标签的`continue`，到达标签的位置，重新进入标签后的循环（语句）

3. 一般的`break`，跳出当前循环

4. 带标签的`break`，中断并跳出标签所指的循环

## 初始化和清理

### 初始化

创建对象时，若类具有构造器，则在用户操作对象前，java自动调用构造器进行初始化（new时）

构造器即以类相同名称定义的函数

无参构造器：默认构造器

也可以有形式参数

但构造器无返回值

有形式参数的构造器在调用时`new ClassName(/* args */)`

### 方法重载

重复定义相同名称不同的方法

区分规则：每个重载方法都必须具有独一无二的参数类型列表

### 默认构造器

无构造器直接new会自动创建

含参数构造器参数类型必须匹配，否则找不到正确构造器

### this

在方法内部获取对当前对象的引用，使用`this`

如果在方法内部调用同类中的其他方法，直接调用即可

在方法内部调用类中数据成员，`this.ClassElement`

this加参数列表`this(/* args */)`，代表对符合参数形式的构造器的引用

于是即可调用其他构造器

必须将构造器调用置于最起始处，且在一个构造器中只能使用`this()`调用一个构造器

`static`方法即没有`this`的方法

### 清理

垃圾回收器只知道释放经由`new`分配的内存，若要释放特殊获得的内存区域，java允许在类中定义`finalize()`方法

1. 对象可能不被垃圾回收

2. 垃圾回收不等于析构

3. 垃圾回收只与内存有关

`System.gc()`强制垃圾回收、终结

垃圾回收机制：
1. 引用计数，每个对象含有一个引用计数器，当引用连接至对象时，引用计数+1，当引用离开作用域或置为`null`时，引用计数-1，垃圾回收器遍历对象，当引用计数为0，释放占用的空间。缺陷：循环引用时，该回收可能引用计数不为0

2. 自适应垃圾回收；从堆栈和静态存储区开始遍历引用，可找到活的对象
    - 停止-复制：暂停程序运行，复制活得对象到新的堆，紧凑排列；
    - 若没有新垃圾产生，则会转换至标记-清扫：对存活对象标记，全部标记工作完成后，清理动作开始，释放没标记的对象，重新整理后可得连续空间

### 初始化

静态子句：
```java
public class Spoon {
    static int i;
    static {
        i = 47;
    }
}
```

实例初始化子句：
```java
class Mug {
    // sth..
}
public class Mugs {
    Mug mug1;
    Mug mug2;
    {
        mug1 = new Mug(1);
        mug2 = new Mug(2);
    }
}
```

#### 数组初始化

定义数组：
`int[] a`或`int a[]`

数组互相直接赋值时只复制引用

可以用`new int[/* nums of elements in array */]`创建未知元素个数数组

#### 可变参数列表

由于所有类继承于`Object`类，可创建以`Object`数组为参数的方法

```java
public class VarArgs {
    static void printArray(Object[] args){
        for (Object obj : args){
            System.out.print(obj+" ");
        }
        System.out.println();
    }
    public static void main(String[] args){
        printArray(new Object[]{ /* args */ });
    }
}
```

或者

```java
public class VarArgs {
    static void printArray(Object... args){
        for (Object obj : args){
            System.out.print(obj+" ");
        }
        System.out.println();
    }
    public static void main(String[] args){
        printArray(new Object[]{ /* args */ });
    }
}
```

可变参数可以是：
```java
Character... args
int... args
String... args
Integer... args
```

### 枚举类型

```java
public enum Spiciness {
    NOT, MILD, MEDIUM, HOT, FLAMING
}
```

调用时使用`Spiciness.MEDIUM`

`toString()`方法，显示枚举实例名称，`ordinal()`方法表示某特定`enum`常量的声明顺序，`values()`方法，产生由常量值构成的数组

`enum`是类，声明对象可以在`switch`语句中应用

## 访问权限

最大权限到最小权限：`public`, `protected`, 包访问权限, `private`

导入`java.util.*`提供一个管理名字空间的机制，所有类成员名称彼此隔离

编写的`java`源码文件称为编译单元，后缀名`.java`，内部最多有一个`public`类，类名称与文件名相同

java可运行程序是一组可以打包并压缩为一个java文档文件(`.jar`)的`.class`文件，java解释器负责文件的查找、装载和解释

一组类文件从属于同一个群组，使用`package libName`，声明时必须是文件中除注释外第一句程序代码，声明该编译单元为`libName`类库中的

使用包可以`import`，java包的命名规则使用小写字母

`import`类库时若有冲突的类则在使用时需要明确指明

`java.util.Vector v = new java.util.Vector`

java没有条件编译，因为跨平台

取得对类中成员访问权途径：

1. 使其成为`public`

2. 通过不加访问权限修饰词，将其他类放置于同一个包内，包内其他类也可以访问该成员

3. 类继承，能够访问`public, protected`成员

4. 提供访问器和变异器方法(get/set)，读取改变数值

在同级目录下未设定包名称的文件默认隶属于目录的默认包，因此互相具有包访问权限

继承

通过继承可利用现有基类，新成员添加或改变现有成员行为，不需动基类
声明：
```java
class Foo extends Bar{}
// Foo new class, Bar basic class
```

包内继承类通过`protected`获取包访问权限

封装

数据与方法打包 具体实现的隐藏

print(Object x)

可以打印类

直接以`String`方式使用对象时，自动索引`toString()`函数，若没有则打印`类名+符号+@+对象的哈希码值`

只有命令行调用的类`main()`方法会调用

即使一个类只有包访问权限，`public main()`仍可访问

## 复用类

### 组合

新的类由当前类的对象组成

每一个非基本类型的对象都有`toString()`方法

初始化引用：

1. 在定义对象之处

2. 在类的构造器中

3. 在使用对象之前，惰性初始化

4. 使用实例初始化

### 继承

`新类 extends 旧类`

定义相同名称对象/方法则覆盖，新对象/方法

super 为指向父类的指针

基类在导出类访问之前就完成初始化，导出类默认构造器为基类构造器

### 代理

### 清理

`try,finally`

`try`后的块为保护区，无论`try`块如何退出，`finally`子句代码一定被执行

### 名称屏蔽

基类中重载方法在导出类中用相同名称可直接覆盖，与C++不同，后者需要屏蔽基类方法

`@override`保证导出类名称与父类一致，即为重写而非新增，否则报错

组合在新类中使用现有类的功能而非它的接口

新类用户看到的是新类定义的接口，而非嵌入对象的接口，因此需嵌入一个现有类的`private`对象

### 向上转型

继承类可以自动转换至基类

### final关键字

final对基本类型，数值恒定不变

final对引用，初始化后指向对象恒定不变

既是`static`又是`final`的变量名称全部大写

`private final int xxx`

空白final：声明时未初始化

必须在使用前初始化

可以在参数里使用，无法更改引用指向的对象

final方法

1. 把方法锁定，以防继承类修改定义

2. 效率（早期）

private方法隐式指定为final，无法覆盖

final类，不可继承，即其中方法隐式为final

## 多态

```java
public enum Note{
    MIDDLE_C
}
class Stringed extends Instrument {
    public void play(Note n){
        // ...
    }
} class Music2{
    public static void tune(Instrument i){
        i.play(Note.MIDDL_C);
    }
    public static void main(String[] args){
        Stringed flute = new Stringed();
        tune(flute);
    }
}
```

编译器不知道Instrument引用指向Wind/Brass/Stringed

绑定

将一个方法调用同一个方法主体关联起来称作绑定

前期绑定：程序执行前期

后期/动态/运行时绑定：运行时判断对象类型

java除了static和final方法外其他所有方法后期绑定

动态绑定实现多态

`Shape s = new Circle()`，向上转型

