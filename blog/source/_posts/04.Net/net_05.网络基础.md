---
banner: /css/images/flower_02.jpg
title: 网络基础
permalink: {{ title }}
date: 2019-08-01 15:32:08
tags: Net
thumbnail: /css/images/flower_02.jpg
categories:
- iOS开发
---

***内容出自《图解HTTP》***
- ### TCP/IP 通信传输流
![](https://upload-images.jianshu.io/upload_images/2149459-6e9e75e73483d6e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->
- 利用 TCP/IP 协议族进行网络通信时，会通过分层顺序与对方进行通信。发送端从应用层往下走，接收端则往应用层往上走。
- 我们用 HTTP 举例来说明，首先作为发送端的客户端在应用层 (HTTP 协议)发出一个想看某个 Web 页面的 HTTP 请求。
- 接着，为了传输方便，在传输层(TCP 协议)把从应用层处收到的数 据(HTTP 请求报文)进行分割，并在各个报文上打上标记序号及端 口号后转发给网络层。
在网络层(IP 协议)，增加作为通信目的地的 MAC 地址后转发给链 路层。这样一来，发往网络的通信请求就准备齐全了。
- 接收端的服务器在链路层接收到数据，按序往上层发送，一直到应用
层。当传输到应用层，才能算真正接收到由客户端发送过来的 HTTP 请求。
![](https://upload-images.jianshu.io/upload_images/2149459-23d6cda4ca590a16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 发送端在层与层之间传输数据时，每经过一层时必定会被打上一个该 层所属的首部信息。反之，接收端在层与层传输数据时，每经过一层 时会把对应的首部消去。
- 这种把数据信息包装起来的做法称为封装(encapsulate)。

- ### 与 HTTP 关系密切的协议 : IP、TCP 和 DNS
   - 负责传输的 IP 协议
按层次分，IP(Internet Protocol)网际协议位于网络层。Internet Protocol 这个名称可能听起来有点夸张，但事实正是如此，因为几乎 所有使用网络的系统都会用到 IP 协议。TCP/IP 协议族中的 IP 指的就 是网际协议，协议名称中占据了一半位置，其重要性可见一斑。可能 有人会把“IP”和“IP 地址”搞混，“IP”其实是一种协议的名称。
IP 协议的作用是把各种数据包传送给对方。而要保证确实传送到对方 那里，则需要满足各类条件。其中两个重要的条件是 IP 地址和 MAC 地址(Media Access Control Address)。
IP 地址指明了节点被分配到的地址，MAC 地址是指网卡所属的固定 地址。IP 地址可以和 MAC 地址进行配对。IP 地址可变换，但 MAC 地址基本上不会更改。
使用 ARP 协议凭借 MAC 地址进行通信
IP 间的通信依赖 MAC 地址。在网络上，通信的双方在同一局域网 (LAN)内的情况是很少的，通常是经过多台计算机和网络设备中转 才能连接到对方。而在进行中转时，会利用下一站中转设备的 MAC 地址来搜索下一个中转目标。这时，会采用 ARP 协议(Address Resolution Protocol)。ARP 是一种用以解析地址的协议，根据通信方 的 IP 地址就可以反查出对应的 MAC 地址。
没有人能够全面掌握互联网中的传输状况
在到达通信目标前的中转过程中，那些计算机和路由器等网络设备只 能获悉很粗略的传输路线。
这种机制称为路由选择(routing)，有点像快递公司的送货过程。想 要寄快递的人，只要将自己的货物送到集散中心，就可以知道快递公 司是否肯收件发货，该快递公司的集散中心检查货物的送达地址，明 确下站该送往哪个区域的集散中心。接着，那个区域的集散中心自会 判断是否能送到对方的家中。
我们是想通过这个比喻说明，无论哪台计算机、哪台网络设备，它们 都无法全面掌握互联网中的细节。
![](https://upload-images.jianshu.io/upload_images/2149459-cb034a1002b4133a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   - 确保可靠性的 TCP 协议
按层次分，TCP 位于传输层，提供可靠的字节流服务。
所谓的字节流服务(Byte Stream Service)是指，为了方便传输，将大 块数据分割成以报文段(segment)为单位的数据包进行管理。而可 靠的传输服务是指，能够把数据准确可靠地传给对方。一言以蔽之， TCP 协议为了更容易传送大数据才把数据分割，而且 TCP 协议能够 确认数据最终是否送达到对方。
确保数据能到达目标
为了准确无误地将数据送达目标处，TCP 协议采用了三次握手 (three-way handshaking)策略。用 TCP 协议把数据包送出去后，TCP 不会对传送后的情况置之不理，它一定会向对方确认是否成功送达。
握手过程中使用了 TCP 的标志(flag) —— SYN(synchronize) 和 ACK(acknowledgement)。
发送端首先发送一个带 SYN 标志的数据包给对方。接收端收到后， 回传一个带有 SYN/ACK 标志的数据包以示传达确认信息。最后，发 送端再回传一个带 ACK 标志的数据包，代表“握手”结束。
若在握手过程中某个阶段莫名中断，TCP 协议会再次以相同的顺序发 送相同的数据包。
![](https://upload-images.jianshu.io/upload_images/2149459-1af8effd833a42e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

   - 负责域名解析的 DNS 服务
DNS(Domain Name System)服务是和 HTTP 协议一样位于应用层的
协议。它提供域名到 IP 地址之间的解析服务。 计算机既可以被赋予 IP 地址，也可以被赋予主机名和域名。比如
www.hackr.jp。
用户通常使用主机名或域名来访问对方的计算机，而不是直接通过 IP 地址访问。因为与 IP 地址的一组纯数字相比，用字母配合数字的表 示形式来指定计算机名更符合人类的记忆习惯。
但要让计算机去理解名称，相对而言就变得困难了。因为计算机更擅 长处理一长串数字。
为了解决上述的问题，DNS 服务应运而生。DNS 协议提供通过域名 查找 IP 地址，或逆向从 IP 地址反查域名的服务。
![](https://upload-images.jianshu.io/upload_images/2149459-9bc3e9fc2901a35f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- ### 各种协议与 HTTP 协议的关系
   - 学习了和 HTTP 协议密不可分的 TCP/IP 协议族中的各种协议后，我 们再通过这张图来了解下 IP 协议、TCP 协议和 DNS 服务在使用 HTTP 协议的通信过程中各自发挥了哪些作用。***（在浏览器地址栏输入访问一个网址的流程）***
![](https://upload-images.jianshu.io/upload_images/2149459-bc660dfafc8458c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)