---
title: 利用KDE Connect和NirCmd，在手机上控制电脑看小说
date: 2022-05-02 20:11:40
urlname: control_PC_by_Phone_with_in_KDE_Connect_and_NirCmd
tags:
  - Windows
  - Android
  - KDE Connect
  - NirCmd
categories:
  玩机
---

## 背景
我经常看小说，然后为了自己的颈椎和眼睛的健康考虑，在电脑上看肯定是比手机上看更好。为此也是花了很多工夫，还写过几个油猴脚本和AHK脚本，但这些都不是这次的主题。
然后还是从健康的角度出发，一直坐着也不太好，本来腰椎就不好，所以最好还是经常站起来。那么问题来了，站起来的时候，手不好操作键盘啊。为此我还买了一个mini键盘，6个按键的，可以自己设置键值。但是这个键盘是有线的，不太方便。
KDE Connect也用了一段时间了，它支持在手机上执行电脑上预设好的命令。今天又发现这些命令是可以复制成URL（`kdeconnect://runcommand/***`）的，这好像是scheme啊，这是不是意味着可以通过外部程序来调用呢？然后试了一下，果然可以，那么基础条件就都齐了。
<!-- more -->

## 实现
思路就是**翻页器App**通过scheme调用**手机上的Kde**执行**电脑上的kde**的各种预定义的**NirCmd**的命令。然后就是从尾到头，一步一步来。
### NirCmd
[NirCmd](https://www.nirsoft.net/utils/nircmd.html)是一个非常**强大**的Windows命令行工具，具体有多强大，可以看[完整文档](https://www.nirsoft.net/utils/nircmd2.html#using)。
下载，解压，直接丢到`c:\windows\`目录下就行了。
这次的目的是看小说，那基本上就只需要实现一个功能，就是模拟`PageUp`、`PageDown`、`➡️`和`⬅️`按键，相应的命令就是`nircmd sendkeypress pageup|pagedown|right|left`。

### 电脑上的KDE Connect
[KDE Connect](https://kdeconnect.kde.org)的[下载](https://kdeconnect.kde.org/download.html)[安装](https://userbase.kde.org/KDEConnect/zh-hans#.E5.AE.89.E8.A3.85)和基本使用方法我就不讲了，挺简单的。
然后添加命令，如下操作：
选取已连接的手机，打开插件设置
![选取已连接的手机，打开插件设置](https://raw.githubusercontent.com/chenyue404/image_storage/main/2022-05-02_20-52-13.png)  

打开执行命令的设置，添加命令
![打开执行命令的设置，添加命令](https://raw.githubusercontent.com/chenyue404/image_storage/main/2022-05-02_20-53-40.png)

### 手机上的KDE Connect
安装，连接电脑，打开**运行命令**页面
![](https://raw.githubusercontent.com/chenyue404/image_storage/main/20220502210357.jpg)

长按命令复制URI
![长按命令复制URI](https://raw.githubusercontent.com/chenyue404/image_storage/main/20220502210401.jpg)

### 翻页器App
就是在屏幕上放俩按钮，点击事件分别是PageUp和PageDown，长按事件是Left和Right。
核心代码就是执行KDE Connect提供的scheme，代码如下
```Kotlin
startActivity(Intent(Intent.ACTION_VIEW, command.toUri()))
```
当然App我已经写好了，直接[下载](https://github.com/chenyue404/PageTurner/releases)安装就好了。
运行，在配置页面填上从KDE Connect复制的对应命令的URI，然后保存就行了。
然后点击上、下按钮就可以触发电脑上的翻页动作了。

## 总结
虽然这次的目的是看小说，所以只做了翻页相关操作。但实际上是可以运行更多命令的，道理是一样的，调整手机上的布局和命令就行了。灵活一点的话，直接用Tasker画场景就好了，就不用单独写一个翻页器来操作了。