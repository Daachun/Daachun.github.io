---
title: 'C#基础系列笔记-7'
categories:
  - 学习笔记
tags:
  - 'C#'
  - 编程
  - 学习
abbrlink: 801585d1
date: 2018-10-04 17:04:57
---

战斗功能抽象化。

<!---more--->

## 回合制对战

### 优化技能目标功能

在之前一节中，我们实现的技能目标功能是将伤害技能和治疗技能的目标写死了，但实际上，治疗技能不光能够为自身回复，还能为友军、敌方回复。就此功能，我们对Skill类进行扩展，添加一个SkillTarget的枚举类型属性来记录技能的目标。

换言之，在全局的GetSkillTarget()方法中我们可以直接判断技能目标类型来确定返回值，而不是只依据技能类型：

    static Role GetSkillTarget(Skill skill, Role self, Role enemy)
    {
        if (skill.target == SkillTarget.Enemy)
        {
            return enemy;   
        }else if(skill.target == SkillTarget.Self)
        {
            return self;
        }
        return null;
    }

有得必有失，在Skill的Create方法中，必须提供SkillTarget作为参数传入。


### 战斗功能抽象化

一直到现在，我们的战斗过程还是用简单的while循环驱动，并由玩家和怪物的血量来控制。

我们可以发现，回合制游戏战斗过程具有对称性：我动一下，敌方动一下。那么根据这一特性，将战斗过程抽象化实现。同时可以优化战斗过程的显示方法。

定义全局方法RoleAct()，它描述每回合特定Role的行动。

    static bool RoleAct(Role self,Role other)
    {
        Console.WriteLine("---------------");
        if (self == player)
            {
            Console.ForegroundColor = ConsoleColor.Green;
        }
        else
        {
            Console.ForegroundColor = ConsoleColor.Gray;
        }
        Console.WriteLine("{0}的回合", self.name);


        Skill skill = self.SelectSkill();
        Role target = GetSkillTarget(skill, self, other);
        target.BeHit(skill);

        if (target == self)
        {
            Console.WriteLine("{0}的生命:{1}",self.name , self.hp);
        }
        else
         {
            Console.WriteLine("{0}被攻击，生命:{1}", other.name,other.hp);
        }

        if (other.hp <= 0 || self.hp <= 0)
            {
            return false;
        }
        return true;
    }

这样就可以在循环中使用：

    RoleAct(player, monster);
    RoleAct(monster, player);

这种抽象化的过程肯定不是最优解，但作为学习素材足够了。