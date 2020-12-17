---
banner: /css/images/tree_07.jpg
title: App启动优化
date: 2020-06-25 10:31:30
tags: OC
thumbnail: /css/images/tree_07.jpg
categories:
- iOS开发
---

##### 一、冷启动和热启动
- 定义：
   - 1.关于冷启动：业界对冷启动的定义没有问题，普遍认为是手机开机后第一次启动某个APP。
   - 2.关于热启动：
        对热启动有两种不同的看法：
        - 1.有些人认为是按下home键把APP挂到后台，之后点击APP的icon再拉回来到前台算是热启动；
        - 2.也有些人认为是手机开机后在短时间内第二次启动APP（杀掉进程重启）算是热启动（此时dyld会对部分APP的数据和库进行缓存，所以比第一次启动要快）。

***（一般认为APP从后台拉起到前台的时间没啥研究的意义，所以在统计启动时间时，会倾向于后一种说法，不过具体怎么定义看个人，知道其中的区别就好）***
>如果启动过，内存页就已经加载进内存了，内存页如果要销毁只能是被覆盖，所以这时候的加载进物理内存的时候，实际内存中已经存在值，所以启动就会变快。

<!--more-->

##### 二、优化的方向
1.减少流程的数量。
2.减少每个流程的消耗。

##### 三、APP的启动过程
![](https://upload-images.jianshu.io/upload_images/2149459-f2c4f65fb7c36bf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 四、启动时间分布

在Xcode中，可以通过设置环境变量来查看App的启动时间，`DYLD_PRINT_STATISTICS`和`DYLD_PRINT_STATISTICS_DETAILS`。
![](https://upload-images.jianshu.io/upload_images/2149459-4282a86147430e34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
根据xcode打印可以看到:
动态库的加载
- rebase/binding  
    - 方法、类、Category这些需要rebase和 binding
    - binding 
        Mach-O _DATA建立指针，指向外部函数，我们编译的时候，共享缓存库里边的代码指向的就是这里，链接的时候，通过dyld指向外部具体真实的地址。PIC技术
fishook就是利用这个中转，hook的系统c函数。
- Objc setup  
    - 类的初始化过程在这里边
    ①读取二进制文件的 DATA 段内容，找到与 objc 相关的信息 
    ②注册 Objc 类，ObjC Runtime 需要维护一张映射类名与类的全局表。当加载一个 dylib 时，其定义的所有的类都需要被注册到这个全局表中； 
    ③读取 protocol 以及 category 的信息，把category的定义插入方法列表 (category registration)， 
    ④确保 selector 的唯一性

- initializer    
    load等函数，C++的构造函数

耗时比较多的
加载自己的库可能比较耗时

两个阶段分别是 pre- main  和  main启动到viewDidAppear:
![](https://upload-images.jianshu.io/upload_images/2149459-a04c355aa8a08685.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
pre-main过程：
![](https://upload-images.jianshu.io/upload_images/2149459-34d3df6d07464d42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
什么是dyld?
    dyld是一个专门用来加载动态链接库的库。 
    执行从dyld开始，dyld从可执行文件的依赖开始, 递归加载所有的依赖动态链接库。
##### 五、 Dyld加载过程
加载Mach-O文件，加载一块比较大的内存空间，地址重定向
启动dyld的main函数
  - 配置xcode的环境变量 (arguments)
  - 加载共享缓存库，UIKit 和 Foundation这些

加载LoadCommons段一系列加载
加载动态库，包括自己的和系统的，实例化主程序的imageLoader，添加进imageList
链接主程序

开始初始化主程序
 *runtime配合加载，runtime加载loadImages方法，这个方法就是prepare_load_methods(mh)
 *加载load和C++构造函数
找main函数
配置（环境变量） —  加载（共享动态库，主程序，动态库） — 链接（链接主程序，动态库） — 初始化（程序）
 - 加载动态库过程，动态链接器dyld需要做如下操作
1.分析依赖的动态库
2.找到动态库的Mach-O文件
3.打开文件
4.验证文件
5.在系统的内核注册文件签名
6.对动态库的每个segment调用mmap()

针对这一步优化
>1.减少非系统的动态库
2.使用静态库而不是动态库
3.合并非系统的动态库为一个

>1.动态库尽可能少，不要多与6个，如果多了，尽可能合并
2.20000个类  800毫秒
3.swift静态语言，性能更高

##### 六、ObjC SetUp
由于之前2步骤的优化，这一步实际上没有什么可做的。几乎都靠 Rebasing 和 Binding 步骤中减少所需 fix-up 内容。因为前面的工作也会使得这步耗时减少。
##### 七、Initializers
- 以上三步属于静态调整，都是在修改`__DATA segment`中的内容
现在开始动态调整，在堆和栈中写入内容。 工作主要有：
1、`Objc`的`+load()`函数 
2、C++的构造函数属性函数 形如`attribute((constructor)) void DoSomeInitializationWork() `
3、非基本类型的C++静态全局变量的创建(通常是类或结构体)(non-trivial initializer) 比如一个全局静态结构体的构建，如果在构造函数中有繁重的工作，那么会拖慢启动速度
***可以做的优化有：***
①使用 +initialize 来替代 +load 
②不要使用 atribute((constructor)) 将方法显式标记为初始化器，而是让初始化方法调用时才执行。 比如使用 dispatch_once(),pthread_once() 或 std::once()。也就是在第一次使用时才初始化，推迟了一部分工作耗时。 也尽量不要用到C++的静态对象。
>从效率上来说，在`+load` 和`+initialize`里执行同样的代码，效率是一样的，即使有差距，也不会差距太大。 但所有的`+load `方法都在启动的时候调用，方法多了就会严重影响启动速度了。 就说我们项目中，有200个左右`+load`方法，一共耗时大概1s 左右，这块就会严重影响到用户感知了。 而`+initialize`方法是在对应 Class 第一次使用的时候调用，这是一个懒加载的方法，理想情况下，这200个`+load`方法都使用`+initialize`来代替，将耗时分摊到用户使用过程中，每个方法平均耗时只有5ms，用户完全可以无感知。
#####八、二进制重排
iOS的虚拟内存和物理内存：
![](https://upload-images.jianshu.io/upload_images/2149459-6813bd4a98988521.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
之前所有应用都加载进内存条中，会有安全问题，会访问到其他人的软件中去

iOS虚拟内存有4G的

CPU通过  操作系统映射表   映射虚拟内存和物理真实内存 
也就是我们访问虚拟内存，都会走到映射表，然后映射到真实的物理内存。（每个映虚拟内存对应一个映射表）
（mmp）分页加载，提高内存的利用率![](https://upload-images.jianshu.io/upload_images/2149459-594826c17d457028.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
内存是以字节为单位，内存以页管理 PAGE，Linux每1页4KB，iOS每一页16KB

用这种分页加载，用到哪块内存就加载到真实内存
上述P2加载的时候，发现真实内存没有，缺页，CPU会临时加载


如果启动的时候缺页比较多（缺页异常），就会不断去映射到物理内存，就会卡顿
iOS的内存地址，与编译顺序有关，所以如果方法在不同的类，类的加载差距比较大，那么就会加载比较多的页，就会浪费时间。
iOS系统还会对加载的每一页做签名认证，所以更加消耗时间。

system trace
Main Thread
Virtual Memory  File Backard Page In

ldyd
order_file    自己写一个文件，里边指定方法的顺序，可以将启动要调用的方法，都放到里边。
libc-order在objc源码里边，这就是二进制重排

所有OC方法获取
用fishhook 去 HOOK objc_msgSend()方法，可以拿到所有的函数
可以用Clang插庒的方式AOP

##### 八、main过程：
![](https://upload-images.jianshu.io/upload_images/2149459-8b027b61da9c161d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- Main函数之后
> 1.懒加载
2.发挥CPU的价值，用多线程的方式。
3.框架的搭建尽量避免stroyboard   和  xib

- 分阶段加载：
 - 在didFinishLunch中启动的：
   * 埋点功能、
   * Crash 采集
   * 网络配置等

- 在首页viewDidappear
   * 初始化SDK，友盟SDK、人脸识别SDK后移
   * 开始定位（显示定位中），定位可以缓存，先加载，定位同步进行，如果缓存的地理位置不一样，然后才更新
   * 配置下发等

- 多线程启动
> 尝试使用多线程启动
将数据库的配置、SDwebimage UA的配置放到子线程中初始化，引入状态管理，需要监控子线程任务状态，判断是否取消假的开屏页面。

广告同步加载，2秒回来了就展示，不回来就过
[Timer Profile的使用]([https://www.dazhuanlan.com/2019/09/28/5d8f40b3545f3/](https://www.dazhuanlan.com/2019/09/28/5d8f40b3545f3/)
)
