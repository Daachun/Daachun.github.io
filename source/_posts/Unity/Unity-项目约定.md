---
title: Unity-项目约定
cover: >-
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201103095215.png
tags:
  - Unity
  - 资源
  - OpenProject
categories: Unity
description: 来自于Unity官方的项目标准
hide: false
top: false
abbrlink: e73be2c6
date: 2020-11-03 09:49:48
---

> 来源：[Open Projets:Conventions](https://docs.google.com/document/d/1-eUWZ0lWREFu5iH-ggofwnixDDQqalOoT4Yc0NpWR3k/edit#heading=h.23fnzd1otnih)

> Unity官方的第一个开放项目，项目最终会上架Steam。获取信息可以看[这里](https://github.com/UnityTechnologies/open-project-1)。这个标准可以直接拿来用到自己的项目中。

# 编码（Coding）

我们使用[.NET standards](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-type-members)作为基础，并在下方进行了更改。

## 命名

- 使用具有**描述性**和**准确性**的名称（即使这使得名称特别长），这易于阅读，易于移植。

- <u>不要</u>使用**缩写**。

- 使用公认标准的**首字母缩写**，例如：UI，IO。

- 方法名（Method）应为**动词**或者**动词短语**。

- 属性名（Property）应为**名词**、**名词短语**或者**形容词**。

- 布尔值（Boolean）名应为**肯定含义**的短语，你可以在名称前加上诸如“Is”，“Has”，“Can”的前缀，例如：IsActive，CanJump。

- 如果一个物体涉及到使用**多个属性**，使用**物体名称**作为属性名或者作用的前缀，例如：

  ```C# 
  Color _gameTitleColor;
  String _gameTitleString;
  TextMeshProUGUI _gameTitleText;
  ```

- 如果名字不在固定列表中，<u>避免</u>使用**数字**命名。例如：*animator1*，*animator2*。命名应**解释**两个属性的不同，例如：*playerAnimator*，*enemyAnimator*。


## 大小写

**定义**

camelCase：第一个字母为小写，之后单词的第一个字母为大写。

PascalCase：单词的第一个字母都是大写。如果一个单词是有两个字母的缩写，两个字母都大写。如果一个单词是有两个以上字母的缩写，则只有第一个字母大写。

- 类、方法、枚举、命名空间、公共字段、公共属性都用**PascalCase**。例如：`ClassName`，`GetValue`。
- 局部变量，方法参数用**camelCase**。例如：`previousValue`，`mainUI`。
- 私有字段、私有属性用camelCase，但是要以下划线开头。例如：`_inputReader`。
- 常量全大写，用下划线分割单词。例如：`GRAVITY_AMOUNT`。

## 编程（Programming）

- 保持字段和方法的**私有性**，除非你*需要*他们公开。

- 如果你想在Inspector中显示字段，而不想让其它类访问这个变量，不要用`public`，使用这个attribute：`[SerializeField]`和`private`。

  **注意：**这样做可能会收到警告：“Field is never assigned to, will always have its default value”，可以用`= default`来为字段指定默认值。

- 尽量避免使用单例（Singletons），请尝试使用ScriptableObjects（[1](https://www.youtube.com/watch?v=VBA1QCoEAX4)，[2](https://www.youtube.com/watch?v=raQ3iHhE_Kk&ab_channel=Unity)）来建立一个类似的、集中的、可以从多个对象中访问的类。

- 不要用`var`来声明变量，要准确声明类型。

- 避免使用静态变量。如果你确定绝对需要他们，请确保它们与*Fast Enter Play Mode*兼容，详情[点击这里](https://docs.unity3d.com/2019.3/Documentation/Manual/DomainReloading.html)。

- 不要在你的代码中使用硬编码的“魔法数字”。例如：角色移动`xInput * 0.035f`。为什么用这个数字？请将这个数字存储在一个有明确名称的字段中——也许再加个注释说明为什么选择这个特殊数字。

- 提交代码之前，删除所有没使用过的`using`命令（例如：`using System;`）。

## 格式化

- 每列用**一个Tab**来缩进代码，<u>不要用空格</u>。

- 大括号：如果内容为空，写在同一行。如果非空，每个占一行。例如：

  ``` C#
  public class EmptyBraces(){};
  public class NonEmptyBraces
  {
      // ...
  }
  ```

- 彼此包含的逻辑关系需要进行缩进，以表示层次。例如：

  ```C#
  public void FunctionName()
  {
      if(somethingHappended)
      {
          // ...
      }
  }
  ```

  

## 注释

- **重要：**不要画蛇添足。如果你认为任何人看一眼就能明白代码的作用，就<u>不要</u>添加注释。请给你的变量、类、方法起个好名字，让他们自己解释自己！

- 写行内注释为单独一行代码进行补充说明。

- 在每一个类上都要写摘要，以描述类的目的。如果一个类可读性不高、或者功能细节不直观时，可以选择性的在摘要中加入这些细节。例如：

  ``` C#
  /// <summary>
  /// This class manages save data
  /// </summary>
  ```

  提示：在IDE中输入三个“/”符号，一般会自动生成摘要。看[微软官方规范](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/documentation-comments)。

- 方法前写个解释它功能的注释，以防方法名不准确或者添加其他重要细节。你也可以用行内注释，例如：

  ```C#
  /// <summary> This function does this... </summary>
  public float CalculateBoundingBox(){ }
  ```

- 使用以`//TODO:`开头的注释来表示需要稍后完成的事情，这样你就不会忘记它。**注意：**这<u>不是</u>在鼓励你推送未完成的功能。

- 不要写`#region`或者`//--------`这类注释。

# 场景、层次结构

## 组织结构

- 在根（root）上创建空的GameObject并命名，创建视觉上的分隔符。例如：“--- Camera ---”，“--- Environment ---”，“--- Lighting ---”等。给这些分隔符添加“EditorOnly”标签，这样打包的时候就不会加入。

- 用来做容器的空物体，如果它里面只会包含1-2个对象，不要使用。

- UI部分

  - 尽可能使用同一个Canvas，只有在Canvas属性改变的时候才创建多个Canvas。

  - 每一个屏幕创建一个面板（Panel）。例如：主菜单、设置、暂停...

  - 用面板（Panel）作为容器，将UI元素进行分组。例如：一个设置标签和它的选项。

    面板能帮忙固定需要固定在一起的元素。例如：一些能量/物品UI在屏幕右下角。

## 命名

- 在GameObject命名中，不要用空格。

- 用PascalCase命名。例如：MainCharacter，DoorTrigger。

- 如果仅用PascalCase命名会产生歧义，用下划线连接两个概念。例如：MainHall_ExitTrigger，BossMinion_AttackWaypoints。

- **预制体（Prefabs）**：如果合理，可以重新命名实例。例如：一个Prefab Variant文件名称为“Protagonist_Scene1Variant”，如果你只用它一次，可以直接重命名为“Protagonist”。

  > **Prefabs:** Rename instances if it makes sense. Ex: A Prefab Variant file is called Protagonist_Scene1Variant, but once you use it you could rename it just Protagonist.

# 项目资源

## 命名

- 与[场景、层次结构](场景、层次结构)规则相同。
- 同一个文件夹下，如果文件相关，使他们名称**自然而然地归为一组**。
  - 一般来说，用对象所属的事物作为前缀。例如：PlayerAnimationController、PlayerIdle、PlayerRun等。
  - 如果这样合理的话，你可以使相似的对象保持在一起，即使他们属于不同事物，或者形容词导致他们分开。例如：在一个道具资产文件夹中，你可以用TableRound、TableRectangular来作为名称，这样它们就呆在一起，如果用RoundTable、RectangularTable，他们就不在一起了。

- 避免在名称中使用文件类型。例如：用ShinyMetal代替ShinyMetalMaterial。

## 文件夹

- 在根目录，将你的资源放到文件夹中，这些文件夹标识了游戏的区域、系统、位置。你可以创建子文件夹来分割不同类型的资源。

- 场景总是放在Scenes目录下。

- 不属于某个系统的脚本会放在Scripts文件夹目录下，可以建立子文件夹来分类。

- 总而言之：如果一个系统/功能只有脚本，那就在Scripts中创建一个文件夹。如果它有其它类型的资产，就把该文件夹放在根目录下，并按照资产类型添加子文件夹。

  例如：

  ![文件夹规范](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201103113812.png)