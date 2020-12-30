---
title: '[设计模式]UML类图和类之间的关系'
index_img: 
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/W1yZ9Vr7XdaoYKP.png.png
tags:
  - 编程
  - 学习
  - 'C#'
  - 设计模式
categories:
  - 学习笔记
abbrlink: bc3562c
date: 2020-03-19 20:12:41
---

# 例图

![类图](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/uml_class_struct.jpg)

# 类之间的关系

## 泛化关系（generalization）

> 例图中的“小汽车”和“SUV”的关系

泛化关系实际上就是类的继承结构，两个对象之间如果可以用“A是B”，就可以说A和B是继承关系。

例如：自行车是车、猫是动物。

泛化关系用带空心箭头的实线连接，A指向B即为A继承自B。

在代码中，泛化关系表现为继承非抽象类：

``` cs
public class Car
{
  // 汽车类
  // 汽车是“车”这一概念的实现
}

public class SUV : Car
{
  // SUV是汽车
}
```

## 实现关系（realization）

> 例图中“车”与“小汽车”或与“自行车”的关系

实现关系用带空心箭头虚线连接，A指向B即为A是B的实现。

在代码中，实现关系表现为继承抽象类：

``` cs
public abstract class Vehicle
{
  // 车的抽象类
}

public class Car : Vehicle
{
  // 汽车类实现了车
}

public class Bike : Vehicle
{
  // 自行车类实现了车
}

```

## 聚合关系（aggregation）

> 类图中“学生”和“班级”之间的关系

聚合关系用一条带空心菱形箭头的直线表示，A指向B称为A聚合到B上，或者说B由A组成。

聚合关系是实体对象之间的关系，是一个整体由多个部分构成。

整体和部分不是强依赖的，也就是说即使整体不存在了，部分依然存在：班级解散了，学生不会消失，他们还会存在。

## 组合关系（composition）

> 类图中“小汽车”和“轮胎”或“发动机”之间的关系

组合关系用一条带实心菱形箭头直线表示，A指向B称为A组成B，或者B由A组成。

类比聚合关系。

组合关系是强依赖的特殊聚合关系，整体不存在了，部分也不存在了。例如公司不存在了，公司的部门也不存在了。

## 关联关系（association）

> 类图中“学生”和“身份证”之间的关系

关联关系用带箭头的实线表示，一般不强调方向。强调方向的例如A指向B，那就表示A知道B，但B不知道A。

这是一种静态关系，一般与运行状态无关，一般由常识等因素决定。一般来定义对象之间静态的、天然的结构。是强关联的关系。

例如乘车人和车票，学生和学校。

代码实现中，关联对象通常以成员变量的形式实现的：

``` cs
public class Student
{
  public IDCard idCard; // 学生有身份证
}
```

## 依赖关系（dependency）

带箭头的虚线表示，A指向B代表A依赖于B。即A运行期间会用到B。

这个依赖关系是一种临时性的关系，在运行期间产生，在运行时依赖关系也可能会改变。

依赖是有方向的，且双向依赖**非常糟糕**，我们应该保持单项依赖，杜绝双向依赖。

代码实现中，依赖关系体现为类构造方法以及类方法的传入参数，箭头的指向为调用关系。依赖关系的含义除了“临时知道”之外，还有“使用”对方的方法或属性。

``` cs
public class Student
{
  public void Read(Book book)
  {
    printf(book.Title);
  }
}
public class Book
{
  public string Title{get; set;}
}
```
