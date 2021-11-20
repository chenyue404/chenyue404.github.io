---
title: windows清除默认打开方式
date: 2021-11-20 09:30
urlname: clear_default_open_on_windows
tags: 
  - Windows
categories: 
  玩机
---
以js文件为例，注册表俩地方  

1. `计算机\HKEY_CLASSES_ROOT\.js`
默认项的数据数值改回`JSFile`
2. `计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\FileExts\.js\OpenWithProgids`
只保留默认和JSFile，其他的项删了