---
layout:       post
title:        "第四十章 生成QTCreator项目进行编译"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - 流式
    - WebRTC
    - Streaming
---

[访问 2img.ai 官网以获取更多AI/AIGC信息](https://2img.ai)
[访问 个人技术博客: fuqifai.github.io](https://fuqifai.github.io)

本文大部分配图使用微信小程序【字形绘梦】免费生成。
![](/img/小程序码.png)

AIGC技术讨论群
![](/img/RA群永久二维码.png)

## 第四十章 生成QTCreator项目进行编译



## 基础情况

一般的，我们可以使用VSCode在Linux下进行编译，调试和运行。

不过我们有时候倾向于需要强大的UI库，此时我们需要用到Qt这个开源库和IDE处理。

以下，我们尝试使用gn的指令，生成QT的工程，以便于在QtCreator这个IDE中打开你的项目。

为了防止你忘记， GN这个指令是Ninja编译体系的文件，我们之前在编译Webrtc源代码的时候用到过。如果你没有，现在还可以单独下载一份，然后放入到系统PATH中，自然就可以使用。

## 具体操作

下面的编译命名，最后的–ide=”qtcreator” 会在out目录下生成一个qtcreator的目录和项目。可以用qtcreator正常打开

```
gn gen out/my --args='target_os="linux" target_cpu="x64" is_debug=true is_component_build=false use_sysroot=false is_clang=false use_lld=false treat_warnings_as_errors=false rtc_include_tests=false rtc_build_examples=false use_custom_libcxx=false use_rtti=true' --ide="qtcreator"
```

在WINDOWS, 我已经准备了好了个all\_debug-qt.bat的文件，就正常编译webrtc的方法即可。 最终windows的内容生成在

![第四十章 生成QTCreator项目进行编译](https://www.shxcj.com/wp-content/uploads/2024/09/image-636.png)

这个感觉是生成了QT编译WebRTC的工程而已。

![第四十章 生成QTCreator项目进行编译](https://www.shxcj.com/wp-content/uploads/2024/09/image-638.png)

Linux下的在这里

![第四十章 生成QTCreator项目进行编译](https://www.shxcj.com/wp-content/uploads/2024/09/image-637.png)

熟悉Qt开发体系的你应该可以继续操作了。

需要注意的几点：

1.  不确保较差项目可以很好的生成
2.  Qt中的Qmake方式可能在这个生成的项目中无法继续工作，因为我们可以看到其还是还有ninja编译系统的文件内容。
3.  Qt本身编译系统的情况也在不断地变化中，请注意使用Qt5，还是其余等版本 。（本文产品内容仅在Qt5.0 体系下运行成功）
