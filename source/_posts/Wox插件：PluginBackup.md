---
title: Wox插件：PluginBackup
date: 2018-11-26 14:11:55
tags:
- C#
- wox
---

# Wox.Plugin.PluginBackup

[![Build Status](https://travis-ci.com/cildhdi/Wox.Plugin.PluginBackup.svg?branch=master)](https://travis-ci.com/cildhdi/Wox.Plugin.PluginBackup)

在闲逛的时候发现了这个[issue](https://github.com/Wox-launcher/Wox/issues/1995)，所以就做了这个备份wox插件与设置的wox插件。（考试周断断续续写的，质量不怎么高）

压缩和解压缩暂时用的是DotNetZip，不过它貌似不支持路径中有中文，所以名字带中文的插件（像有道词典）是没办法恢复的（压缩是压缩进去了，你可以手动解压缩hhh），等有时间了就把DotNetZip换掉。



### 安装

到Release中下载插件包（推荐，可以获取最新版本），解压到wox插件文件夹中并重启wox

或者打开wox，输入命令（无法保证为最新版本）

```
wpm install PluginBackup
```



### 使用

![usage](http://img02.sogoucdn.com/app/a/100520146/9075c33011d31cccbce69b87b175cbdc)

默认路径为桌面，也可以附加路径.

恢复后会自动退出wox使改动生效，可以重新打开。