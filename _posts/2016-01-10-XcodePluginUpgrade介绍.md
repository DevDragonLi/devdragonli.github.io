---
layout: post
title: 2016-01-10-XcodePluginUpgrade介绍
date: 2016-01-10
categories: blog
tags: [iOSDev]
description: 2016-01-10-XcodePluginUpgrade介绍(针对自己写的Xcode插件兼容辅助工具的介绍)

---

## [XcodePluginUpgrade地址](https://github.com/DevDragonLi/XcodePluginUpgrade-LFL/)


##Why:

- 每当Xcode升级之后，都会导致原有的Xcode插件不能使用，这是因为每个插件的Info.plist中记录了该插件兼容Xcode版本的DVTPlugInCompatibilityUUID，而每个版本的Xcode的DVTPlugInCompatibilityUUID都是不同的。如果想让原来的插件继续工作，我们就得将新版Xcode的DVTPlugInCompatibilityUUID加入到每一个插件的Info文件中，手动添加的话比较费时间还可能出错



## adjust
- 如果电脑的用户名曾经更改而且没有更改个人目录那么获取的路径无效,修正后为不获取用户名,而是获取用户下个人目录方式



## 代码实现: 加了注释

```

        // 1.1 获取当前电脑Users下所有文件名
        NSMutableArray *users = [NSMutableArray arrayWithCapacity:1];
        NSFileManager *fileManager = [NSFileManager defaultManager];
        NSString *path = @"/Users";
        NSArray *tempArray = [fileManager contentsOfDirectoryAtPath:path error:nil];
        
        for (NSString* fileName in tempArray) {
            BOOL flag = YES;
            NSString* fullPath = [path stringByAppendingPathComponent:fileName];
            if ([fileManager fileExistsAtPath:fullPath isDirectory:&flag]) {
                if (!flag) {
                    [users addObject:fullPath];
                }
            }
        }
        //1.2 遍历 得到 目录名
        NSString *userCatalogueNameReally = [[NSString alloc]init];
        for (NSString *userCatalogueName in tempArray) {
            
            if ([userCatalogueName isEqualToString:@".localized"] ||
                [userCatalogueName isEqualToString:@"Deleted Users"]||
                [userCatalogueName isEqualToString:@"Guest"]||
                [userCatalogueName isEqualToString:@"Shared"]) {
            }else {
                userCatalogueNameReally = userCatalogueName;
                LFLog(@"当前用户%@",userCatalogueNameReally);
            }
        }
        
        //2.1 -------------- 获取 Xcode 插件路径名---------------///
        NSString *pluginPath = [NSString stringWithFormat:@"/Users/%@/Library/Application Support/Developer/Shared/Xcode/Plug-ins", userCatalogueNameReally];
        
        //2.2.1 加载本地的Info.plist.
        NSDictionary *xcodeInfoDictionary = [[NSDictionary alloc] initWithContentsOfFile:@"/Applications/Xcode.app/Contents/Info.plist"];
        
        //2.2.2 获取 Xcode  UUID
        NSString *xcodeUUID = xcodeInfoDictionary[@"DVTPlugInCompatibilityUUID"];
        NSError *error;
        NSArray *pathArray = [fileManager contentsOfDirectoryAtPath:pluginPath error:&error];
//        3. 遍历处理插件的UUID
        if (!error) {
            for (NSString *xcpluginFileName  in pathArray) {
                if ([xcpluginFileName hasSuffix:@".xcplugin"]) {
                    LFLog(@"你电脑之前安装了%@这个插件",[xcpluginFileName componentsSeparatedByString:@".xcplugin"].firstObject);
                    NSString *pluginPlistPath = [NSString stringWithFormat:@"%@/%@/Contents/Info.plist", pluginPath, xcpluginFileName];
                    NSMutableDictionary *pluginInfoDictionary = [[NSMutableDictionary alloc] initWithContentsOfFile:pluginPlistPath];
                    NSMutableArray *supportedUUIDs = [NSMutableArray arrayWithArray:pluginInfoDictionary[@"DVTPlugInCompatibilityUUIDs"]];
                    
                    if (![supportedUUIDs containsObject:xcodeUUID]) {
                        [supportedUUIDs addObject:xcodeUUID];
                        [pluginInfoDictionary setValue:supportedUUIDs forKey:@"DVTPlugInCompatibilityUUIDs"];
                        [pluginInfoDictionary writeToFile:pluginPlistPath atomically:YES];
                    }
                }
            }
            LFLog(@"Processed, please restart xcode");
        }
        else{
            LFLog(@"Wrong ,Please try again!");
        }
        return 0;
    }

```

