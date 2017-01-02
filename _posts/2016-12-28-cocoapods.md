---
layout: post
title: cocoapods和使用本地或者公司内部库
date: 2016-12-28
categories: blog
tags: [iOSDev]
description: 2016-12-28-cocoapods

---
# cocoapods 管理第三方工具总结

##Podfile本质上是用来描述Xcode工程中的targets用如果我们不显式指定Podfile对应的target，CocoaPods会创建一个名称为default的隐式target，会和我们工程中的第一个target相对应。换句话说，如果在Podfile中没有指定target，那么只有工程里的第一个target能够使用Podfile中描述的Pods依赖库

## 1. pod update --no-repo-update 
## 1.1 pod update  只要不是写死的,就更新
Podfile中指定的依赖库版本不是写死的，当对应的依赖库有了更新，无论有没有Podfile.lock文件都会去获取Podfile文件描述的允许获取到的最新依赖库版本

## 2.搜索框架
   		- 空格 下一页
		- q 退出
		- / 搜索
## 2.1只搜索符合名字的框架   		 pod search AFNetworking --simple


## 3.如果有Swift --------> use_frameworks!
## 3.1 inhibit_all_warnings! 屏蔽库的警告 
# 注意:cocoapod 1.0 版本以上一定要有 target 
target 'DemoProject' do
pod 'AFNetworking', '~> 3.0.4'  :inhibit_all_warnings =>true
end

## 4. 使用Podfile管理Pods依赖库版本说明
pod 'AFNetworking'      //不显式指定依赖库版本，表示每次都获取最新版本  
pod 'AFNetworking', '2.0'     //只使用2.0版本  
pod 'AFNetworking', '> 2.0'     //使用高于2.0的版本  
pod 'AFNetworking', '>= 2.0'     //使用大于或等于2.0的版本  
pod 'AFNetworking', '< 2.0'     //使用小于2.0的版本  
pod 'AFNetworking', '<= 2.0'     //使用小于或等于2.0的版本  
pod 'AFNetworking', '~> 0.1.2'     //使用大于等于0.1.2但小于0.2的版本  
pod 'AFNetworking', '~>0.1'     //使用大于等于0.1但小于1.0的版本  
pod 'AFNetworking', '~>0'     //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本

# **************使用本地或者公司内部库********

## 1.指定specs的位置,自定义添加自己的podspec。公司内部使用cocoapods 官方source是隐式的需要的，一旦你指定了其他source 你就需要也把官方的指定上

source 'ssh://git@gitlab.LFL.com:9830/iOS/Specs.git'
source 'https://github.com/CocoaPods/Specs.git'

### 当我们使用pod install或者pod setup时，会自动在~/.cocoapods/repo master 同目录 出现coding***podspec 文件夹.保存对应的pod
         
## 2.在你的本地pod库添加xxx.podspec文件，（一定要注意是根目录添加）LFLKu 目录下

pod 'LFLKu', :path => '/Users/LFL/LFLKu

对应的podsepc s.homePage 改成你的 本地目录

## 3.From a podspec in the root of a library repository (引用仓库根目录的podspec)
使用仓库中的master分支:
pod 'DevDragonLi', :git => 'https://github.com/DevDragonLi.git'

使用仓库的其他分支:
pod 'DevDragonLi', :git => 'https://github.com/DevDragonLi.git' :branch => 'release'

使用仓库的某个tag:
pod 'DevDragonLi', :git => 'https://github.com/DevDragonLi.git', :tag => '0.0.1'

或者指定一个提交记录:
pod 'DevDragonLi', :git => 'https://github.com/DevDragonLi.git', :commit => '5e473f1e0530bb3799f2f0d70554b292570bd8f0'

## 4.From a podspec outside a spec repository, for a library without podspec（在一个不带podsepec的库里引用外部的spec）
pod 'JSONKit', :podspec => 'https://example.com/JSONKit.podspec'


### 4.1  ~/Library/Caches/CocoaPods/ 缓存文件目录


