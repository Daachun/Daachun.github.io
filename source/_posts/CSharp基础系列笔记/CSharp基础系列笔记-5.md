---
title: 'C#基础系列笔记-5'
categories:
  - 学习笔记
tags:
  - 'C#'
  - 编程
  - 学习
abbrlink: 6e1be4fd
date: 2018-10-04 16:53:06
---

让我们在回合制对战中加入技能系统。

<!---more--->


## 回合制对战

### 加入技能系统

加入技能系统之前需要考虑清楚对应数据结构。一个角色可能有多个技能，所以使用List来存储技能**Skill**类。

    class Skill
    {
        int skillType = 1;
        // 属性
        public int damage { get; private set; }
        public Skill(int skillType,int damage)
        {
            this.skillType = skillType;
            this.damage = damage;
        }

        public int GetDamage()
        {
            return damage;
        }
    }

那么先设定一个角色可以随机选择技能释放，在角色中添加随机种子并应用到获取随机技能。

    public List<Skill> skills = new List<Skill>();
    Random random = new Random();
    public Skill RandomSkill()
    {
        return skills[random.Next(0, skills.Count)];
    }

那么即可将普通攻击归为一个技能，并入技能类中。将角色的attack字段和相关参数删除。同时，更改主循环内容。

    monster.BeHit(player.RandomSkill().damage);
    Console.ForegroundColor = ConsoleColor.DarkGreen;
    Console.WriteLine("【怪物】被攻击，目前生命：{0}",  monster.hp);
    if (monster.hp <= 0) break;


### 玩家选择技能

设计**ChooseSkill()**方法来实现玩家选择技能。其中，玩家操作用Console.ReadLine()，通过判断success条件来判断返回的Skill对象。

    public Skill ChooseSkill()
    {
        bool success = false;
        int n = -1;
        while (!success)
        {
            Console.WriteLine("请选择技能 {0}~{1}", 1,skills.Count);
            string input = Console.ReadLine();
            success = int.TryParse(input, out n);
            if (n <= 0 || n > skills.Count)
            {
                success = false;
            }
        }
        return skills[n - 1];
    }

改写主循环

    var skill = player.ChooseSkill();
    monster.BeHit(skill.damage);

### 利用继承实现框架设计

在这个例子中，怪物和玩家本质上是一致的Role对象，那么为了在主循环内统一使用格式，需要我们设计类之间的继承关系。

设定Player类和Monster类，继承Role类。注意：Player类和Monster类的构造函数同样要和Role类相同。

    class Player : Role
    {
        public Player(int _hp) : base(_hp) { }
    }

    class Monster : Role
    {
        public Monster(int _hp) : base(_hp) { }
    }

Role类为基类，那么在基类中定义一个virtual方法

    public virtual Skill SelectSkill()
    {
        return null;
    }

在Player类和Monster类中override实现即可。例如，对于Monster类：

    class Monster : Role
    {
        public Monster(int _hp) : base(_hp) { }
        public override Skill SelectSkill()
        {
            return skills[random.Next(0, skills.Count)];
        }
    }

> CSDN：[函数重载（overload）和函数重写（override）的基本规则](https://blog.csdn.net/inter_peng/article/details/53940179)
