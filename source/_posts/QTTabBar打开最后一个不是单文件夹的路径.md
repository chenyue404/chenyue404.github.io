---
title: QTTabBar打开最后一个不是单文件夹的路径
date: 2021-11-20 09:40
urlname: open_the_last_path_that_is_not_a_single_folder_qttabbar
tags: 
  - Windows
  - QTTabBar
categories: 
  玩机
---
在浏览代码的时候，经常遇到包名文件夹，需要一层一层点进去，很麻烦，所以写了个QTTabBar的脚本来解决这个问题。效果就是打开前面对象的，最深一个，非单文件夹的路径。

将[这个链接](https://gist.github.com/chenyue404/a450719f3a57ce76485cec30a7728648#file-)保存为js文件，放到电脑上合适的地方，然后拖放到QTTabBar的命令栏上。点击即可运行。  

默认作用对象是当前文件夹，如果当前选中的项目是单个文件夹，则作用对象是选中文件夹。

拖放之后，可以编辑图标、快捷键等信息。