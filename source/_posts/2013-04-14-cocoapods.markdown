---
layout: post
title: "CocoaPods"
date: 2013-04-14 07:08
comments: true
categories: Objective-c
---

在 ruby 中用惯了 gem， python 中有 easy_install 和 pip。 总希望开发 ios 应用的时候也可以有这样方便的包管理工具可以简化自己的开发流程。非常幸运，最近知道了 [CocoaPods](https://github.com/CocoaPods/CocoaPods), 一个为 mac 和 ios 开发者提供的包第三方包管理工具。

安装 cocoapods 需要 ruby gem 的支持。 这个应该不是问题， mac 系统自带 ruby。

直接命令行运行：

``` sh
gem install cocoapods
```

我们可以用 **pod list** 命令列出可以被安装的插件列表， 非常长。 所以更多时候我们可以用 **pod search 'keyword'** 命令来找出我们需要的插件。

和 ruby gem 一样，我们需要在项目目录下建立一个 Podfile （对应 ruby 中的 Gemfile）。 格式如下：

``` ruby
platform :ios
pod 'JSONKit',       '~> 1.4'
pod 'Reachability',  '~> 3.0.0'
pod 'ASIHTTPRequest'
pod 'RegexKitLite'
```

是不是和 Gemfile 很类似？ 完成后在命令行运行 ：

``` sh
pod install
```

最后我们需要用 pod 生成的 .xcworkspace 文件来代替 xcode 默认的 .xcodeproj 来启动项目就可以了。

##原理

1. Pods 项目最终会编译成一个名为 libPods.a 的文件，主项目只需要依赖这个.a文件即可。
2. 对于资源文件，CocoaPods 提供了一个名为 Pods-resources.sh 的 bash 脚本，该脚本在每次项目编译的时候都会执行，将第三方库的各种资源文件复制到目标目录中。
3. CocoaPods 通过一个名为 Pods.xcconfig 的文件来在编译时设置所有的依赖和参数。

