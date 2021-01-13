---
title: 'C#基础系列笔记-6'
categories:
  - 学习笔记
tags:
  - 'C#'
  - 编程
  - 学习
abbrlink: f712b547
date: 2018-10-04 17:03:24
---

优化技能系统，添加技能类型。

<!---more--->

## 回合制对战

### 特殊技能的实现

实现玩家半血后进入愤怒状态，攻击力变为3倍的功能。

将BeHit方法抽象，在Player和Monster类中重写BeHit方法，以玩家为例：

    public override void BeHit(int cost_hp)
    {
        base.BeHit(cost_hp);
        // 判断“狂暴”
        if (hp * 1.0f / maxHp < 0.5f)
        {
            Console.WriteLine("愤怒！攻击力大大增加！");
            foreach (var skill in skills)
            {
                skill.SetDamageMutiply(3);
            }
        }
    }


注意：int类型的hp除以同为int类型的maxHP得到只能是int，在这里也就是非0即1，要转化成float类型。

### 优化技能系统

先搞清楚回合制游戏中的常见技能种类
1. 直接伤害类   - 直接
2. 直接治疗类   - 直接
3. 持续伤害类   - 持续
4. 持续治疗类   - 持续
5. 定身、沉默、眩晕等debuff     - 持续
6. 无敌、增加攻击、防御等buff   - 持续

归纳：持续类技能使用**状态**来实现。那么如果一个技能给角色附加了一个状态，这之后状态的效果、持续时间、消失与这个技能没有关系，和状态系统有关。我们用状态系统来控制这些状态。

例如上一节中“愤怒”的实现可以归为buff的状态。

> 好的框架设计，最重要的是对逻辑的理解。

基于以上理解，将技能系统进行细化，增加“技能目标”的定义。

将Skill分为3种类型：
1. 直接伤害类
2. 治疗类
3. 添加状态


    enum SkillType
    {
        Damage = 1,     // 直接伤害
        Heal,           // 治疗
        AddState        // 添加状态
    }

完善技能类的内容，添加static工具类用于创建技能，删除原来的构造函数：

    public static Skill CreateDamageSkill(int damage)
    {
        Skill s = new Skill();
        s.skillType = SkillType.Damage;
        s.damage = damage;
        return s;
    }

    public static Skill CreateHealSkill(int heal)
    {
        Skill s = new Skill();
        s.skillType = SkillType.Heal;
        s.heal = heal;
        return s;
    }
    
同时更改相关的技能创建语句。将Role类内的BeHit方法进行优化：

    public virtual void BeHit(Skill skill)
    {
        switch (skill.type)
        {
            case SkillType.Damage:
                hp -= skill.damage;
                if (hp < 0) hp = 0;
                break;
            case SkillType.Heal:
                hp += skill.heal;
                if (hp > maxHp) hp = maxHp;
                break;
        }
    }
并增加一个获取目标Role的static方法：

    static Role GetSkillTarget(Skill skill, Role self, Role enemy)
    {
        if (skill.type == SkillType.Damage)
        {
            return enemy;   
        }else if(skill.type == SkillType.Heal)
        {
            return self;
        }
        return null;
    }

完善相关语句，完成技能选择、治疗技能、伤害技能的实现。