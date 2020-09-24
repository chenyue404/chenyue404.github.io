---
title: Win10给右键菜单添加“在此处打开命令窗口（管理员）”
date: 2020-06-13 06:41:36
urlname: add_cmd_in_right_menu_on_windows10
tags: 
  Windows
categories: 
  玩机
---

在win10的桌面上，按住shift，并点击右键，就会发现比不按shift的时候，多了两个选项：“在此处打开命令行窗口”和“在此处打开PowerShell窗口”，很是方便。但是，有些时候，我们需要用管理员权限打开命令行窗口进行某些操作，怎么办？以前的方法是：打开开始菜单，输入cmd，可以看到“命令提示符”，右键-以管理员身份运行。然后再一路cd到需要操作的路径。那有没有更简单的办法呢？有的。

win+r，输入“regedit”，打开注册表编辑器，定位到“\HKEY_CLASSES_ROOT\Directory\Background\shell”，可以看到shell下面有一个cmd和一个Powershell，这俩就是我们在右键菜单里看到的“在此处打开命令行窗口”和“在此处打开PowerShell窗口”，那怎么增加一个“在此处打开命令窗口（管理员）”呢？

1. 在shell上右键，新建-项，输入名字“runas”
2. 选中runas，双击右侧的“默认”，将数值数据修改成“在此处打开命令窗口（管理员）”
3. 在runas上右键，新建-项，名称command，双击右侧的“默认”，数值数据改成“`cmd.exe /s /k pushd "%V"`”
4. 在runas上右键，新建-字符串值，名称填“Extended”，数值数据留空
5. 在runas上右键，新建-字符串值，名称填“NoWorkingDirectory”，数值数据留空
6. 在runas上右键，新建-DWOR(32位)值，数值名称ShowBasedOnVelocityId，数值数据639bc8

4-6步，可以参考系统自带的cmd项，一摸一样地抄一遍就行了。
完成，现在在桌面上按住shift并单击右键，即可看到多出来一个“在此处打开命令窗口（管理员）”。

如果想要不按shift就可以看到，删除Extended项即可。
如果想排序，在runas前面加上数字就行了。比如0cmd，1runas，这俩就会排到前面去。