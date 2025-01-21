---
layout:       post
title:        "部署你个人的精简版大模型框架Ollama+WebUI Lite"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - Ollama
    - LLM
    - WebUI Lite
    - GPT
    - 个人大模型
---

[访问 2img.ai 官网以获取更多AI/AIGC信息](https://2img.ai)
本文大部分配图使用微信小程序【字形绘梦】免费生成。
fuqifai.github.io

## 9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite

我们知道Ollama是开源大模型框架。本文，我们就来部署一个自己的本地大模型+精简的UI访问框架，实现本地的GPT体系。

先来看看最终的产品界面

![9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite](https://www.shxcj.com/wp-content/uploads/2025/01/image-138-1024x625.png)

## 安装Ollama

目前直接访问官网记得得到安装方式。如下图中，每个平台都有。

![9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite](https://www.shxcj.com/wp-content/uploads/2025/01/image-133.png)

以前传统的方式，使用curl 的命令获取如下图。

1.  Console run curl -fsSL [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh  
    ![9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite](https://www.shxcj.com/wp-content/uploads/2025/01/image-134.png)  
    ![9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite](https://ewiki.51arena.com/download/attachments/77955424/image-2024-11-4_14-15-1.png?version=1&modificationDate=1737374938660&api=v2)

## 安装好后查看

打开浏览器，访问

1.   [http://localhost:11434/](http://localhost:11434/) ， 看到页面内容就算成功了。
2.  或者 ，使用命令 ollama serve 看控制台输出，如下图

![9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite](https://www.shxcj.com/wp-content/uploads/2025/01/image-135.png)

![9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite](https://www.shxcj.com/wp-content/uploads/2025/01/image-136.png)

## 安装大模型

上面我们安装的是一个Ollama的框架，还是需要下载大模型：

比如执行: ollama run qwen2:7b-instruct-q4\_0

或者有时候必须要用到最新的，很奇怪 ollama run llama2

[**这里可以下载所有的模型** https://github.com/ollama/ollama](https://github.com/ollama/ollama)    

## Ollama模型下载后的位置

macOS: ~/.ollama/models   
Linux: /usr/share/ollama/.ollama/models  
Windows: C:Users<username>.ollamamodels

Ollama默认只允许127.0.0.1:11434访问，我们可以修改配置，让所有IP都可以访问

修改文件/etc/systemd/system/ollama.service , 在\[Service\]这块最下面新增

Environment=”OLLAMA\_HOST=0.0.0.0″

保存，重启Ollama服务即可（Ollama Serve)

## 如何有Web界面的访问方式：

1.  [https://github.com/open-webui/open-webui](https://github.com/open-webui/open-webui) 这是完整的库，比较新。但是安装使用相对复杂
2.  [https://github.com/ollama-webui/ollama-webui-lite](https://github.com/ollama-webui/ollama-webui-lite) 这个已经停止更新，但是非常精简，适合个人使用。本人尝试用这个。

安装方式：

本地获取 [https://github.com/ollama-webui/ollama-webui-lite](https://github.com/ollama-webui/ollama-webui-lite) 库之后。在目录下执行npm ci 即可。

完成后执行npm run dev 

一切成功输入内容如下：

![9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite](https://www.shxcj.com/wp-content/uploads/2025/01/image-137.png)

打开浏览器，输入上述中的地址你就可以看到内容：比如 http://localhost:3000

![9-13 部署你个人的精简版大模型框架Ollama+WebUI Lite](https://www.shxcj.com/wp-content/uploads/2025/01/image-138-1024x625.png)

是不是很像你的GPT页面了？

Settings中还有很多本地的设置和简易的功能就留给大家挖掘了。

有任何问题，请二维码添加技术交流群。探讨更多AIGC能力

海量AI知识文库，大家一起飞。 技术交流群

![一张照片如何生成简易数字人视频 - 喂饭教程](https://www.shxcj.com/wp-content/uploads/2025/01/d71f77e9-14cb-4b05-9f78-964ead8ff787.png)

AIGC训练营 公众号

![一张照片如何生成简易数字人视频 - 喂饭教程](https://www.shxcj.com/wp-content/uploads/2025/01/a7bbe03c-0530-40f9-830b-f3ab7c301a10.png)

