---
title: 给avd模拟器安装magisk
date: 2022-06-11 12:49:40
urlname: install_magisk_on_avd
tags:
  - Android
  - Magisk
categories:
  开发
---

想给avd安装magisk，查了一下，多是针对三方模拟器的，github上有一个项目[MagiskOnEmulator](https://github.com/shakalaca/MagiskOnEmulator)是可以针对avd的，然而试了，不行。最终找到magisk项目本身就支持安装到avd（从24.3开始），命令是`build.py avd_patch ${镜像目录下的ramdisk.img}`。
<!-- more -->

具体没什么好说的，项目clone下来，运行命令，跟着报错信息一路解决就行了。我遇到的，需要注意的问题有如下这些：
- 项目有submodule，注意更新
- 保持网络通畅
- 环境变量`ANDROID_SDK_ROOT`
- avd要运行起来

安装完成之后，冷重启avd就行了。第一次启动，magisk可能提示要修复，确认就行了。再重启一次就OK了。