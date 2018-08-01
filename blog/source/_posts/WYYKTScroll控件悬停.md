---
banner: http://blog.zhangruipeng.me/hexo-theme-icarus/gallery/math.jpg
title: WYYKTScroll控件悬停
date: 2017-10-12 15:25:30
tags: UI
thumbnail: http://blog.zhangruipeng.me/hexo-theme-icarus/gallery/math.jpg
categories:
- iOS开发
---
#### 效果示例
![WYDemo2.gif](http://upload-images.jianshu.io/upload_images/2149459-53f2c26e6c7f08d7.gif?imageMogr2/auto-orient/strip)
### 这是一个控件悬停的UI效果实现，类似于网易云课堂的详情页UI效果

<!--more-->

#### 1.工程引入FMBaseViewController, 并添加要自定义的controller

* 有关头部image及button的相关设置通过，FMBaseViewController的属性进行设置，示例代码如下：
````
FMBaseViewController *bvc = [[FMBaseViewController alloc] init];
bvc.btnBackColor = [UIColor cyanColor];
bvc.btnTitleArr = @[@"张三", @"李四", @"王五"];
bvc.indicatorColor = [UIColor yellowColor];
bvc.isIndicatorHidden = YES;
bvc.headImage_H = 100;
bvc.button_H = 30;
bvc.headImageName = @"picture_3";
bvc.isStretch = NO;
````

#### 2.注意：自定义的controller 必须继承于FMParentViewController.h, 并且子控制器暂时只支持UITableViewController
* 子控制器类型1 ：FMTableViewStylePlain 初始化代码如下：
````
FMT2ViewController *t2 = [[FMT2ViewController alloc] initWithTableViewStyle:FMTableViewStylePlain];
或者（default）
FMT1ViewController *t1 = [[FMT1ViewController alloc] init];
````
* 子控制器类型2：FMTableViewStyleGroup 初始化代码如下：
````
FMT2ViewController *t2 = [[FMT2ViewController alloc] initWithTableViewStyle:FMTableViewStyleGrouped];
或者（用属性修改）
FMT1ViewController *t2 = [[FMT1ViewController alloc] init];
t2.tableViewStyle = FMTableViewStyleGrouped;
````
#### 3.头部视图是否可以拉伸：

 ````
isStretch 属性（default is YES）
 ````
* 测试效果查看，在AppDelegate.m 的launch函数中添加（或替换）如下代码：
````
FMBaseViewController *bvc = [[FMBaseViewController alloc] init];
    self.window.rootViewController = bvc;
    [self.window makeKeyAndVisible];
````
* 自定义子controller初始化后传入该数组childVCArr，示例代码如下：
````
    FMBaseViewController *bvc = [[FMBaseViewController alloc] init];
    FMT1ViewController *t1 = [[FMT1ViewController alloc] init];
    FMT2ViewController *t2 = [[FMT2ViewController alloc] initWithTableViewStyle:FMTableViewStyleGrouped];
    FMT3ViewController *t3= [[FMT3ViewController alloc] init];
    bvc.childVCArr = @[t1, t2, t3];
````
####子控制器最好不要超过5个， 暂不支持滑动（以后可能添加，敬请期待！）
* headView上的内容可自定义添加，通过 ftc.headView可拿到head部分的视图添加自己的控件。
* 支持cocoaPods 安装 
````
pod search WYTest
在Podfile中添加
pod "WYTest"
pod install || pod update
````
###2018.08.01更新
####增加对UICollectionView的支持，controller必须继承自FMBaseCollectionViewController

* 示例代码

````
    FMT1ViewController *t1 = [[FMT1ViewController alloc] init];
    t1.tableViewStyle = FMTableViewStyleGrouped;
    FMC1ViewController *c1 = [[FMC1ViewController alloc] init];
    FMT3ViewController *t3= [[FMT3ViewController alloc] init];
````

#### 效果示例 - 全collectionView
![collectionSupport1.gif](https://upload-images.jianshu.io/upload_images/2149459-81372f034ffad1fa.gif?imageMogr2/auto-orient/strip)

#### 效果示例 - collectionView 和 tableView混合
![collectionSupport.gif](https://upload-images.jianshu.io/upload_images/2149459-5e3a134d074774c1.gif?imageMogr2/auto-orient/strip)

##github Demo下载地址
[点击去下载](https://github.com/OSzhou/WYYKTScroll.git)
