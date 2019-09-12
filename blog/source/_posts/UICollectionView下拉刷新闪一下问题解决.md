---
banner: https://upload-images.jianshu.io/upload_images/2149459-4562b897de66abe7.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1024/format/webp
title: UICollectionView下拉刷新闪一下问题解决
date: 2017-10-12 15:31:08
tags: UI
thumbnail: https://upload-images.jianshu.io/upload_images/2149459-4562b897de66abe7.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1024/format/webp
categories:
- iOS开发
---
最近写项目遇到UICollectionView的下拉刷新数据回来时，屏幕会闪一下，在网上找了几个方法，亲自试了都不好使，后来自己试了下回主线程刷新UI，发现可以，代码如下:
````
 dispatch_async(dispatch_get_main_queue(), ^{
                [self.collectionView reloadData];
            });
````
大家都知道，在子线程刷新UI是很危险的，有时会出现莫名的Bug, 有些情况甚至会直接崩溃。而网络数据的请求一般都是在子线程进行的，数据回来时去刷新UI（UITableView 和 UIConllectionView）但是UITableView在子线程刷新UI没有出现这个Bug（估计苹果做了优化）。
