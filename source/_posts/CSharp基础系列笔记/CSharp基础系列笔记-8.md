---
title: 'C#基础系列笔记-8'
categories:
  - 学习笔记
tags:
  - 'C#'
  - 编程
  - 学习
abbrlink: 10aa9840
date: 2018-10-04 17:06:12
---

添加状态系统。

<!---more--->

## 回合制对战

### 状态系统

在前几节中，我们定义了三种Skill类型。直接治疗类型和直接伤害类型的技能我们都进行了完整的实现，但没有对持续类型的技能进行实现。我们明确了持续类技能相当于给角色添加一个标记，标记的生命周期用状态系统来控制，标记实现具体的功能。

    enum StateType
    {
        DamageOverTime,
        HealOverTime,
    }

    class State
    {
        public StateType type;
        public int time;
        public int data;

        public State(StateType type, int time, int data)
        {
            this.type = type;
            this.time = time;
            this.data = data;
        }
    }

注意：状态类不能有权限给别人改写数据。若状态主动更改角色数据，那么就跟角色自身更改自身数据产生了耦合，不利于后期的设计。我们可以设计为角色单向依赖状态来更改自身数据。

我们改写Role，为其增加一个存放State的List，并添加AddState()方法。扩展BeHit()方法：

    case SkillType.AddState:
        AddState(skill.state);
        break;

    public virtual bool AddState(State state)
    {
        states.Add(state.Copy());
        return true;
    }

    public virtual bool StateEffect()
    {
        foreach (var state in states)
        {
            if(state.time <= 0)
            {
                continue;
            }
            if(state.type == StateType.DamageOverTime)
            {
                Console.WriteLine("DOT生效");
                CostHp(state.data);
            }
            else if(state.type == StateType.HealOverTime)
            {
                Console.WriteLine("HOT生效");
                HealHp(state.data);
            }
            state.time--;
        }
        // Todo: 遍历删除已经失效的状态。从列表删除元素时，注意从后往前删

        return true;
    }

这里有一个细节，观察AddState()方法，你会发现这里是使用state.Add(state.Copy())，这里的Copy()方法实际上是为传入的State创建了一个新的复制。这么做的原因是之前这些参数传入的都是引用，那么会导致StateEffect()方法直接修改传入的那个实例，这可能会导致一些问题。那么我们新申请内存空间，基于原来那个实例的模板（持续时间和值）新建一个实例以供修改即可。

要注意后期的垃圾处理。

在RoleAct()方法中添加判定时机：

    self.StateEffect();

之后在Main中为角色添加新技能吧：
    
    // 添加dot技能
    State state = new State(StateType.DamageOverTime, 3, 5);
    player.skills.Add(Skill.CreateAddStateSkill(SkillTarget.Enemy, state));

至此状态添加功能基本实现。

### 状态消除与回合显示优化

上一节我们还没有实现遍历删除失效状态的方法，并且返回值也并没有相关的内容，这一节我们完善这一步骤。

首先要注意要删除List中某个或几个元素，直接Remove()传入一个参数相同的实例是没有用的。如果这里我们用foreach的话是实现不了这个功能的。

那么怎样才能遍历List之后删除某个特定条件的元素呢？

> [C#遍历List并删除某个或者几个元素的方法](https://www.cnblogs.com/hedianzhan/p/9130296.html)
 
>用for正序遍历删除,只删除了一个姓名为Tang的学生。为什么会出现这种情况呢？
>这是因为当i=1时，满足条件执行删除操作，会移除第一个Tang，接着第二个Tang会前移到第一个Tang的位置，即游标1对应的是第二个Tang。
>接着遍历i=2，也就跳过第二个Tang。

得到答案：用for循环倒序遍历删除，即判断每一个state实例的持续时间，若time<=0就删除掉，不进入结算。更改StateEffect()：
    
    for (int i = states.Count -1 ; i >= 0; i--)
    {
        if (states[i].time <= 0)
        {
            Console.WriteLine("{0}身上的{1}状态失效", name, states[i].type);
            states.Remove(states[i]);
        }
    }

