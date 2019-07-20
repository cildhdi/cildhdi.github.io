---
title: Windows下build skia
date: 2018-06-08 14:57:14
tags: 'lib'
---



## skia 

[官方文档](https://skia.org)

## 说明

- 以下所有命令均在命令行（cmd）中执行，在power shell中可能出现操作不成功但不报错的情况。
- 安装git并配置以及python 2.7.x（python 3会出错）。
- 需要科学上网，下载的东西还挺多。



## 安装depot_tools

​	下载[depot_tools](https://storage.googleapis.com/chrome-infra/depot_tools.zip) 并解压到任意位置，然后将解压的文件夹添加到系统环境变量中（需要移动到python的路径之前），最后在cmd中输入ninja并回车，没有报错的话就可以下一步了。

## 下载skia

```
git clone https://skia.googlesource.com/skia.git
```
进入skia目录后，执行以下命令来安装依赖

```
python tools/git-sync-deps
```

## 生成build文件

​	生成build文件需要运行bin目录下的GN，为了方便，直接把bin目录添加到环境变量中。然后运行

```
bin/gn gen out/Shared --args='is_official_build=true is_component_build=true'
```

来生成动态链接库的构建文件。还有其他的生成方式，具体见官方说明。

## 编译

```
ninja -C out/Shared
```

可能会编译一段时间，完成后在out/Shared文件夹中就可以找到对应的dll文件。

## 使用vs

在运行生成构建文件时，传入 --ide=vs 参数，然后运行

```
python gn/gn_meta_sln.py
```

就会在out文件夹里生成sln文件，找到后打开就可以开始编译了。

## 有关x86与x64

​	上述方法生成的目标平台都是x64，因此在我使用vs来构建时，我尝试在配置管理器中添加了Win32的配置，用最后生成的dll文件写了一个简单的测试，编译时还是报错目标平台不匹配。在官方文档中有提到如何构建32位版本的skia，可能是由于网络问题或者其他的，一直都没有构建成功，暂时也没找到其他的办法，只好放弃了..........