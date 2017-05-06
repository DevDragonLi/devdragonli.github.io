---
layout: post
title: IJKPlayer视频直播-编译和使用
date: 2017-05-06
categories: blog
tags: [iOSDev]
description: IJKPlayer视频直播-编译和使用

---

##前言
- IJKPlayer 由Bilibili开发并开源的框架[源码GitHub地址](https://github.com/Bilibili/ijkplayer)
- IJKPlayer 是一个基于 ffplay 的轻量级 Android/iOS 视频播放器。API 易于集成；编译配置可裁剪，方便控制安装包大小；支持 硬件加速解码，更加省电。
- 我编译后的地址,可以直接使用[IJKMediaDemo](https://coding.net/u/LFL/p/IJKMediaDemo/git)

## 下载后编译的准备工作
- 下载部分 (建议翻墙,下载ffmpeg后ijkplayer-ios文件夹大小超过一个G)

```
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
cd ijkplayer-ios
./init-ios.sh   //://下载ffmpeg和相关脚本
cd ios  // :选择其中的ios文件夹再操作
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all
编译期间,电脑发热....

```
- Link Binary with Libraries

```
# 		  AudioToolbox.framework
#         AVFoundation.framework
#         CoreGraphics.framework
#         CoreMedia.framework
#         CoreVideo.framework
#         libbz2.tbd
#         libz.tbd
#         MediaPlayer.framework
#         MobileCoreServices.framework
#         OpenGLES.framework
#         QuartzCore.framework
#         UIKit.framework
#         VideoToolbox.framework

```

## 运行Demo
- 我们在Online Samples随意选择一个测试ijkplayer是否运行正常。老版本电视维修画面既视感

## 制作framework
- 首先我们打开IJKMediaPlayer.xcodeproj,点击IJKMediaFramework出现选择框，选择edit scheme,将build configuration改为Release后点Close
- 分别在模拟器和真机(Generic iOS Device也可以)上编译(com+b)
- 通过lipo的命令将两者合并输出，在Products目录下输入

```
lipo -create /Users/DragonLi/Library/Developer/Xcode/DerivedData/IJKMediaPlayer-bpgcmxgrotghheguxjsvyrrebtyl/Build/Products/Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework /Users/DragonLi/Library/Developer/Xcode/DerivedData/IJKMediaPlayer-bpgcmxgrotghheguxjsvyrrebtyl/Build/Products/Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework -output /Users/DragonLi/Library/Developer/Xcode/DerivedData/IJKMediaPlayer-bpgcmxgrotghheguxjsvyrrebtyl/Build/Products/IJKMediaFramework
```
- lipo -info IJKMediaFramework查看framework的支持

##cocoapods集成方式介绍
- 可以自己打包后制作podspec文件,到国内第三方平台,podspec可以参考我之前的一个第三方[LFLSegmentControl](https://github.com/DevDragonLi/LFLSegmentControl)

## ijkplayer基本使用方式,默认不添加依赖可以直接编译的
- 核心集成代码

```
IJKFFOptions *options = [IJKFFOptions optionsByDefault]; //使用默认配置
self.player = [[IJKFFMoviePlayerController alloc] initWithContentURL:self.url withOptions:options]; //初始化播放器，播放在线视频或直播(RTMP)
// NSString *filePath = [[NSBundle mainBundle] pathForResource:@"init"ofType:@"mp4"];
// self.player = [[IJKFFMoviePlayerController alloc]initWithContentURLString:filePath withOptions:options]; //初始化播放器，播放本地视频
self.player.view.autoresizingMask = UIViewAutoresizingFlexibleWidth|UIViewAutoresizingFlexibleHeight;
self.player.view.frame = self.view.bounds;
self.player.scalingMode = IJKMPMovieScalingModeAspectFit; //缩放模式
self.player.shouldAutoplay = YES; //开启自动播放

self.view.autoresizesSubviews = YES;
[self.view addSubview:self.player.view];

//准备
[self.player prepareToPlay];
//播放
[self.player play];
//暂停
[self.player pause];
//销毁
[self.player shutdown];
```




