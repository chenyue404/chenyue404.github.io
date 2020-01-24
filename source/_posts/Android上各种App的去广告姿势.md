---
title: Android上各种App的去广告姿势
date: 2020-01-24 16:59:05
urlname: remove_app_ads_on_android
tags: 
  Android
categories: 
  玩机
---
## host
AdAday的默认列表先来一个

| 应用 | 域名 |
| --- | --- |
|铁路12306|ad.12306.cn|
|搜书大师|snssdk.com|
|下厨房|api.xiachufang.com/v2/ad/*|

## Activity
```shell
adb shell am start -n *
```
| 应用 | Activity | 是否需要root |
| --- | --- | --- |
|搜书大师|com.flyersoft.seekbooks/com.flyersoft.seekbooks.ActivityMain|是|
|饿了么|me.ele/me.ele.application.ui.home.HomeActivity|是|
|招商银行|长按图标，点击“账户”进入|否|
|     |浏览器打开`javascript:window.location.href='cmbmobilebank://cmbls/functionjump?id=1&action=gofuncid&funcid=6001003&keeponPath=true&source=today';`|否 |