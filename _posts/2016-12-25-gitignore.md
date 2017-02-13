---
layout: post
title: gitignore配置 与 隐藏文件设置
date: 2016-12-25
categories: blog
tags: [iOSDev]
description: 2016-12-25-gitignore配置 与 隐藏文件设置

---
# gitignore配置 与 隐藏文件设置

# 添加.gitignore文件的步骤操作最好在git init步骤之后，也就是创建初始版本库之后就在工做根目录（也就是与.git同一层及目录）下添加.gitignore文件，然后再用Xcode新建一个项目
## 1.gitignore文件的获取与配置
可以在 GitHub 官网上搜索 gitignore , 有项目维护着各种语言开发所使用的.gitingore文件
## 2.gitignore 文件  解释
/#/ 	此为注释	表示注释, 将被忽略
 	 
*.a				* 代表所有, 即忽略所有 .a结尾的文件

/todo	仅仅忽略 RootSRC(项目根目录)/todo, 即忽略指定的文件或文件夹(不包括子目录)

todo	忽略 todo/ 目录下的所有文件, 包括子目录

doc/*.txt	会忽略 doc/ 目录内的所有.txt文件, 但不包括子目录的


