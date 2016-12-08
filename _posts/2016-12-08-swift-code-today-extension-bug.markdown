---
layout:     post
title:      "Swift 代码构建 Today Extension UI 时遇到的bug"
subtitle:   ""
date:       2016-12-08 22:00
author:     "isan"
header-img: "img/bug-bg.jpg"
tags:
    - Swift
    - Today Extension
    - iOS
---


之前没有开发过 Today Extension, 最近业余在做 [Routine](http://routine.isan.io), 想加一个 Today Extension 。

看了[官方的文档](https://developer.apple.com/library/content/documentation/General/Conceptual/ExtensibilityPG/Today.html), 很简单，可以了，开始做，万万没想到被坑到了，现在把这个坑记录下来，也许能够帮助一些人。

新建 Today Extension 的 target 之后，我立马把 storyboard 文件删掉，然后按照官方的文档把 Info.plist 中的 `NSExtensionMainStoryboard` 字段删掉并加上了 `NSExtensionPrincipalClass` 字段，之后我就运行看效果，当然我得到了：

```
2016-12-08 20:27:48.198 fenglewidget[1296:58259] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** setObjectForKey: object cannot be nil (key: 771765C3-7639-4B61-B58C-59E88223CF3C)'
*** First throw call stack:
(
	0   CoreFoundation                      0x03917212 __exceptionPreprocess + 194
	1   libobjc.A.dylib                     0x00663e66 objc_exception_throw + 52
	2   CoreFoundation                      0x03825b77 -[__NSDictionaryM setObject:forKey:] + 1015
	3   Foundation                          0x003e833b -[_NSExtensionContextVendor _setPrincipalObject:forUUID:] + 105
	4   Foundation                          0x003e7835 __105-[_NSExtensionContextVendor _beginRequestWithExtensionItems:listenerEndpoint:withContextUUID:completion:]_block_invoke + 938
	5   libdispatch.dylib                   0x06d64e07 _dispatch_call_block_and_release + 15
	6   libdispatch.dylib                   0x06d876ef _dispatch_client_callout + 14
	7   libdispatch.dylib                   0x06d6bf3a _dispatch_queue_serial_drain + 908
	8   libdispatch.dylib                   0x06d6c728 _dispatch_queue_invoke + 1103
	9   libdispatch.dylib                   0x06d6cabc _dispatch_queue_override_invoke + 357
	10  libdispatch.dylib                   0x06d6e4bc _dispatch_root_queue_drain + 384
	11  libdispatch.dylib                   0x06d6e2d4 _dispatch_worker_thread3 + 134
	12  libsystem_pthread.dylib             0x07101d5e _pthread_wqthread + 1070
	13  libsystem_pthread.dylib             0x0710190a start_wqthread + 34
)
libc++abi.dylib: terminating with uncaught exception of type NSException
```

从崩溃的堆栈可以看出程序是在给一个 key 设置了 nil 的 value 时崩溃的，但是看不出来是哪里。于是我打开了异常断点，重新运行程序，捕获异常在这个地方：
![img](/img/in-post/dev/swift-oc-runtime-bug.png)
可以看出是 Objective-C 抛出的异常，但我明明用的是 Swift，所以我猜测是跟 Objective-C 的运行时有关。
在网上各种搜，能找到的都是教你怎么用代码而不是 storyboard 来构建 Today Extension 的 UI，也就是官方文档的那一套。

我没办法，用 OC 创建了一个测试项目，加上 Today Extension target，按照之前的做法走了一遍，运行之后没有问题，我更加确定了这个问题肯定是跟 Objective-C 的运行时有关。

后来我在[苹果官方开发者论坛](https://forums.developer.apple.com)翻关于 App Extension 的帖子终于让我在这一个[帖子](https://forums.developer.apple.com/thread/6677)中找到了一个回答：

> I finally figured out what the heck was going on here. Turns out the difference was that the project I had created was using Swift. I tried creating an identical project using Objective-C instead, and everything worked as expected. The reason was that the extension mechanism is looking up the NSViewController subclass to instantiate by name, based on the NSExtensionPrincipalClass value in the extension's Info.plist. However, when dealing with a Swift class, the module name needs to be included in order for the Objective-C runtime to be able to find it. Changing the NSExtensionPrincipalClass value from "PhotoEditingViewController" to "MyExtension.PhotoEditingViewController" (where "MyExtension" is the name of the extension target), then it's able to find the Swift class and the extension loads properly in Photos. I've filed this with 

简单翻译一下就是：

> Extension 的机制是通过 Info.plist 中的 `NSExtensionPrincipalClass` 字段提供的类名来实例化一个 ViewController，然而当提供的是一个 Swift 类时，这个类所在的模块名称也必须包含在类名中，否则 Objective-C runtime 没有办法找到这个类，所以设置 `NSExtensionPrincipalClass` 字段时应该设置成：`ExtensionName.ViewConntrollerName`

这个解答是正确的，也解释了之前的崩溃：

```
setObjectForKey: object cannot be nil
```

好了，可以了。
