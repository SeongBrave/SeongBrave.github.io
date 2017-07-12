---
layout: post
title: "Cocoapods私有库"
date: 2017-07-12
comments: true
categories: Cocoapods
tags: [Cocoapods,组件化,模块化]
published: ture
keywords: Cocoapods,私有库
description: Cocoapods创建私有库
---

> 该文章主要简单记录我在模块化过程中使用Cocoapods的过程中遇到的一些问题已经一些简单技巧吧

#### 1.添加私有仓储:
```
 pod repo add CYSpecs https://git.coding.net/SeongBrave/CYSpecs.git
```
#### 2.模块开发:
1.  模块初始化:
```
 pod lib create ModelProtocol
```
2. 模块验证：
 ```
 pod lib lint --verbose
 ```
 3. 提交模块到私有仓储:
 ```
 pod repo push CYSpecs ModelProtocol.podspec --verbose
 ```
#### 3.之后就可以更新模块，更新需要注意:

```
update code, 然后打tag ，推送tag ，记得修改ModelProtocol.podspec文件对于的tag 版本
```
#### 4.私有库中引用私有库

> [参考资料](https://aotu.io/notes/2016/01/27/how-to-make-cocoapods/)

1.   模块验证:

-  本地调试验证模块
```
pod lib lint --allow-warnings --sources='https://git.coding.net/SeongBrave/CYSpecs.git,https://github.com/CocoaPods/Specs'
```
- 确定没问题了可以先打tag 验证:
```
pod spec lint --sources='https://git.coding.net/SeongBrave/CYSpecs.git,https://github.com/CocoaPods/Specs'
```
2. 提交模块到私有仓储

```
pod repo push CYSpecs NetWorkCore.podspec --sources='https://git.coding.net/SeongBrave/CYSpecs.git,https://github.com/CocoaPods/Specs'
```
#### 5. 遇到的问题
1. 当搜索不到私有库的时候可以尝试 :
```
rm ~/Library/Caches/CocoaPods/search_index.json
```
2.可以不用打新标签更新库，具体操作为:
```
rm -rf "${HOME}/Library/Caches/CocoaPods"
rm -rf "`pwd`/Pods/"
pod update
```
#### 6.git 相关操作
1. 删除远程分支和tag:
-  删除远程仓储:
```
git push origin --delete tag  0.0.1
```
-  删除本地仓储:
```
git tag -d 0.0.1
```
2. 设置本地记录用户名和密码
 
```
 git config --global credential.helper osxkeychain
```


#### 7.参考资料
1.  [创建私有库](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)
2.  [私有库引用私有库](http://www.cnblogs.com/tufeibo/p/5654268.html)
3.  [私有库管理和模块化管理](http://www.pluto-y.com/cocoapod-private-pods-and-module-manager/)
4.  [pod 导入静态资源](http://blog.startry.com/2016/03/17/the-trap-of-image-resource/)
