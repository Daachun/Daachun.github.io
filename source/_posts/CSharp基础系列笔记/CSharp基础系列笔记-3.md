---
title: 'C#基础系列笔记-3'
categories:
  - 学习笔记
tags:
  - 'C#'
  - 编程
  - 学习
abbrlink: 877841c8
date: 2018-10-04 11:42:21
---

让自己的List支持泛型<T>。

<!---more--->

## 自己实现List - 泛型扩展

上一节使用设计的MyList类使用的数据类型全部为int型，要想兼容所有类型，需要用到泛型这个概念。

> 泛型是程序设计语言的一种特性。允许程序员在强类型程序设计语言中编写代码时定义一些可变部分，那些部分在使用前必须作出指明。

在这部分改写中，需要在类名后添加尖括号，声明泛型：

    class MyList<T>
    { /...

我们还要在之前用到int类型参数的地方修改，例如Add()方法：

    public void Add(T n)
        {   /...


在使用中，声明MyList类型需要附加对应的泛型参数：

    MyList<int> list = new MyList<int>();

注意：这里只是简单使用泛型进行改写，实际上泛型更加复杂。例如我们可以对T进行约束，使其只支持对应类型，其余类型不能够使用。

扩展：下标重写

要进行例如List[index]的引用使用，需要在MyList中进行下标重写。

    public T this[int index]
        {
            get
            {
                if (index < 0 || index > -len)
                {
                    return default(T);
                }
                return mem[index];
            }
            set
            {
                if (index < 0 || index >= len)
                {
                    return;
                }
                mem[index] = value;
            }
        }

这里使用get与set方法的原因，是将这种List[index]设定为一个属性。
