---
title: 数据结构 - 笔记
description: Notes about Data Structure
url: data-structure-notes
date: 2022-07-19 00:00:00+0000
categories:
    - Data Structure
tags:
    - study
    - Data Structure
    - CS
weight: 1
---

# 基本数据结构

## 栈和队列

### 栈

栈：后入先出，LIFO
属性`S.top`为指向栈顶指针
栈下溢：对空栈弹出，栈上溢：`S.top>n`；

基本操作：

```C
Stack-Empty(S){
    if S.top==0{
        return True
    }
    else{
        return False
    }
}

PUSH(S,x){
    S.top+=1
    S[S.top]=x
}

POP(S){
    if Stack-Empty(S){
        error "underflow"
    }
    else{
        S.top-=1
        return S[S.top+1]
    }
}
```

执行时间：O(1)

### 队列

队列：先入先出，FIFO
属性`Q.head`指向队列头，`Q.tail`指向队列尾端，事实上可以用循环链表表示，只需要一个指针
`underflow`：对空队列执行删除，`overflow`对满队列执行插入

入队出队操作，默认`n=Q.length`：

```C
ENQUEUE(Q,x){
    Q[Q.tail]=x
    if Q.tail==Q.length{
        Q.tail=1
    }
    else{
        Q.tail+=1
    }
}

DEQUEUE(Q){
    x=Q[Q.head]
    if Q.head=Q.length{
        Q.head=1
    }
    else{
        Q.head+=1
    }
    return x
}
```

## 链表

双向链表：结构包含`key`, `next`, `prev`，后继前驱
单链表省略`prev`
循环链表，表头元素`prev`指向队尾元素

可以通过设置哨兵辅助循环便利列表，`L.nil`

基本操作：

```C
//不带哨兵
List-Search(L){
    x=L.head
    while x!=NIL and x.key!=k{
        x=x.next
    }
    return x
}

//带哨兵
List-Search'(L){
    x.next=L.nil.next
    while x!=L.nil and x.key!=k{
        x=x.next
    }
    return x
}

List-Insert(L){
    x.next=L.head
    if L.head!=NIL{
        L.head.prev=x
    }
    L.head=x
    x.prev=NIL
}
```

## 有根树

二叉树T，属性`p, left, right`分别指向父结点，左孩子，右孩子，以及属性`key`
无限制树：左孩子右兄弟表示法，包含`T.root`指向树T的根结点，指向父结点指针`p`，还有两个指针：
- `x.left-child`指向结点`x`最左边孩子
- `x.right-sibling`指向`x`右侧相邻兄弟的结点
若无满足结点，则为`NIL`

# 散列表

利用关键字计算其下标，进行直接寻址

## 直接寻址表（字典）

数组`T[0,...,m-1]`，每个位置称为槽(slot)，对应一个关键字元素，若无则为`NIL`

操作：

```C
Dir-Addr-Search(T,k){
    return T[k]
}

Dir-Addr-Insert(T,x){
    T[x.key]=x
}

Dir-Addr-Delete(T,x){
    T[x.key]=NIL
}
```

## 散列表

利用散列函数将关键字映射到槽：
$$h:U\xrightarrow\{0,1,\cdots,m-1\}$$
冲突：多个值映射到相同的槽
通过链接法解决冲突，每个槽包含一个链表，后加入的结点延伸链表
对此方法的散列表，一次成功或不成功的查找平均时间都是$\theta(1+\alpha)$

### 散列函数

#### 除法散列法

$$h(k)=k\pmod{m}$$

#### 乘法散列法

$$h(k)=\lfloor m(kA\pmod{1})\rfloor$$

#### 全域散列法

随机选择函数

# 二叉搜索树

既可以作为字典，又可以作为优先队列
基本操作花费的时间与高度成正比

高度为结点层数-1

每个结点为一个对象，属性有`key, left, right, p`，后三者指向左孩子，右孩子，父结点

> 存储性质：
即左子树所有结点小于根结点，右子树所有结点不小于根结点。
设`x`为二叉搜索树，如果`y`为`x`左子树的一个结点，那么`y.key`$<$`x.key`。若`y`为`x`右子树结点，则`y.key`$\geqslant$`x.key`。

## 遍历

先序遍历：根左右
中序遍历：左根右
后序遍历：左右根

输出方式很简单，中序遍历：

```C
TREE_WALK(x){
    if (x!=NIL){
        TREE_WALK(x,left);
        print(x.key);
        TREE_WALK(x.right);
    }
}
```

**定理：若x为有n个结点子树的根，那么调用遍历函数需要$\theta(n)$的时间**

## 查询

### 查找

```C
TREE_SEARCH(x,k){
    if(x==NIL or k==x.key){
        return x;
    }
    if(k<x.key){
        return TREE_SEARCH(x.left,k);
    }
    else{
        return TREE_SEARCH(x.right,k);
    }
}
```

运行时间为$O(h)$，h为树高度

迭代版本效率更高：

```C
ITERATIVE_TREE_SEARCH(x,k){
    while(x!=NIL and k!=x.key){
        if(k<x.key){
            x=x.left;
        }
        else{
            x=x.right;
        }
    }
    return x;
}
```

### 最大最小

更简单，最小一定在最左子结点，最大一定在最右子结点：

```C
TREE_MINIMUM(){
    while(x.left!=NIL){
        x=x.left;
    }
    return x;
}

TREE_MAXIMUM(){
    while(x.right!=NIL){
        x=x.right;
    }
    return x;
}
```

### 前驱后继

前驱结点`key`值小于该结点但最大
后继结点反之

#### 后继结点

若`x`右子树非空，其后继结点为其右子树中的最左结点
否则去向上查找一个父结点，直到父结点的左子树等于当前结点（在向上过程变化赋值，即父结点为右父结点），返回该父结点。

```C
TREE_SUCCESSOR(x){
    if(x.right!=NIL){
        return TREE_MINIMUM(x.right);
    }
    y=x.p;
    while(y!=NIL and x==y.right){
        x=y;
        y=y.p;
    }
    return y;
}

TREE_SUCCESSOR(x){
    if(x.left!=NIL){
        return TREE_MAXIMAM(x.left);
    }
    y=x.p;
    while(y!=NIL and x==y.left){
        x=y;
        y=y.p;
    }
    return y;
}
```

**定理：以上算法在$O(h)$时间内完成**

## 插入删除

### 插入

以结点`z`为输入，满足`z.key=v, z.left=NIL, z.right=NIL`

```C
TREE_INSERT(T,z){
    y=NIL;
    x=T.root;
    while(x!=NIL){
        y=x;
        if(z.key<x.key){
            x=x.left;
        }
        else{
            x=x.right;
        }
    }
    z.p=y;
    if(y==NIL){
        T.root=z;
    }
    else if(z.key<y.key){
        y.left=z;
    }
    else{
        y.right=z;
    }
}
```

### 删除

从二叉搜索树T中删除结点z，分三种基本情况：

- 若z没有孩子结点，简单将其删除，修改其父结点，用NIL作为父结点的孩子结点替换z
- 若z只有一个孩子，修改父结点对应的孩子结点，将孩子替换z
- 若z有2个孩子，则z的后继结点y一定为其右子树最左结点，即y没有左孩子，则y替换z，原位置换位NIL，改变y和z的父结点
    - 若y为z的右孩子，则用y起替换z，并仅留下y的右孩子。
    - 否则先用y的右孩子替换y，再用y替换z。

#### 移动子树

用另一棵子树替换一棵子树并成为其双亲的孩子结点。

```C
TRANSPLANT(T,u,v){
    if(u.p==NIL){
        T.root=v
    }
    else if(u==u.p.left){
        u.p.left=v;
    }
    else{
        u.p.right=v;
    }
    if(v!=NIL){
        v.p=u.p;
    }
}
```

删除操作：

```C
TREE_DELETE(T,z){
    if(z.left==NIL){
        TRANSPLANT(T,z,z.right);
    }
    else if(z.right==NIL){
        TRANSPLANT(T,z,z.right);
    }
    else{
        y=TREE_MINIMUM(z.right);
        if(y.p!=z){
            TRANSPLANT(T,y,y.right);
            y.right=z.right;
            y.right.p=y;
        }
        TRANSPLANT(T,z,y);
        y.left=z.left;
        y.left.p=y;
    }
}
```

> 定理：在高度为h的二叉搜索树上，实现`INSERT`和`DELETE`操作的运算时间均为`O(h)`。

## 随即构建二叉搜索树

> 定理：一颗有n个不同关键字的随机构建的二叉搜索树的期望高度为`O(log n)`。
指数高度：$X_{n}$表示一颗有n个不同关键字的随机构建二叉搜索树的高度，并定义指数高度$Y_{N}=2^{X_{n}}$

最终有$E[Y_{n}]\leqslant \frac{1}{4}{n+3\choose 3}$，$E[X_{n}]=O(\log{n})$

# 红黑树

红黑树是一棵二叉搜索树，在每个结点增加一个存储位表示结点的**颜色**，为`red`或`black`。通过从根到叶子的简单路径对各个结点的颜色约束，保证没有路径比其他路径长2倍。近似为**平衡的**

结点属性：`color, key, left, right, p`

NIL视为指向二叉搜索树叶结点（外部结点）的指针，带关键字的结点为内部结点。

红黑树为满足下列**红黑**性质的二叉搜索树：
1. 每个结点或是红色，或是黑色的
2. 根结点为黑色的
3. 叶结点（NIL）为黑色的
4. 如果一个结点为红色的，则它的两个子结点为黑色的
5. 对每个结点，从该结点到其所有后代叶结点（NIL）的简单路径上，均包含相同数目的黑色结点

简化而言，使用`T.nil`哨兵代替NIL，`color`为`black`

**黑高**：从某个结点x出发（不含该结点），到达一个叶结点的任意简单路径上的黑色结点个树为该店黑高，记作`bh(x)`

定义红黑树的黑高为根结点的黑高

> 引理：一棵内部有n个结点的红黑树的高度至多为$2\log{(n+1)}$

需要保持二叉树红黑性质的操作

## 旋转

当结点x做左旋，假设y为其右孩子且部位`T.nil`，*x可以为其右孩子不是`T.bil`结点书内任意结点*。

旋转操作使y成为盖子树新的根结点，x为y的左孩子，原先y的左孩子成为x的右孩子。

![rotate](photos/red_black_rotate.png)

```C
LEFT_ROTATE(T,x){
    y=x.right;
    x=right=y.left;
    if(y.left!=T.nil){
        y.left.x;
    }
    y.p=x.p;
    if(x.p==T.nil){
        T.root=y;
    }
    else if(x==x.p.left){
        x.p.left=y;
    }
    else{
        x.p.right=y;
    }
    y.left=x;
    x.p=y;
}
```

## 插入

可以在$O(\log{n})$时间内向一棵n结点红黑树插入一个新结点。
直接调用`RB_INSERT`来插入结点，其中调用`RB_INSERT_FIXUP`来重新染色，保证红黑性质能够持续

```C
RB_INSERT(T, x){
    y=T.nil
    x=T.root
    while(x!=T.nil){
        y=x
        if(z.key<x.key){
            x=x.left;
        }
        else{
            x=x.right;
        }
    }
    z.p=y;
    if(y==T.nil){
        T.root=z;
    }
    else if(z.key<y.key){
        y.left=z
    }
    else{
        y.right=z
    }
    z.left=T.nil;
    z.right=T.nil;
    z.color=RED;
    RB_INSERT_FIXUP(T,z);
}

RB_INSERT_FIXUP(T,z){
    while(z.p.color==RED){
        if(z.p==z.p.p.left){
            y=z.p.p.right;
            if(y.color==RED){
                z.p.color=BLACK;
                y.color=BLACK;
                z.p.p.color=RED;
                z=z.p.p;
                continue;
            }
            else if(z==z.p.right){
                z=z.p;
                LEFT_ROTATE(T,z);
            }
            z.p.color=BLACK;
            z.p.p.color=RED;
            RIGHT_ROTATE(T,z.p.p);
        }
        //reverse
        else{
            y=z.p.p.left;
            if(y.color==RED){
                z.p.color=BLACK;
                y.color=BLACK;
                z.p.p.color=RED;
                z=z.p.p;
                continue;
            }
            else if(z==z.p.left){
                z=z.p;
                RIGHT_ROTATE(T,z);
            }
            z.p.color=BLACK;
            z.p.p.color=RED;
            LEFT_ROTATE(T,z.p.p);
        }
    }
    T.root.color=BLACK;
}
```

## 删除

同样花费$O(\log{n})$时间。

先提供一个子过程-结点替换，使删除结点过程用于红黑树。

```C
//在T中用u替换v
RB_TRANSPLANT(T,u,v){
    if(u.p==T.nil){
        T.root=v;
    }
    else if(u==u.p.left){
        u.p.left=v;
    }
    else{
        u.p.right=v;
    }
    v.p=u.p;
}
```

与普通`TREE_DELETE`区别：
- 需要记录操作结点`y`的踪迹，因为它可能破坏红黑性质。另有`x`，因此结束后需调用另一辅助子过程`RB_DELETE_FIXUP`，利用改变颜色和执行旋转来回复红黑性质;
- 始终维持结点`y`为书中删除的结点或一致树内的结点。当`z`的子结点少于2个，将`y`移除；当`z`的子结点为2个时，y将移动至`z`的位置；
- 由于`y`的颜色可能改变，`y-original-color`存储了改变前的y的颜色。在删除操作结束时测试它，若为黑，则删除或移动`y`会引起红黑性质的破坏。
- 保存`x`结点的踪迹，使它移动至`y`结点的原始位置上。属性`x.p`被设置指向树中`y`的父结点的原始位置（即使`x`为`T.nil`）
- 若`y`结点为黑色，调用恢复函数，否则当`y`为红色或呗删除/移动时，红黑性质保持，后者原因：
    - 树高没有变化
    - 不存在两个相邻的红结点，因为`y`在树中占据了`z`的位置，考虑`z`的颜色，`y`的新位置不可能有两个相邻的红结点
    - 若y为红色，则不可能为根结点，因此根结点仍为黑色。

若'y'为黑色，调用`RB_DELETE_FIXUP`补救

```C
RB_DELETE(T,z){
    y=z;
    y-original-color=y.color;
    if(z.left==T.nil){
        x=z.right;
        RB_TRANSPLANT(T,z,z.right);
    }
    else if(z.right==T.nil){
        x=z.left;
        RB_TRANSPLANT(T,z,z.left);
    }
    else{
        t=TREE_MINIMUM(z.right);
        y-original-color=y.color;
        x=y.right;
        if(y.p==z){
            x.p=y;
        }
        else{
            TB_TRANSPLANT(T,y,y.right);
            y.right=z.right;
            y.right.p=y;
        }
        RB_TRANSPLANT(T,z,y);
        y.left=z.left;
        y.left.p=y;
        y.color=z.color;
    }
    if(y-original-color==BLACK){
        RB_DELETE_FIXUP(T,x);
    }
}

RB_DELETE_FIXUP(T,z){
    if(x==x.p.left){
        w=x.p.right;
        if(w.color=RED){
            /*case 1*/
            w.color=BLACK;
            x.p.color=RED;
            LEFT_ROTATE(T,x,p);
            w=x.p.right;
        }
        if(w.left.color==BLACK && w.right.color==BLACK){
            /*case 2*/
            w.color=RED;
            x=x.p;
        }
        else if(w.right.color==BLACK){
            /*case 3*/
            w.left.color=BLACK;
            w.color=RED;
            RIGHT_ROTATE(T,w);
            w=x.p.right;
        }
        /*case 4*/
        w.color=x.p.color;
        w.p.color=BLACK
        w.right.color=BLACK;
        LEFT_ROTATE(T,x.p);
        x=T.root;
    }
    else{
        w=x.p.left;
        if(w.color=RED){
            w.color=BLACK;
            x.p.color=RED;
            RIGHT_ROTATE(T,x,p);
            w=x.p.left;
        }
        if(w.right.color==BLACK && w.left.color==BLACK){
            w.color=RED;
            x=x.p;
        }
        else if(w.left.color==BLACK){
            w.right.color=BLACK;
            w.color=RED;
            LEFT_ROTATE(T,w);
            w=x.p.left;
        }
        w.color=x.p.color;
        w.p.color=BLACK
        w.left.color=BLACK;
        RIGHT_ROTATE(T,x.p);
        x=T.root;
    }
    x.color=BLACK
}
```

对`FIXUP`，`while`循环的目标是将额外的黑色沿树上移，直到下列情况：
1. `x`指向红黑结点，将`x`着黑色（最后一行）
2. `x`指向根结点，此时可移除
3. 执行旋转和重新着色，退出循环

x总指向一个具有双重黑色的非根结点
> 双重黑色结点，一个结点贡献两个黑色，消除方法：黑色结点=双重黑色结点+红色结点

