---
title:  Android手机通过USB线连接电脑上网
date: 2024-02-29 17:09:12
urlname: install_magisk_on_avd
tags:
  - Android
categories:
  玩机
---

很久以前写过一篇博客介绍如何让[Android设备通过USB线连接电脑网络](https://chenyue404.blogspot.com/2018/08/androidusb.html)，但是文章的最后也说了，这个办法是有遗憾的，就是`没法在Android上科学上网了`。接下来就介绍另外一种方法来弥补这个遗憾，即让没有任何网络连接（没插卡、没wifi、开飞行模式）的Android设备通过usb连接电脑即可上网，同时还可以兼容各种网络代理。
<!-- more -->
## 操作步骤
废话不多说，先讲怎么作。需要的东西就两样，[adb](https://developer.android.com/tools/adb)和[sing-box](https://sing-box.sagernet.org/)（写这篇博客的时候，sing-box的版本是1.8.7）。
1. 电脑上下载adb和sing-box。下载之后解压就好了，都是不需要安装的。
2. 手机上安装sing-box的apk。
3. 分别保存下面的配置文件到手机和电脑

<details>
<summary>手机端配置文件</summary>

```json
{
  "dns": {
    "servers": [
      {
        "tag": "dns_114",
        "address": "114.114.114.114",
        "strategy": "ipv4_only",
        "detour": "out_socks"
      }
    ]
  },
  "inbounds": [
    {
      "type": "tun",
      "tag": "tun-in",
      "interface_name": "tun0",
      "inet4_address": "172.19.0.1/30",
      "inet6_address": "fdfe:dcba:9876::1/126",
      "auto_route": true,
      "strict_route": true
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    },
    {
      "type": "dns",
      "tag": "dns-out"
    },
    {
      "type": "socks",
      "tag": "out_socks",
      "server": "127.0.0.1",
      "server_port": 1080,
      "udp_over_tcp": true
    }
  ],
  "route": {
    "rules": [
      {
        "protocol": "dns",
        "outbound": "dns-out"
      },
      {
        "port": 53,
        "outbound": "dns-out"
      }
    ],
    "final": "out_socks",
    "auto_detect_interface": true,
    "override_android_vpn": true
  }
}
```
</details>

<details>
<summary>电脑端配置文件</summary>

```json
{
    "inbounds": [
        {
            "type": "socks",
            "tag": "socks-in",
            "listen": "127.0.0.1",
            "listen_port": 1280
        }
    ],
    "outbounds": [
        {
            "type": "direct",
            "tag": "direct"
        },
        {
            "type": "block",
            "tag": "block"
        },
        {
            "type": "dns",
            "tag": "dns-out"
        }
    ],
    "route": {
        "rules": [
            {
                "protocol": "dns",
                "outbound": "dns-out"
            },
            {
                "port": [
                    53
                ],
                "outbound": "dns-out"
            }
        ],
        "final": "out_socks"
    }
}
```
</details>

4. 导入配置文件到手机上的sing-box内，运行
5. 电脑上执行
```shell
sing-box run -c config.json
```
6. 电脑上再开一个命令窗口，执行
```shell
adb reverse tcp:1080 tcp:1280
```

DONE
如无意外，这个时候手机已经正常打开网页了🎉🎉🎉
有些app会有异常，是因为它们直接判断了设备既没连接wifi也没连接数据网络。

## 大致原理
很简单，上网可以非常简略地分为两步：
1. 通过dns查询域名获取ip
2. 和这个ip对应的服务器建立连接

第6步adb就是把手机的1080端口映射到电脑的1280端口，即访问手机的1080端口就等同于访问电脑的1280端口。
电脑上sing-box就是在本地127.0.0.1:1280上开了一个socks5的代理，规则是dns流量和目的地是53端口的流量通过dns的outbounds出去，其余流量最终走out_socks出去，即前面开的socks5代理，默认就是direct直连。
来到Android这边，Android这边提供了一个114的dns，通过out_socks出去，dns本身是udp流量，所有在out_socks的socks代理上打开了udp_over_tcp。

原理很简单，配置文件我也尽量精简，应该很容易看明白。如果有其他代理分流的需求，根据sing-box的文档再修改配置文件即可。这就比[gnirehtet](https://github.com/Genymobile/gnirehtet)灵活多了😁。