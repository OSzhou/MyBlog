---
banner: /css/images/tree_19.jpg
title: Socket原理知识摘要03
date: 2018-09-10 11:30:50
tags: Net
thumbnail: /css/images/tree_19.jpg
categories:
- iOS开发
---

### socket中TCP的三次握手建立连接详解
我们知道tcp建立连接要进行“三次握手”，即交换三个分组。大致流程如下：

- 客户端向服务器发送一个SYN J
- 服务器向客户端响应一个SYN K，并对SYN J进行确认ACK J+1

- 客户端再想服务器发一个确认ACK K+1

只有就完了三次握手，但是这个三次握手发生在socket的那几个函数中呢？请看下图：
![socket中发送的TCP三次握手](https://upload-images.jianshu.io/upload_images/2149459-0a5ca6309a72c1f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->
从图中可以看出，当客户端调用connect时，触发了连接请求，向服务器发送了SYN J包，这时connect进入阻塞状态；服务器监听到连接请求，即收到SYN J包，调用accept函数接收请求向客户端发送SYN K ，ACK J+1，这时accept进入阻塞状态；客户端收到服务器的SYN K ，ACK J+1之后，这时connect返回，并对SYN K进行确认；服务器收到ACK K+1时，accept返回，至此三次握手完毕，连接建立。
>总结：客户端的connect在三次握手的第二个次返回，而服务器端的accept在三次握手的第三次返回。

### socket中TCP的四次握手释放连接详解
上面介绍了socket中TCP的三次握手建立过程，及其涉及的socket函数。现在我们介绍socket中的四次握手释放连接的过程，请看下图：
![socket中发送的TCP四次握手](https://upload-images.jianshu.io/upload_images/2149459-2c7c1328f844f87c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图示过程如下：

- 某个应用进程首先调用close主动关闭连接，这时TCP发送一个FIN M；

- 另一端接收到FIN M之后，执行被动关闭，对这个FIN进行确认。它的接收也作为文件结束符传递给应用进程，因为FIN的接收意味着应用进程在相应的连接上再也接收不到额外数据；

- 一段时间之后，接收到文件结束符的应用进程调用close关闭它的socket。这导致它的TCP也发送一个FIN N；

- 接收到这个FIN的源发送端TCP对它进行确认。

这样每个方向上都有一个FIN和ACK。