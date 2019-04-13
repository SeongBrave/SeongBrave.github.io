---
layout: post
title: 'Swift + RxSwift MVVM 模块化项目实践'
date: 2019-04-13
comments: true
categories: Swift
tags: [RxSwift]
published: ture
keywords: RxSwift
description: Swift + RxSwift MVVM 模块化项目实践
---

![banner.png](/images/20190413/banner.png)

本文主要介绍个人在 Swift 项目开发中的一些实践经验，供大家所借鉴或者探讨。

提高开发效率，降低 Bug 发生率，是我们每个开发所追随的目标。个人认为通过 CocoaPods 实现模块化组件化，积累适合的组件模块，重复利用公用模块，不仅可以提高开发效率并且可以有效的降低 Bug 的发生，另外可以借助 [Gckit-CLI](https://github.com/SeongBrave/gckit) 等脚本工具降低重复无用的代码编写，进一步提高开发效率，降低低级错误的发生，本文以下内容主要讲解个人通过 CocoaPods 结合 [Gckit-CLI](https://github.com/SeongBrave/gckit) 实现开发效率的最大化的一些项目实践

## 项目介绍

> Twilight，项目取自暮光之城电影名
> 所有的资源都已经开源道 Github 上了，包括服务端的接口项目

Demo 效果演示

![showapp.gif](/images/20190413/showapp.gif)

## 模块功能介绍

### 业务模块

| 模块            | 介绍             | 地址                                            |
|---------------|----------------|-----------------------------------------------|
| Carlisle      | 登陆注册模块         | `https://github.com/SeongBrave/Carlisle.git`  |
| Bella         | 上下文模块          | `https://github.com/SeongBrave/Bella.git`     |
| Alice         | 项目配置模块         | `https://github.com/SeongBrave/Alice.git`     |
| Jacob         | 首页模块           | `https://github.com/SeongBrave/Jacob`         |
| Twilight      | 主工程项目          | `https://github.com/SeongBrave/Twilight.git`  |
| TwilightSpecs | CocoaPods 私有仓储 | `https://github.com/SeongBrave/TwilightSpecs` |

登陆注册模块([Carlisle](https://github.com/SeongBrave/Carlisle.git))

> 包含用户注册、登陆、找回密码等功能，主要是用户权限相关的管理界面，登陆注册模块是参考[RxSwift](https://github.com/ReactiveX/RxSwift)官方 Demo 简单修改完成的。

上下文模块([Bella](https://github.com/SeongBrave/Bella.git))

> 上下文模块主要用于用户对象的管理，后期会把考虑把本地缓存等加密功能加上，上下文模块被每个业务模块所依赖，用于管理用户上下文对象，同步用户信息的修改。

项目配置模块([Alice](https://github.com/SeongBrave/Alice.git))

> 包括项目的主题等各个模块的配置，涉及所有业务模块的主题颜色配置，以及一些第三方库的 key，各个模块的通知等。

首页模块([Jacob](https://github.com/SeongBrave/Jacob))

> 商品列表模块 取值暮光之城中 -Jacob

该模块 **90%** 的代码是通过[Gckit-CLI](https://github.com/SeongBrave/gckit)生成的，一键生成包含了大部分的逻辑代码，
上拉加载更多、下拉刷新、错误提示、出错重试处理等逻辑，这些大部分的逻辑代码是不需要修改的。

目录结构:

```bash
├── Api
│   ├── Home_api.swift
│   └── Product_api.swift
├── Model
│   ├── Home_model.swift
│   └── Product_model.swift
├── Module
│   ├── JacobCore.swift
│   └── Jacob_router.swift
├── View
│   └── tCell
│       ├── Home_tCell.swift
│       └── Product_tCell.swift
├── ViewController
│   ├── Home_vc.swift
│   └── Product_vc.swift
└── ViewModel
    ├── Home_vm.swift
    └── Product_vm.swift
```

目录结构分为:

- Api: 接口 Api
- Model: 实例 Model
- Module: 模块相关管理类，包含路由注册和提供别的模块访问的管理类
- View: 相关自定义的 View
- ViewController: 对应的 ViewController
- ViewModel: 对应的 ViewModel

```swift
  /// 界面第一次初始化
 let _ =  Observable.of(
     input.firstLoadTriger,
     reloadTrigger.withLatestFrom(input.firstLoadTriger))
     .merge().map{ Home_api.homes(page: 0, pageSize: 10)}.share(replay: 1)
     .emeRequestApiForArray(Home_model.self,activityIndicator: loading)
     .subscribe(onNext: {[unowned self] (result) in
         switch result {
         case .success(let data):
             self.hasNextPage.value = data.count == 10
             self.homeElements.value = data
             self.page = 1
         case .failure(let error):
             self.refresherror.onNext(error)
         }
     })
     .disposed(by: disposeBag)
```

上面的代码 通过信号筛选，`reloadTrigger`代表点击重新加载的事件，经过参数格式化、发送网络请求、数据解析等数据处理，最后只需关注解析成功之后的 Model 数据然后更新 UI 界面。

## RxSwift 的使用

项目中大部分的逻辑处理是借助 RxSwift 实现的响应式编程，当界面上的每个操作都会转换为一个信号然后通过对信号的各种加工网络请求，到返回的数据 JSON 解析以及错误对象的处理，感觉整个开发都是在开凿水渠，等开发完了就不用管了。

## 网络请求

[NetWorkCore](https://github.com/SeongBrave/NetWorkCore)通过对[Alamofire](https://github.com/Alamofire/Alamofire)简单封装，配合[RxSwift](https://github.com/ReactiveX/RxSwift)可以很简单的实现一个网络请求，并且完成数据解析对应的 Mode 实体类，如下所示，即可实现一个用户登录的网络请求。

```swift
 input.loginTaps
            .withLatestFrom(Observable.combineLatest(input.username, input.password) { ($0, $1) })
            .map{Carlisle_api.login(phone: $0, password: $1)}
            .emeRequestApiForObj(User_Model.self, activityIndicator: loading)
            .subscribe(onNext: {[unowned self] (result) in
                switch result {
                case .success(let user):
                    //登陆成功就更新上下文中的登陆对象
                    Global.updateUserModel(user)
                    self.loginSuccess.onNext(user)
                case .failure(let error):
                    self.error.onNext(error)
                }
            })
            .disposed(by: disposeBag)
```

## 错误处理

监控整个 App 的所有错误，然后通过一些规则筛选最后展示给用户是我们在开发一个 App 的时候需要考虑处理的，比如在下拉列表的时候，发送网络请求，这时候网络请求失败了，需要界面上展示网络错误，并且显示重新加载的按钮，或者是如果在调用相机获取授权的时用户没有授权的时候，需要提示给用户授权相关的信息，等等这些逻辑处理都可以通过流的形式处理，在处理用户网络错误加载失败的时候，通过 `RxSwift` 的一个很简单的 Api:`withLatestFrom`就能实现数据重新加载，而不需要记住各种复杂的参数。

根据错误码的不同进行不同的错误逻辑处理，如下代码所示

```swift
/**
     通过 mikerError 显示错误信息
     202024： 请登录后再操作
     - parameter error:
     */
    public func toastError(_ error:MikerError){
        if error.code == UtilCore.sharedInstance.toLoginErrorCode {
            self.toastCompletion(error.message){ _ in
                /**
                 *  在这块 就是跳转到登陆模块,如果已经跳转就不需要直接忽略 否则 先将AppData.sharedInstance.isHasToLoginVc改为true然后再跳转
                 */
                if UtilCore.sharedInstance.isHasToLoginVc == false {
                    _ = "login".openURL()
                }
            }
        } else if error.code == UtilCore.sharedInstance.toForcedupdatingErrorCode {
            /*
            表示版本强制更新
             */
            if UtilCore.sharedInstance.isHasForcedupdating == false {
                UtilCore.sharedInstance.isHasForcedupdating = true
                _ = "forcedupdating".openURL(["message":error.message])
            }

        } else {
            if UtilCore.sharedInstance.isDebug {
                self.toast(error.message)
            } else {
                 ///表示是生产模式
                let code = "\(error.code)"
                if code.hasPrefix("2") {
                    self.toast(error.message)
                } else {
                    self.toast(UtilCore.sharedInstance.errorMsg)
                }
            }
        }
    }
```

### 指令码

与服务端确认配合确定，通过`错误码`与`路由`结合能达到一种指令码的效果，客户端取到服务端返回的`错误码`的时候先进行逻辑判断，适配一些规则，如果符合则取服务端返回的`uri`字段，直接进行路由跳转，否则走错误处理抛出。这种`指令码`可以达到一些客户端的跳转逻辑交由服务端来控制，比如在注册完毕之后是跳转首页还是继续补充完详细信息的这种需求是可以根据服务端返回的`指令码`来决定。

## MVVM 架构设计

一直觉得[南峰子](http://southpeak.github.io/)翻译的这两篇文章挺不错的虽然是 2014 的文章了，感兴趣的可以看下

- [MVVM Tutorial with ReactiveCocoa: Part 1/2](http://southpeak.github.io/2014/08/08/mvvm-tutorial-with-reactivecocoa-1/)
- [MVVM Tutorial with ReactiveCocoa: Part 2/2](http://southpeak.github.io/2014/08/12/mvvm-tutorial-with-reactivecocoa-2/)

  ![mvvm](/images/20190413/mvvm.png)

另外`登陆注册模块(Carlisle)`是参考[RxSwift](https://github.com/ReactiveX/RxSwift)官方 Demo 设计的，使用 MVVM 架构设计，虽然没有严格遵守上面文章所说的 MVVM 引用层次，不过`登陆注册模块(Carlisle)`还是可以灵活的适用于不同的需求的在简单修改之后。

## 模块路由

Swift 下一直使用[URLNavigator](https://github.com/devxoul/URLNavigator)作为模块之间的路由框架使用，感觉非常方便

```swift
extension String {
    /// 返回路由路径
    ///
    /// - Parameter param: 请求参数
    public func  getUrlStr(param:[String:String]? = nil) -> String {
        let that = self.removingPercentEncoding ?? self
        let appScheme = Navigator.scheme
        let relUrl = "\(appScheme)://\(that)"
        guard param != nil else {
            return relUrl
        }
        var paramArr:[String] = []
        for (key , value) in param!{
            paramArr.append("\(key)=\(value)")
        }
        let rel = paramArr.joined(separator: "&")
        guard rel.count > 0 else {
            return  relUrl
        }
        return relUrl + "?\(rel)"
    }
    /// 直接通过路径 和参数调整到 界面
    public func openURL( _ param:[String:String]? = nil) -> Bool {
        let that = self.removingPercentEncoding ?? self
        /// 为了使html的文件通用 需要判断是否以http或者https开头
        guard that.hasPrefix("http") || that.hasPrefix("https") || that.hasPrefix("\(Navigator.scheme )://") else {
            var url = ""
            ///如果以 '/'开头则需要加上本服务域名
            if that.hasPrefix("/") {
                url = UtilCore.sharedInstance.baseUrl + that
            }else{
                url = that.getUrlStr(param: param)
            }
            // 首先需要判断跳转的目标是否是界面还是处理事件 如果是界面需要: push 如果是事件则需要用:open
            let isPushed = Navigator.that?.push(url) != nil
            if isPushed {
                return true
            } else {
                return (Navigator.that?.open(url)) ?? false
            }
        }
        // 首先需要判断跳转的目标是否是界面还是处理事件 如果是界面需要: push 如果是事件则需要用:open
        let isPushed = Navigator.that?.push(that) != nil
        if isPushed {
            return true
        } else {
            return (Navigator.that?.open(that)) ?? false
        }
    }
}
```

这块其实可以更进一步的封装，比如每次调整都可以通过正则表达式进行有效性的验证，或者一些其他路由规则判断

借助[URLNavigator](https://github.com/devxoul/URLNavigator)实现各个模块的解耦，理论上每个界面都可以实现互相跳转的，在处理商品列表界面的行点击事件(`didSelectRowAt`)的时候是由服务端返回的`uri`字段决定的，具体跳转哪个界面是有服务端决定的，个人的理解是界面负责产生信号，每个信号都会经过复杂的筛选变化又会反应到界面上的，所有的跳转事件都可以通过 `URLNavigator` 路由实现，比如逻辑处理、界面跳转等事件

每个模块都有各自的模块路由注册类，比如`Jacob_router.swift`，包含了该模块内部所有的可路由的界面和事件处理的路由注册，最后会在主模块中统一注册

## Gckit-CLI 的使用

CocoaPods 公共组件模块可以很方便集成现有的模块，但是我们每个业务都是完全不一样的，每个接口返回的 JSON 文件也不一样，然后我们得手动创建与之对应的 Model，这些操作完全没有任何意义但是又是必须的，不过现在我们可以使用 [Gckit-CLI](https://github.com/SeongBrave/gckit) 一键生成对应的所有 Model 实体类，我们只需要把对应的 JSON 文件放到对应的目录即可，[Gckit-CLI](https://github.com/SeongBrave/gckit) 不仅可以生成 Model 文件，ViewModel、ViewController、View、Cell 等各种文件，并且是一键生成，大家可以尝试使用下，如果觉得可以的话麻烦给一个**Star**吧 😂。

## 公用模块

公司的公用组件应该是长期积累的，不同的该功能，大部分是与业务无关的可以扩 App 或者夸业务使用的，经过长时间的积累会慢慢完善，比如京东内部有各种各样的模块组件，对与新开发一个项目来说会提高很多倍，这些公用组件模块通过 CocoaPods 管理，或者也可以通过 Framework 管理

以下是我个人积累的一些公用库，平常写 Demo 啥的都是非常方便的

| 模块            | 介绍            | 地址                                            |
|---------------|---------------|-----------------------------------------------|
| UtilCore      | 基础工具库         | `https://github.com/SeongBrave/UtilCore`      |
| NetWorkCore   | 网络工具库         | `https://github.com/SeongBrave/NetWorkCore`   |
| EmptyDataView | 列表为空时自定义展示空界面 | `https://github.com/SeongBrave/EmptyDataView` |

## Node.js 接口服务

[twilight_app](https://github.com/SeongBrave/twilight_app) 为项目后台的接口服务，一个客户端开发的思维开发的后台接口服务 😂，功能很简单，如果感兴趣的可以下载看下

## App 架构设计

![structurechart](/images/20190413/structurechart.png)

最顶层为 `主工程`，包含一些简单的配置、路由注册等，相当于一个空壳，模块化之后需要注意的一点是:模块的版本管理，每次发版一定要记录好每个模块的版本号等，否则代码回退、Bug 排查是一件很困难的事，我们主工程中会记录每次发版时各个模块的版本号的。接下来就是`业务层`，包括各个不同的业务模块，这些模块之间的调用是通过路由实现的，不能存在引用关系的，每个模块会依赖一个`上下文模块`和`项目配置模块`，`上下文模块`主要是管理用户对象等用户权限相关的事，`项目配置模块`主要是整体 App 的一些配置数据、以及主题颜色和一些第三方 key 的配置等(主要为了方便配置统一管理)。业务层是整个 App 的核心功能，而`公用组件模块`是跨业务、跨 App 的，不同的 App 之间是可以公用这些组件的，这一层最好作为公司级别的供大家所有人使用。最下层为第三方库，一般情况下我们需要对第三方做一层脱离耦合的封装，以便我们在修改第三方时而不影响我们的业务模块。整个项目从上到下为依赖关系，下层为上层提供功能服务。

## 总结

本文简单介绍了自己在 Swift 模块化项目中的一些实践经验，借助 RxSwift 实现 MVVM 框架的设计，内容比较杂，供大家参考，随着 Swift 5 的发布，Swift ABI 的稳定，相信会有更多团队会选择 Swift 语言开发自己的 App 的， 周围认识的很多朋友都说如果尝试过 Swift 之后就很难再回去用 Objective-C 了，Swift 本身带有的很多特性是 Objective-C 不具有的，呀感觉又扯远了，我个人比较喜欢通过一些工具去实现一些效率方面的提升的，通过模块化实现代码的复用，通过一些脚本工具实现重复无用代码的自动生成，比如 Model 文件的生成等，这样我们通过借助 CocoaPods 和 [Gckit-CLI](https://github.com/SeongBrave/gckit) 结合使用，使我们的开发效率大大提高了，节省出来的时间我们专注于业务功能的开发。

🤝 最后感谢您的阅读!
