---
layout:       post
title:        "辣妈之野望 8 — DeepSeek-r1和Deepseek-v3使用对比"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - Ollama
    - LLM
    - Deepseek
---

## 辣妈之野望 8 — DeepSeek-r1和Deepseek-v3使用对比


之前我们其实在对比分析所有的Ollama框架中使用的各种大模型的能力时，有简单用过Deepseek-r1 的模型。

考虑到v3版本的能力貌似更强大，我们来进行下对比。

本文大部分配图使用微信小程序【字形绘梦】免费生成。
![](/img/小程序码.png)

AIGC技术讨论群
![](/img/RA群永久二维码.png)
先上总结：

1.  r1 在某些内容表现比v3 好。
2.  在代码能力方面和准确度还是 v3 好很多，非常高质量 。

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/bbce154a-4f8a-4c39-af1a-15d3ca92a9c6.jpg)

### **Deepseek官方开源地址：**

DeepSeek-r1 的 Github地址：[https://github.com/deepseek-ai/DeepSeek-R1?tab=readme-ov-file](https://github.com/deepseek-ai/DeepSeek-R1?tab=readme-ov-file)

DeepSeek-v3 的 Github地址：https://github.com/deepseek-ai/DeepSeek-V3

个人技术博客： [fuqifacai.github.io](https://link.juejin.cn/?target=https%3A%2F%2Ffuqifacai.github.io)

更多技术资讯下载: [2img.ai](https://link.juejin.cn/?target=https%3A%2F%2F2img.ai)

相关配图由微信小程序【字形绘梦】免费生成

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/2dc7b058-b526-47bd-ad3a-59d6f8160f65.jpg)

### **关于Deepseek各模型的情况如下**

<table class="has-fixed-layout"><tbody><tr><td><strong>模型规模</strong></td><td><strong>最低 </strong><strong>GPU</strong><strong>显存</strong></td><td><strong>推荐 </strong><strong>GPU</strong><strong> 型号</strong></td><td><strong>纯 </strong><strong>CPU</strong><strong>内存</strong><strong>需求</strong></td><td><strong>适用场景</strong></td></tr><tr><td>1.5B</td><td>4GB</td><td>RTX 3050</td><td>8GB</td><td>个人学习</td></tr><tr><td>7B、8B</td><td>16GB</td><td>RTX 4090</td><td>32GB</td><td>小型项目</td></tr><tr><td>14B</td><td>24GB</td><td>A5000 x2</td><td>64GB</td><td>专业应用</td></tr><tr><td>32B</td><td>48GB</td><td>A100 40GB x2</td><td>128GB</td><td>企业级服务</td></tr><tr><td>70B</td><td>80GB</td><td>A100 80GB x4</td><td>256GB</td><td>高性能计算</td></tr><tr><td>671B</td><td>640GB+</td><td>H100 集群</td><td>不可行</td><td>超算/云计算</td></tr></tbody></table>

我们基础框架还是ollama+web UI ,有不清楚使用教程的，可以看之前的 [辣妈之野望 1 — 部署个人大模型框架](https://www.shxcj.com/archives/8813) 和

[辣妈之野望 3 — Ollama各大模型全方位对比1](https://www.shxcj.com/archives/8835)

### 官方性能总览对比

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/424ba7ce-c4a0-4416-ab47-83c744183328.png)

### 注意点：

#### 1 容量

首先r1的实际大小在10G左右。Ollama轻松下载即可使用

但是v3版本，672B参数，400G左右的体量，下载都吃力。这个每个同学自己使用时候注意磁盘和网络流量情况。

尤其是运行时，直接提示需要的显存。

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/4465c6ae-50b8-4db7-a206-a756b810e71f.png)

#### 2 Ollama版本

v3要求ollama版本至少0.5.5以上。r1版本中并没有此要求

这个自行在cmd中ollama -v 查询版本信息。

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/ff398fbd-d35a-4a50-bbac-016bc1834b10.jpg)

## 基本数据对比

r1的数据,最主要的是参数7.62B ,存储尺寸4.7GB

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/5ada5cde-068b-445e-a3ee-2a71add601eb.png)

v3的数据,最主要的是参数671B,存储尺寸404GB

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/6cf2f7d1-0406-4972-9beb-1b03a66e9ccc.png)

## 开始实际效果对比

我们尝试问几个问题。看它的回复。

基本上我会问几个维度的问题

### 1 你的大模型数据是截止到何时的

r1:截止到2024年7月 v3:截止日期是2024年7月

### 2 中国有个微信小程序【字形绘梦】，请问你知道是什么产品吗？

r1:没有找到讯息。但给了一些建议和纠错的讯息。还不错。

v3:有答案，但是是错误的回复。反而误导了。差评。

这个问题，天工AI的回复非常正确。所以国内质量对比就差很多了。

不过查了下GPT也是同样错误的答案。

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/2a2b0c2b-b72f-43e9-ae33-06b45fad12fa.jpg)

### 3 请作为一个风水算命大师，给我算算今日财运如何。我的生日是1988年8月8日，职业是工程师

r1: 算命的答案让我惊讶。非常好。如下图

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/de0fb23b-88ac-4439-a66a-5f0de78b43fa.png)

v3: 答案让我失望，甚至比r1都差。这是为啥？

以下是截图

![辣妈之野望 8 -- DeepSeek-r1和Deepseek-v3使用对比](https://www.shxcj.com/wp-content/uploads/2025/02/fe12ea84-29e6-422e-a703-c6959a74cc38.png)

### 4 请作为一名C#编程专家，回到我在C#中如何使用RestRequest库请求multipart/form-data的Post内容，直接给我代码接口

r1: 代码生成的内容非常的丰富，但是经过正式的历练，显得准确度不高。

甚至过多的内容，影响了最终的效能和决策。开发人员拿到这种代码直接使用的话，搞不好就会出现比自己亲自写的都要累。需要修改的地方太多了。有的甚至是错误的引导。

v3: 代码质量效果非常好。可以和微软的phi4 相比了。 因此看来，就代码这块而言V3 全量版本胜出太多了。

更多技术资讯进群
![RA技术群](/img/RA群永久二维码.png)