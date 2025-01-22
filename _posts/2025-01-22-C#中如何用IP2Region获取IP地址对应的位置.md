---
layout:       post
title:        "C#中如何用IP2Region获取IP地址对应的位置"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - C#
    - IP2Region
    - DB
---

[访问 2img.ai 官网以获取更多AI/AIGC信息](https://2img.ai)
本文大部分配图使用微信小程序【字形绘梦】免费生成。
fuqifai.github.io

最近需要用到根据IP地址，查询对应的城市位置。找了一些网页资料，结果很差，尤其是CSDN，不仅文章内容质量低下，浪费了很多时间，代码不可用。而且还各种限制。所以本文更新使用方法后，也不会在CSDN上发布。

来，直接展开。

备注：本文中的截图/配图 使用微信小程序【字形绘梦】制作，谢谢该软件的免费支持. AI课程大学堂 [2img.ai](http://2img.ai/)

![C#中如何用IP2Region获取IP地址对应的位置](https://www.shxcj.com/wp-content/uploads/2025/01/aab8aff8-f21e-43f2-a265-2b2d723638c5.jpg)

## 什么是IP2Region

官方的主要介绍有：

ip2region – 是一个离线IP地址定位库和IP定位数据管理框架，10微秒级别的查询效率，提供了众多主流编程语言的 `xdb` 数据生成和查询客户端实现。

## 第一步 找到IP库IP2Region

该库的地址：https://github.com/lionsoul2014/ip2region

可以下载全库，使用data目录下的ip2region.xdb文件

也可以直接下载地址 https://github.com/lionsoul2014/ip2region/tree/master/data/ip2region.xdb

这个是最核心的离线IP库，后续你的代码中需要用到它。放到你的磁盘位置上。

## 第二步 在C#代码中使用

首先在VisualStudio的包管理器中，直接安装依赖包，执行以下命令

Install-Package IP2Region.Net

直接代码

using IP2Region.Net.Abstractions;

using IP2Region.Net.XDB;

ISearcher searcher = new Searcher(CachePolicy , “刚才第一步你下载的XDB文件的全路径”);

String location=searcher.Search(“你的IP地址，比如10.10.9.9”);

最终的效果，在我们的DEMO中如下

![C#中如何用IP2Region获取IP地址对应的位置](https://www.shxcj.com/wp-content/uploads/2025/01/afb79d9e-f9fd-4344-b4c0-5c4f8c3a2507.png)

非常方便，高效，快捷。

更多关于地址的生成，数据新增，更多查询细节。

可以直接访问官网地址获取更多信息：

https://github.com/lionsoul2014/ip2region

有任何问题，请二维码添加技术交流群。探讨更多AIGC能力

![C#中如何用IP2Region获取IP地址对应的位置](https://www.shxcj.com/wp-content/uploads/2025/01/e8ae4ed4-a96e-4d10-891a-2ae3e3cf0621.png)

海量AI知识文库，大家一起飞。 技术交流群

AIGC训练营 公众号

![C#中如何用IP2Region获取IP地址对应的位置](https://www.shxcj.com/wp-content/uploads/2025/01/475923fa-49af-49d5-ab13-87384166d161.png)

