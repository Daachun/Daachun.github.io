---
title: '[Unity]InputSystem输入系统初探'
index_img: 
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200610210443.gif
tags:
  - Unity
  - 输入系统
categories:
  - Unity
abbrlink: ba0c7706
date: 2020-03-14 20:30:26
---

# 什么是Input System？

Input System是Unity 2019版本中新加入的一个输入系统，旨在成为经典的Input类的替代品。

创建Input Syetem的宗旨是提供一个平台，统一管理不同外设的输入。开发者进行开发时，不用再去烦恼输入设备的种类，只用去管哪种类型的Action就行了。可以参考VRTK这个为VR设备提供的输入输出平台。

# 如何安装Input System？

> 注意：新的输入系统需要Unity2019.1以上和.NET4运行时。旧版.NET3.5运行时的项目中不起作用。

## 通过包管理器安装

新输入系统通过包管理器(Package Manager)进行安装，需要勾选“Advanced”中的“Show Preview Packages”选项。

![勾选高级选项](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/manual/Images/ShowPreviewPackages.png)

找到Input System后安装即可。

## 在Player Settings中开启

在Unity中创建项目后，默认使用经典输入方式。我们需要将“Active Input Handling”项调整为新输入系统，并**重启编辑器**。

![调整新输入系统](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/manual/Images/ActiveInputHandling.png)

当然可以选择Both，同时启用传统输入和新的输入系统。

# 获取输入

## 直接从设备获取输入

直接搬过来文档的代码

``` cs
using UnityEngine;
using UnityEngine.InputSystem;

public class MyPlayerScript : MonoBehaviour
{
    void FixedUpdate()
    {
        var gamepad = Gamepad.current;
        if (gamepad == null)
            return; // No gamepad connected.

        if (gamepad.rightTrigger.wasPressedThisFrame)
        {
            // 'Use' code here
        }

        Vector2 move = gamepad.leftStick.ReadValue();
        // 'Move' code here
    }
}
```
简单来说，可以通过UnityEngine.InputSystem这个命名空间里面的对应输入设备的类获取输入，比如Keyboard.current或者Mouse.current。

``` cs
private void FixedUpdate() 
{
  var keyboard = Keyboard.current;
  if (keyboard == null) return;
  if (keyboard.anyKey)
  {
    // do something
  }
}
```

这种方法还是有弊端的，针对不同设备，需要不同的API来调用。

## 通过InputAction获取输入

### 添加Player Input组件

通过Input Action作为中介来控制输入和响应更加合理，官方在这里给出的是使用“PlayerInput”组件来控制输入。

### 创建Input Action Asset

这里Player Input引导我们创建一个“Input Action Asset”，这个文件可以看作是Input Action的定义文件。（文件后缀名为.inputactions）

![创建Input Action Asset](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/manual/Images/PlayerInputCreateActions.png)

在弹出的窗口中我们就可以针对不同的Action为不同的设备定义按键了。

![](https://i.loli.net/2020/03/15/rNvK1fFlgGCX2De.png)

### 设置回调

通过Player Input组件我们可以在Inspector窗口中创建Unity中常用的回调，比如直接发送信息，广播信息，调用Unity事件，调用C#事件：

![创建回调](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/manual/Images/PlayerInputNotificationBehaviors.png)

选Invoke Unity Event这个就可以看到<del>我这个萌新比较熟悉的</del>事件调用的列表了，之后就可以开心的拖拖拽拽了！

> 注意：需要获取输入数值时（比如摇杆的Vector2数据），需要定义动态回调方法，参数里面要有一个“CallbackContext”类的参数，这个里面有相关Action的数据。
>
>![注意调用](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/manual/Images/MyPlayerActionEvents.png)

 ``` cs
 public class MyPlayerScript : MonoBehaviour
 {
    public void Fire(InputAction.CallbackContext context)
    {
        Debug.Log("Fire!");
    }
 }
 ```

### 注意

这部分内容设置输入和回调都是基于Player Input组件的，对比较复杂的输入方式，用这个组件的功能可能会让人感到麻烦。实际上还有其他方式来整理Input Action：

![](https://i.loli.net/2020/03/15/3q2GRdUMNfJLDoQ.png)

更多内容随后整理。

# 扩展阅读

  - [Unity新输入方案InputSystem介绍-序](https://www.bilibili.com/read/cv2938602)
  - [【Unity3d】新输入系统New InputSystem 0.1.2版本简介](https://blog.csdn.net/u010019717/article/details/86358975)
  - [Unity-Technologies/InputSystem](https://github.com/Unity-Technologies/InputSystem)