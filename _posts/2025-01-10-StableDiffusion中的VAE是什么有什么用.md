---
layout:       post
title:        "StableDiffusion中的VAE是什么，有什么用"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - SD
    - VAE
    - AIGC
---

[访问 2img.ai 官网以获取更多AI/AIGC信息](https://2img.ai)
[访问 个人技术博客: fuqifai.github.io](https://fuqifai.github.io)

本文大部分配图使用微信小程序【字形绘梦】免费生成。
![](/img/小程序码.png)

AIGC技术讨论群
![](/img/RA群永久二维码.png)

1）VAE 简介
VAE 全称 Variational Auto Encoder （变分自编码器），是 stable diffusion 整个模型算法的组成部分之一，位于运作流程的末端，作用是让 stable diffusion 生成的图像颜色更鲜艳、细节更锐利；同时也能在一定程度上改善局部细节的生成质量，如手部、服装、脸部等，我们可以简单地把它理解为一种“滤镜”。

在机器学习中，变分自动编码器( VAE ) 是由 Diederik P. Kingma 和Max Welling引入的一种人工神经网络架构。它属于概率图模型和变分贝叶斯方法家族的一部分。
除了被视为自动编码器神经网络架构之外，变分自动编码器还可以在变分贝叶斯方法的数学公式中进行研究，通过与变分分布的参数相对应的概率潜在空间（例如，多元高斯分布）将神经编码器网络连接到其解码器。
因此，编码器将大型复杂数据集中的每个点（例如图像）映射到潜在空间内的分布中，而不是该空间中的单个点。解码器具有相反的功能，即同样根据分布从潜在空间映射到输入空间（尽管在实践中，在解码阶段很少添加噪声）。通过将点映射到分布而不是单个点，网络可以避免过度拟合训练数据。这两个网络通常使用重新参数化技巧一起训练，尽管可以单独学习噪声模型的方差。

 ![](/img/20250110-1.png)

2）如何判断是否需要使用 VAE 模型
VAE 模型不能单独使用，要配合大模型一起使用。
但不是所有大模型都需要使用 VAE, 因为一些稳定的大模型本身就自带 VAE ，可以直接生成色彩正常的图像，再使用 VAE 可能会起到反作用。
而有一些大模型在融合训练的时候 VAE 被损坏了，导致生成的图像色彩不佳，就需要搭配外挂 VAE。

我们可以通过以下 2 种方式判断一款大模型是否需要配合 VAE 使用：
① 根据图像生成效果判断
判断是否需要使用 VAE 最直接的方式就是看生成图像的效果，如果颜色发灰、发白，与你看到的参考图像效果有明显差异，那么就需要启用 VAE。

② 看大模型的发布提示
一些大模型在发布的时候，如果需要使用外挂 VAE，会在发布声明里说明。如下图的Aniflatmix 大模型，在模型介绍页面就明确有提到需要配合 VAE模型使用。我们可以按提示在网上搜索相关资源 ，然后下载使用就可以了。

 ![](/img/20250110-2.png)

3）VAE 模型下载
C 站可用 VAE 模型：https://civitai.com/models
VAE 模型和其他 AI 绘画模型一样，资源主要集中在 Civitai 和 Huggingface 两个平台上。
如果是国内，当然也可以上Liblib获取，如下图

 ![](/img/20250110-3.png)


这里也为大家推荐几款常用的 VAE 模型，可以解决大部分的图像色彩发灰问题。
① 通用 VAE
ema-560000: stabilityai/sd-vae-ft-ema-original at main 
mse-840000: stabilityai/sd-vae-ft-mse-original at main 

Stability AI 官方推出过 2 款 VAE ，vae-ft-ema-560000-ema-pruned 和 vae-ft-mse-840000-ema-pruned，它们适用于各种风格的大模型，能有效提升画面色彩的鲜艳程度。
从使用经验来说，ema-560000 会明显地修改画面的细节，mse-840000 则没有那么明显。

 ![](/img/20250110-4.png)

② 风格化 VAE
ClearVAE：ClearVAE - v2.3 | Stable Diffusion VAE | Civitai 
kl-f8-anime2：https://civitai.com/models/23906 
Anything：https://civitai.com/models/110630/anything-model-vae-v40 
有些 VAE 则只适合特定风格的大模型，大家在下载的时候可以仔细阅读 VAE 制作者的使用说明。
比如 ClearVAE、kl-f8-anime2、anything 等，都是适合动漫风大模型的 VAE。
不同的 VAE 下，图像生成效果会略有差异，大家可以根据实际情况对比选择：

 ![](/img/20250110-5.png)

4）VAE 的使用与管理
VAE 模型下载后，需要放进根目录的 models\VAE 文件夹内；如果要卸载某个VAE，也是在VAE 文件夹中删除对应的文件就可以了。

 ![](/img/20250110-6.png)
在 WebUI 中，选择好大模型后，在“外挂 VAE ” 的下拉栏中选择对应的模型就可以了；不需要 VAE 的话，默认“自动”或者“无”。

 ![](/img/20250110-7.png)
4）要注意的是：
在API请求的过程中，VAE的设置是一个字符串，并且不是一个全局的设定，而是在每次请求的时候，同Prompt一样的一次包含的内容中。
以下是我们的产品(微信小程序  字形绘梦 )中的请求数据结构示意

 ![](/img/20250110-8.png)

 最后备注：本文有部分内容摘自网络内容，如有侵权请及时告知后删除。
