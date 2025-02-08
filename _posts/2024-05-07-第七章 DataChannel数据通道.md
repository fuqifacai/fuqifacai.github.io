---
layout:       post
title:        "第七章 DataChannel数据通道"
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

作为WebRTC的三大模块之一，DataChannel支持短消息（short message）二进制和文本数据的传输，因此，对于通常以传输音视频为主的WebRTC来说，当需要传输音视频以外的数据的时候就有很大用处了。 `DataChannel`看起来和`Websocket`实现的功能很相似。DataChannel模型最初是基于webSocket建立的，具有简单可设置的send方法和onMessage方法。

但是它们之间的区别是很明显的：

1、RTCDataChannel通信是在peer与peer之间直接连接，所以RTCDataChannel比WebSocket更快，因为WebSocket需要服务器中转；但是相应的，WebRTC依靠ICE Servers来穿透NAT，有的场景下可能会多一层TURN服务器的转发。

2、WebSocket协议是基于TCP传输的，它能够保证数据的安全有序地到达；而RTCDataChannel是基于SCTP传输协议（SCTP是一种TCP、UDP同级的传输协议）， 默认情况下是可靠有序传输。但是它也可以配置是否进行可靠的传输，这就意味着有可能会通过丢失数据来达到性能的要求，这使得RTCDataChannel更为灵活。这里说一下，为什么会有配置不可靠传输的需求呢？因为实时通信对时间是非常敏感的，以音视频为例，它可以容忍间接性的数据包丢失，可以通过算法来填补这个丢失的数据，因此WebRTC对及时性和低延时的要求要比数据传输的可靠性要求更高。

3、WebSocket构造需要一个url，与服务器建立连接，创建一个唯一的SocketSessionId；DataChannel 的连接依赖于一个RTCPeerConnection对象，当RTCPeerConnection建立起来后，可以包含一个或多个RTCDataChannel。 还有其他的一些区别，例如大文件传输时的拥塞控制、加密等，不在列举。

有一点很重要，就是DataChannel的创建是在生成本地offer之前，这样才能在生成offer后，使offer中包含DataChannel的信息。

## **关于DataChannel的配置参数**

你可以查看`<WebRTC/RTCDataChannelConfiguration.h>`中的各种参数及说明。

```
/// <summary>
/// 創建DataChannel管道，
/// </summary>
void QLSignalConnection::CreateDataChannel()
{
    QL_LOG("Begin to Create Data Channel");
    //this is for C++ activate the connection,currently,it won't be triggered
    webrtc::DataChannelInit config;
    config.maxRetransmits = 30; //最大重傳次數
    config.ordered = true; //是否順序接受數據
    mDataChannel=mPeerConnection->CreateDataChannel("ID", &config);
     
    mDataChannel->RegisterObserver(this);
    QL_LOG("End of Data Channel Creation");
}
```

在这个示例中之所以要带 options 参数，是因为在端与端之间传输文件时，必须要保证文件内容的有序和完整，所以在创建 RTCDataChannel 对象时，你需要给它设置一些参数，即需要设置 **ordered** 和 **maxRetransmits** 选项。当 ordered 设置为真后，就可以保证数据的有序到达；而 maxRetransmits 设置为 30，则保证在有丢包的情况下可以对丢包进行重传，并且最多尝试重传 30 次。

通过实践证明，如果你在创建 RTCDataChannel 对象时不设置这两个参数的话，那么在传输大文件（如 800M 大小的文件）时就很容易失败。而设置了这两个参数后，传输大文件时基本就没再失败过了.

RTCDataChannel API支持灵活的数据类型。它的API是模仿WebSocket设计的，并且支持JavaScript中的二进制类型如Blob、ArrayBuffer和ArrayBufferView，另外还支持字符串。

这些类型对于文件传输和多玩家的游戏来说意义重大。

WebRTC的RTCdatachannel是一个双工通道，是点对点连接的，中间没有server中转，所使用的协议是SCTP协议。

SCTP和TCP\\UDP的对比如下，

![第七章 DataChannel数据通道](https://www.shxcj.com/wp-content/uploads/2024/09/image-590.png)

