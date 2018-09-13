---
layout: post
title: 'cocoapods私有库'
date: 2018-08-03
comments: true
categories: cocoapods
tags: [模块化]
published: ture
keywords: cocoapods
description: cocoapods私有库以及framework的打包
---

## 1. 添加私有仓储

```ruby
pod repo add CYSpecs https://git.coding.net/SeongBrave/CYSpecs.git
```

## 2.模块开发

### 1. 模块初始化

```ruby
 pod lib create UtilCore
```

#### 1.1 只创建一个 podspecs

```ruby
pod spec create UtilCore_Lib.podspec
```

### 2. 模块验证

- 本地模块验证

```ruby
 pod lib lint --allow-warnings --sources='https://git.coding.net/SeongBrave/CYSpecs.git,https://github.com/CocoaPods/Specs'
```

- 确定没问题了可以先打 tag 验证:

```ruby
pod spec lint --sources='https://git.coding.net/SeongBrave/CYSpecs.git,https://github.com/CocoaPods/Specs'
```

### 3. 提交模块到私有仓储

```ruby
pod repo push CYSpecs UtilCore.podspec --sources='https://git.coding.net/SeongBrave/CYSpecs.git,https://github.com/CocoaPods/Specs'
```

#### 3.1 或者提交到公共仓储

3.1.1 首先需要注册

```shell
pod trunk register seongbrave@sina.com
```

3.1.2 提交注册到公共仓储

```shell
pod trunk push UtilCore_Lib.podspec --allow-warnings
```

### 3. 打包成 Framework 或者 Library

> 可以使用 pod package 命令打包成 Framework 或者 Library ，默认是头文件都会打包，如果需要指定公开的头文件可以在 `UtilCore.podspec`文件中配置`s.public_header_files`

### 1. 默认打包成 Framework

```ruby
pod package UtilCore.podspec  --force
```

### 2. 加上`--library`即可打包成.a 文件

```ruby
pod package UtilCore.podspec  --force --library
```

### 3. 指定私有仓储

> 如果项目中使用了私有库，需要指定对应的私有仓储,需要加上 `--spec-sources`指定私有仓储

```ruby
 pod package UtilCore.podspec  --force  --spec-sources=https://git.coding.net/SeongBrave/CYSpecs.git,https://github.com/CocoaPods/Specs.git
```

#### 3.1.私有仓储依赖 **Framework**

在`UtilCore.podspec`文件中添加一下类似代码即可依赖 Framework

```ruby
  s.vendored_frameworks = ['UtilCore/vendored_frameworks/UtilCore.framework']
  spec.resource_bundles = {'Resources' => 'UtilCore/vendored_frameworks/UtilCore.framework/Resources/UtilCore.bundle'}
```

### 4. 打包完后使用注意

#### 1. 提供给外部人员使用 Framework

> 可以将打包成的 Framework 放大**JBox**上(理论应该可以，但是我测试了貌似不行不过我将 Framework 压缩然后放到骑牛上就完全没有问题),以下以设备指纹项目`UtilCore`为例说明:

- 第一步取得上面生成打包好的 **UtilCore.Framework** 将其放到桌面或者其它地方已经创建好的`UtilCore`文件夹中,文件夹中应该包含的内容包括

  - 最终要的 **UtilCore.Framework** 文件库
  - 一定要包含的 **Doc** 文件，帮助用户使用的
  - 还要包含的就是版本更新记录`UtilCore_Sdk_ReleaseNotes`
  - 当然最好再包含一份 **Demo**项目就完美了

  参照友盟的如:

  ![](/images/20180803/001.png)

- 第二步 然后压缩文件夹 为`UtilCore.zip`包，并在直接上传到骑牛空间，最终得到一个地址:`http://o82jar85w.bkt.clouddn.com/UtilCore.zip`

- 第三步 生成对应外部人员使用的`UtilCore.podspec`文件
  `UtilCore.podspec`文件上传到提供静态服务的网站即可，我是在本地测试通过的

```ruby
Pod::Spec.new do |s|
  s.name             = 'UtilCore'
  s.version          = '0.0.3'
  s.summary          = '京东金融基础工具库'

  s.description      = <<-DESC
TODO:包含以下简单的工具封装，JSON的解析等基础库.
                       DESC

  s.homepage         = 'http://jci.cbpmgt.com/repo/repoDetailShow/4316/git'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'seongbrave' => 'seongbrave@sina.com' }
  s.source              = { :http => "http://o82jar85w.bkt.clouddn.com/UtilCore.zip" }
  s.social_media_url = 'http://seongbrave.github.io/'
  s.vendored_frameworks = "*/UtilCore.framework"
  s.framework           = "CoreTelephony"
  s.xcconfig            = { "LIBRARY_SEARCH_PATHS" => "\"$(PODS_ROOT)/UtilCore/**\"" }
end
```

- 第四步 具体 demo 的`pofile`文件中的使用

```ruby
pod 'UtilCore', :podspec => "http://10.13.100.75:8887/UtilCore.podspec"
```

最后说下 上面的代码中的 `s.name = 'UtilCore'`这个 name 很关键 ，这块的 name 直接决定了使用的时候在 `podfile`文件中引用第三方项目的名称的

#### 2. 私有库中包含了其它静态库或者私有的 Framework

> 如果私有库中包含了其它静态库或者私有 Framework，则要使用`pod package` 就会存在问题

> > [参考资料](https://www.jianshu.com/p/9096a2eb2804),以下是解决办法

##### 1. 第一步:打包命令添加参数 --no-mangle

> 这时候如果拿去运行，则会报错，因为我们依赖的静态库`没有`被打包到`Framework`中，这个文件是我们的静态库依赖的另一个静态库，需要你手动添加进项目，所以再把依赖的这个静态库 拖进项目就好了，运行成功

```ruby
pod package CYSpecs.podspec  --force  --spec-sources=https://git.coding.net/SeongBrave/CYSpecs.git,https://github.com/CocoaPods/Specs.git --no-mangle
```

上面的命令中，有一个是--no-mangle，表示 Do not mangle symbols of depedendant Pods，当你的项目依赖包含 **静态库**时

##### 2. 第二步:作为 cocoapods 的库输出`Framework`和其它静态库

> 其实也不需要这一步了，因为已经打包成了目标`Framework`了，告诉使用者直接使用即可，但是有两条原因:

- 作为 cocoapods 库的形式提供，方便导入其它依赖项目，使用者无需手动导入

- 如果想将该`Framework`作为其它模块使用的时候作为依赖

> 参照 [libWeChatSDK](https://github.com/zfan67/libWeChatSDK) 专门为静态库创建一个 仓储即可解决以上问题

```ruby
#libWeChatSDK.podspec
Pod::Spec.new do |s|
s.name         = "libWeChatSDK(1.6.2)"
s.version      = "1.6.2"
s.summary      = "微信官方SDK 1.6.2版本."

s.homepage     = "https://open.weixin.qq.com/cgi-bin/index?t=home/index&lang=zh_CN"
s.license      = 'MIT'
s.platform     = :ios, "6.0"
s.ios.deployment_target = "6.0"
s.source       = { :git => "https://github.com/zfan67/libWeChatSDK.git", :tag => s.version}
s.source_files  = 'zfan67/libWeChatSDK/*.{h,m,a}'
s.requires_arc = true
end
```

## 3. 项目使用

### podfile

首先导入私有仓储

```ruby
source 'https://github.com/CocoaPods/Specs.git'  # 官方库
source 'https://git.coding.net/SeongBrave/CYSpecs.git'   # 私有库
```

然后引入私有库即可

```ruby
pod 'UtilCore', '0.0.7'
```

#### 如果处于开发阶段可以指定模块仓储地址并且指定分子

```ruby
 pod 'UtilCore', :git => 'https://github.com/SeongBrave/UtilCore.git', :branch => 'dev'
```

## 4. 遇到的问题

### 1.当搜索不到私有库的时候可以尝试 :

```
rm ~/Library/Caches/CocoaPods/search_index.json
```

### 2. 可以不用打新标签更新库，具体操作为:

```
rm -rf "${HOME}/Library/Caches/CocoaPods"
rm -rf "`pwd`/Pods/"
pod update
```

### 3. 更新 pod 源

```
pod repo update
```

### 4. 如果需要 swif4 则需要在终端中执行

```
echo 4.0 > .swift-version
```

## 5. git 相关操作

### 1. 删除远程分支和 tag:

```
git push origin --delete tag  0.0.1
```

### 2. 删除本地分子:

```
git tag -d 0.0.1
```

### 3. 设置本地记录用户名和密码

```
git config --global credential.helper osxkeychain
```

## 5. cocoapods 插件

### 1. cocoapods-playground

> 可以在 playground 中方便测试第三方项目

#### 1. 下载

```
sudo gem install cocoapods -n /usr/local/bin
```

#### 2. 使用

```
pod playgrounds SnapKit
```

### 2. cocoapods-package

#### 1. 头文件导入问题

> [参考资料](https://www.jianshu.com/p/544df88b6a1e) ![](/images/20180803/002.png)

## 参考资料

1.  [创建私有库](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)
2.  [私有库引用私有库](http://www.cnblogs.com/tufeibo/p/5654268.html)
3.  [私有库管理和模块化管理](http://www.pluto-y.com/cocoapod-private-pods-and-module-manager/)
4.  [pod 导入静态资源](http://blog.startry.com/2016/03/17/the-trap-of-image-resource/)
5.  [官方资料](https://guides.cocoapods.org/syntax/podspec.html)
6.  [cocoaPods 进行 SDK 二次包装(cocoapods-packager 完成 framework 静态库打包,避免第三方库冲突)](https://blog.csdn.net/iOSTianNan/article/details/81007691)
7.  [CocoaPods 动/静态库混用封装组件化](https://www.jianshu.com/p/544df88b6a1e) 推荐:<font color=red>**\*\*\***</font>
