---
layout:       post
title:        "第三十八章 Ubuntu或Debian下的情况"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - 流式
    - WebRTC
    - Streaming
---

[访问 2img.ai 官网以获取更多AI/AIGC信息](https://2img.ai)
本文大部分配图使用微信小程序【字形绘梦】免费生成。
fuqifai.github.io

## 第三十八章 Ubuntu或Debian下的情况



## Ubuntu情况

在Ubuntu下是肯定可以安装的。网上资料很多不多赘述。

## Debian下情况

以下是在Debian系统上的编译步骤。M90下验证。M108也同样可以。

<table class="has-fixed-layout"><tbody><tr><td>步骤</td><td>备注</td></tr><tr><td>复制源代码到Debian下</td><td></td></tr><tr><td>下载GN 命令工具，将其所在目录添加到系统PATH中。最简单的直接下载后复制到 /sbin/目录下。整个目录是Debian存放很多CMD的地方。参考 <a href="https://ewiki.51arena.com/pages/viewpage.action?pageId=35651737">2 Debian中如何添加依赖库路径到系统PATH中</a></td><td>下载地址<a href="https://chrome-infra-packages.appspot.com/dl/gn/gn/linux-amd64/+/latest">https://chrome-infra-packages.appspot.com/dl/gn/gn/linux-amd64/+/latest</a>下载该文件后解压，放在PATH中能执行到即可。</td></tr><tr><td>CMD中切换到webrtc-checkout/src/目录下，执行gn gen out/Default出现右侧的错误</td><td></td></tr><tr><td>apt-get install python后出现右侧错误</td><td>那就去执行那句</td></tr><tr><td>还是继续出现错误。奇怪的是出现MacOS的字样，即便指定target_os=”linux”还是一样。</td><td></td></tr><tr><td>更新好Deps后，出现这个奇怪的测试相关的编译错误。 不知道哪里可以去掉测试的开关</td><td>解决方法：　<a href="https://ewiki.51arena.com/pages/viewpage.action?pageId=35651773">3 gmock错误 （含解决方法）</a></td></tr><tr><td></td><td>执行 gn gen out/CentOS后一切正常。此时需要使用ninja 编译。需要首先安装ninja 及其依赖</td></tr></tbody></table>

## 可能遇到的错误

### 错误libappindicator

![第三十八章 Ubuntu或Debian下的情况](https://www.shxcj.com/wp-content/uploads/2024/09/image-635.png)

原因是在Debian11上，这个库的名字被修改了。

libappindicator ,修改成了下图中的第一个

![第三十八章 Ubuntu或Debian下的情况](https://www.shxcj.com/wp-content/uploads/2024/09/image-633.png)

### 可能需要用到Boost库

1.  下载boost zip包，自行解压
2.  执行 /bootstrap.sh
3.  按照输出提示执行 ./b2 输出结果，可以看到静态库等在stage目录下

![第三十八章 Ubuntu或Debian下的情况](https://www.shxcj.com/wp-content/uploads/2024/09/image-634.png)

4.  在GCC或者QT 中引入这些.a 即可。
5.  运行时期，可能需要.so ，这个要注意。

