---
title: 'C#基础系列笔记-2'
categories:
  - 学习笔记
tags:
  - 'C#'
  - 编程
  - 学习
abbrlink: f07f715e
date: 2018-10-04 11:21:36
---

这期我们自己实现List的基本功能。

<!---more--->

## 自己实现List - 基本功能

使用数组来自己实现List功能。List有以下几个主要功能：
1. 初始化
2. Add - 添加元素
3. RemoveAt - 删除对应位置元素
4. Get - 获得对应位置元素
5. Count - 元素总数

以Add为例，在添加一个元素之前要临时构建一个新数组，将原数组的数据存入，并将添加的元素加在新数组的最后。

    public void Add(int n)
        {
            if (len >= mem.Length)
            {
                int[] mem_new = new int[mem.Length * 2];
                mem.CopyTo(mem_new, 0);
                mem = mem_new;
            }

            mem[len] = n;
            len++;
        }

而Remove等方法需要注意下标越界等问题，在进入方法后先做相关判断。

    if(len<0 || len <= index)
            {
                Console.WriteLine("RemoveAt 下标越界{0} {1}", len, index);
                return;
            }

Count与Get即为简单的获取长度和对应元素，在这里不详细展开。