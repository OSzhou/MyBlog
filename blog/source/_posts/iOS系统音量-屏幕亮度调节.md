---
banner: http://blog.zhangruipeng.me/hexo-theme-icarus/gallery/shoes.jpg
title: iOS系统音量&屏幕亮度调节
date: 2017-10-12 15:35:14
tags:
thumbnail: http://blog.zhangruipeng.me/hexo-theme-icarus/gallery/shoes.jpg
---
##### 一，系统音量获取
系统框架
````
#import <AVFoundation/AVFoundation.h>
#import <MediaPlayer/MediaPlayer.h>
#import <AVKit/AVKit.h>
````
获取系统音量slider
````
- (MPVolumeView *)volumeView {
    if (_volumeView == nil) {
        _volumeView  = [[MPVolumeView alloc] init];
        [_volumeView sizeToFit];
#warning 获取系统的音量的UISlider
        for (UIView *view in [_volumeView subviews]){
            if ([view.class.description isEqualToString:@"MPVolumeSlider"]){
                self.volumeViewSlider = (UISlider*)view;
                break;
            }
        }
    }
    return _volumeView;
}
````

<!--more-->

监听系统物理按键调节音量
````
/** 监听 */
- (void)registerVolumeChangeEvent {
//    NSError *error;
//    [[AVAudioSession sharedInstance] setActive:YES error:&error];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(changeVolumeValueFunc) name:@"AVSystemController_SystemVolumeDidChangeNotification" object:nil];
}
/** 移除 */
- (void)unregisterVolumeChangeEvent {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:@"AVSystemController_SystemVolumeDidChangeNotification" object:nil];
}
````
获取系统当前音量
````
[[AVAudioSession sharedInstance] outputVolume];
````

##### 二，屏幕亮度
很简单就一句

````
[[UIScreen mainScreen] setBrightness:值（0 ~ 1）]
//示例
            if (panPoint.y < 0) {
                //增加亮度
                [[UIScreen mainScreen] setBrightness:self.startVB + (-panPoint.y / 30.0 / 10)];
            } else {
                //减少亮度
                [[UIScreen mainScreen] setBrightness:self.startVB - (panPoint.y / 30.0 / 10)];
            }
````
