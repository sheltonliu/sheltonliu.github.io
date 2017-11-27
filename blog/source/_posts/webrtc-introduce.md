---
title: WebRTC 介绍
date: 2017-11-22 13:20:34
categories: "webrtc"
tags:
	- "webrtc"
---

<font size=4>

webRTC的整体框架
![图1](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/11/27/webrtc-01.png)


三个服务器
![三个服务器](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/11/27/webrtc-03.png)

webRTC的呼叫流程
![图2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/11/27/webrtc-02.png)


上述序列中，标注的场景是ClientA向ClientB发起对聊请求，调用描述如下：

·ClientA首先创建PeerConnection对象，然后打开本地音视频设备，将音视频数据封装成MediaStream添加到PeerConnection中。

·ClientA调用PeerConnection的CreateOffer方法创建一个用于offer的SDP对象，SDP对象中保存当前音视频的相关参数。ClientA通过PeerConnection的SetLocalDescription方法将该SDP对象保存起来，并通过Signal服务器发送给ClientB。

·ClientB接收到ClientA发送过的offer SDP对象，通过PeerConnection的SetRemoteDescription方法将其保存起来，并调用PeerConnection的CreateAnswer方法创建一个应答的SDP对象，通过PeerConnection的SetLocalDescription的方法保存该应答SDP对象并将它通过Signal服务器发送给ClientA。
·ClientA接收到ClientB发送过来的应答SDP对象，将其通过PeerConnection的SetRemoteDescription方法保存起来。

·在SDP信息的offer/answer流程中，ClientA和ClientB已经根据SDP信息创建好相应的音频Channel和视频Channel并开启Candidate数据的收集，Candidate数据可以简单地理解成Client端的IP地址信息（本地IP地址、公网IP地址、Relay服务端分配的地址）。

·当ClientA收集到Candidate信息后，PeerConnection会通过OnIceCandidate接口给ClientA发送通知，ClientA将收到的Candidate信息通过Signal服务器发送给ClientB，ClientB通过PeerConnection的AddIceCandidate方法保存起来。同样的操作ClientB对ClientA再来一次。

·这样ClientA和ClientB就已经建立了音视频传输的P2P通道，ClientB接收到ClientA传送过来的音视频流，会通过PeerConnection的OnAddStream回调接口返回一个标识ClientA端音视频流的MediaStream对象，在ClientB端渲染出来即可。同样操作也适应ClientB到ClientA的音视频流的传输。


注：这里的sdp和Candidate都是本地生成的。sdp是本地native方法生成的，Candidate是本地方法监听回调的对象。需要将这两个描述符通过服务端进行P2P连接

客户端代码长连接采用的websocket


参考：

[webrtc官网](https://webrtc.org/)

[WebRTC之Android客户端代码介绍](http://blog.csdn.net/chenhande1990chenhan/article/details/70862208)

[WebRTC Experiments & Demos](https://www.webrtc-experiment.com/)

[WebRTC 的 Android 2 Android 实现: CSDN上的demo](http://blog.csdn.net/youmingyu/article/details/53192714)

[WebRTC 前端实时通信技术: 腾讯云文章](https://cloud.tencent.com/community/article/566106)

[github book http服务/前端技术/html5/与服务器端通讯：： 这个介绍的很全面,socket-io,websocket都有介绍](http://blog.hszofficial.site/TutorialForWebTech/http%E6%9C%8D%E5%8A%A1/%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF/html5/%E4%B8%8E%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E9%80%9A%E4%BF%A1.html)

[WebRTC学习笔记（二）之文档学习：：这个是一系列的文章,介绍的也很好](http://blog.sina.com.cn/s/blog_724faf110102vnf0.html)

[WebRTC学习笔记（三）之NAT类型](http://blog.sina.com.cn/s/blog_724faf110102vnhc.html)

官网推荐的文档《WebRTC - APIs and RTCWEB Protocols of the HTML5 Real-Time Web 》

小记：关于技术方面的文档,google比百度靠谱的多


//coturn服务器的配置
http://www.linuxdiyf.com/linux/32285.html
