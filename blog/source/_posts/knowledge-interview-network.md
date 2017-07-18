---
title: 知识点总结--网络协议
date: 2017-07-15 18:30:14
categories: "知识点总结"
tags:
	- "知识点总结"
	- "网络协议"
	- "HTTPS"
---


# 网络协议知识点

<font size=4>

## 1.HTTP
### HTTP协议中比较容易混淆的点

1 http1.1/http1.0区别

* 缓存处理方面
* 带宽优化及网络连接的使用
* Host头处理
* 长连接（最重要区别 1.1支持）

2 1.1和1.0存在的问题

* 1.0：传输数据时，每次都需要重新建立连接
* 1.x: 在传输数据时，都是明文传输，客户端服务端无法验证对方身份
* 1.x: 在使用过程中header里携带的内容过大，增加了传输成本
* 1.1：虽然1.1支持了keep-alive弥补了多次创建连接产生的延迟，但keep-alive使用过多同样给服务端带来性能压力

3 cookie和session的区别

* cookie:是客户端的解决方案，Cookie是由服务端发给客户端的特殊信息，这些信息已文本文件的方式存在客户端，然后客户端每次向服务端发送请求的时候都会带上这些特殊信息。

![cookie](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_http_cookie.png)


![cookie 2](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_http_cookie2.png)

* session: 是另一种记录客户状态的机制。session保存在服务器上
* session工作原理：
 * 第一步创建session
 * 在创建session的同时，服务器为该session创建唯一的sessionID
 * 在session创建之后，就可以调用session相关的方法往session中添加内容
 * 当客户端再次发送请求的时候，会将sessionID带上，服务端接收请求后根据sessionid找到相应的session

* 区别：
  * 存放位置不同：cookie保存在客户端； session保存在服务端
  * 存取方式不同：cookie:保存的是ascii字符串； session可以存取任何类型数据
  * 安全性(隐私策略)cookie存在客户端可能会被其他程序篡改
  * 有效期不同：cookie可以设置很长时间；session依赖于当中的id的，当id设置为-1，关闭浏览器session也就失效了
  * 对服务端造成的压力：由于cookie保存在客户端，如果并发量过多的话，选择cookie会减少压力
 

## 2.HTTPS
### HTTPS是什么
* HTTPS不是一个单独的协议。是工作在加密连接(SSL/TLS)上的常规HTTP协议。通过在TCP和HTTP之间加入TLS(Trasnport Layer Security)来加密

* SSL/TLS协议
 * SSL协议是一种安全传输协议，TLS是ssl v3.0升级版

![https01](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_https01.png) 
 

* HTTP传输速度
 * 通讯慢
 * SSL必须进行加密处理


### TLS/SSL握手
* 密码学原理
 * 对称加密：加密数据跟解密数据用的密钥是一样的.以DES算法为代表；
 * 不对称加密：SSL层采用这种。以RSA算法为代表；
 		* 私有密钥：一方保管
 		* 共有密钥：双方共有

  
* 数字证书
 * 是互联网通讯标志通讯双方信息的一串数字
 * 为什么要有数字证书？这个是第三方权威机构发布的

* SSL与TLS握手整个过程
* ![https02](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_https02.png)


[也可以参考腾讯Bugly专栏](http://blog.csdn.net/tencent_bugly/article/details/72626127)



## 3.TCP/IP
![网络分层](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_tcp%3Aip.png)

* 物理层
* 链接层
 * 链接层协议：以太网规定了电信号的分组方式 head:data
 * mac 地址:每台网卡出厂时地址，48位
 * 广播
 ![广播](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_broadcast.png)
 如果在同一个子网环境，1与2通讯，则3，4，5都会收到通知，然后根据mac地址是否匹配进行回应，效率较低
 
* 网络层 
  * 作用：使得我们可以区分不同的计算机是否在同一个子网环境下
  * ip协议
  * ip数据包
  * arp协议：需要一种机制能够从ip地址得到MAC地址。ARP是解决同一局域网内主机或路由器的ip地址和硬件地址之间的映射问题

* 传输层
 * 当数据包从互联网发过来，我们需要知道这个数据包是要给哪个应用程序使用的。
 * 传输层功能：就是建立端口到端口之间的通讯 



## 4.HTTPS中的加密算法相关
* 密钥
 * 密钥是一种参数,它是在使用密码算法过程中输入的参数。同一个明文在相同的密码算法和不同的密钥计算下会产生不同的密文。
 ![密文](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_https%E7%AE%97%E6%B3%9501.png)
 
 * 对称密钥：在加密和解密过程中使用的密钥是相同的。常见的对称加密算法：DES,3DES,AES,RC5,RC6。 优点是计算速度快，缺点是密钥需要通讯两端共享。
 * 不对称密钥。服务端会生成一对密钥，一个私钥保存在服务端，，公钥可自由发布供任何人使用 RSA
* 密钥 RSA加密简单过程 
 * 服务端生成配对的公钥私钥
 * 私钥保存服务端，公钥发送给客户端
 * 客户端使用公钥加密明文传输给服务端
 * 服务端使用私钥解密密文得到明文
* 思考：浏览器和服务端传输数据时，有可能内容被替换，如何保证数据是真实服务端发送，没有被篡改？
* 数字签名
 * 数字签名
  * 1.用于验证传输的内容是不是真实服务器发送的。2.验证数据有没有被篡改过。它就验证这两件事
  * 是非对称加密的应用场景。不过它是反过来用私钥加密，通过配对的公钥来解密
 ![签名生成](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_%E7%AD%BE%E5%90%8D%E7%9A%84%E7%94%9F%E6%88%90.png)
 
 ![签名校验](https://raw.githubusercontent.com/sheltonliu/sheltonliu.github.io/hexo/blog/MarkdownPhotos/2017/07/15/android_%E7%AD%BE%E5%90%8D2.png)
 
* 数字证书
 * 如何确保现在使用的公钥就是真实服务器发送的呢？ 数字证书简称CA,是一种认可的凭证。 









