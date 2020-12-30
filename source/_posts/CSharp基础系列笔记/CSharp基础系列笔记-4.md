---
title: 'C#基础系列笔记-4'
categories:
  - 学习笔记
tags:
  - 'C#'
  - 编程
  - 学习
abbrlink: 191cd46b
date: 2018-10-04 12:35:33
---

编写一个控制台回合制游戏。练习面向对象的程序设计思维。

<!---more--->

## 回合制对战

### 简介

这个主题主要练习面向对象的程序设计思维。

从设计的角度来说，本对象应该仅管理本对象相关的信息。例如写一个玩家类，应设计为“被攻击”的方法而不是“攻击他人”的方法。

回合制对战基本设计思路：
1. 实现回合制对战基本框架
2. 加入技能系统，考虑清楚数据结构
3. 技能的配套方法，比如一开始添加技能
4. 玩家可以选择技能，怪物随机释放
5. 利用继承实现框架设计
6. 初级的特殊技能实现（愤怒状态的实现）
7. 状态系统实现特殊技能

这里有个思路上的tip：实现功能要从最基本的地方开始设计，而不是一下子把所有的东西都想好。从最基本的地方来设计是有利于轻松实现的，而一下子想太复杂就会让人千头万绪，不知所措。

同理，等一步一步逻辑搭建起来后，再回头去将需要保护的变量private起来。

>敏捷开发:当策划给出策划案，先不管那么多复杂的东西，快速建立一个原型，让成员先看一下实现效果。基于这个实现效果再去决定下一步是重做还是继续完善。使用快速迭代、循序渐进的方法进行软件开发，将一个大项目分为多个独立运行的小项目，并分别完成。具体内容以后再做详细分析。

>百度百科：[敏捷开发](https://baike.baidu.com/item/%E6%95%8F%E6%8D%B7%E5%BC%80%E5%8F%91/5618867?fr=aladdin)

### 实现回合制对战基本框架

角色类Role定义角色的血量、攻击力以及相关方法。

``` c#
    class Role
    {
        public int hp;
        public int attack

        public Role()
        {
            hp = 100;
            attack = 5;
        }

        public Role(int _hp, int _attack)
        {
            hp = _hp;
            attack = _attack;
        }

        public void BeHit(int cost_hp)
        {
            hp -= cost_hp;
            if (hp < 0) hp = 0;
        }
    }
```
这是一个简单的角色类，使用它只需要实例化即可。

    Role player = new Role();
    Role monster = new Role(200,10);

而游戏主循环的设计更加简单，在Main函数中使用一个while循环即可。

    while (true)
    {
        monster.BeHit(player.attack);
        Console.ForegroundColor = ConsoleColor.DarkGreen;
        Console.WriteLine("【怪物】被攻击，目前生命：{0}",  monster.hp);
        if (monster.hp <= 0) break;
        player.BeHit(monster.attack);
        Console.ForegroundColor = ConsoleColor.Gray;
        Console.WriteLine("【玩家】被攻击，失去{0}点生命，目前生命：{0}", player.hp);
        if (player.hp <= 0) break;
    }
    if (player.hp > 0)
    {
        Console.WriteLine("玩家胜利");
    }
    else
    {
        Console.WriteLine("怪物胜利");
    }

一个简单的回合制游戏循环完成了。输出方面使用的是命令行输出，通过更改**Console.ForegroundColor**来更改输出文字颜色。在最后判断玩家和怪物的血量从而输出哪边胜利。