---
layout:     post
title:      "iMessage App 在国行手机上无法联网的问题"
subtitle:   ""
date:       2016-12-08 22:00
author:     "isan"
header-img: "img/great-fire-wall-bg.jpg"
catalog: true
tags:
	- iOS
    - iMessage App
---

我有一台港行的 iPhone 6 和一台国行的 iPad Air 2， 开发iMessage App 的时候我一直不知道 国行 iPhone 的 iOS 10 加入了应用访问网络的权限控制，这好像是我国相关部分出台的规定┑(￣Д ￣)┍。开发阶段完成后公司同事参与测试时发现部分同事没办法访问网络，我就懵逼了，同样的网络环境，有的机器能访问有些机器不能访问。。。我各种查，后来有同事说看到新闻说 iOS 10 引入了应用网络访问的权限控制以及一位同事鬼使神差般的猜测会不会是国行机器的问题才算是弄清楚了问题。

之所以没有快一点找到问题根源的一个重要原因是：普通 App 在首次访问网络的时候系统会弹框询问用户是否允许网络访问，而我们的 iMessage App 没有一个 host App，系统不会询问用户是否允许网络访问。

于是我去 [bugreport](https://bugreport.apple.com)提交了bug 28382947。那发生在9月20号。。。

今天（12月7号） Apple 终于给我正式回复了：

![img](/img/in-post/dev/iMessage-bug.png)

我用一台国行 iPhone 刷了一个 iOS 10 beta 6, 测试了一下果然可以了，但是系统并没有弹框问我要网络的访问权限而是可以直接访问网络，不知天朝会作何应对呢┑(￣Д ￣)┍。

总之，正常了，可以了。


活在天朝，好累。
如果有钱的话，可能就不累了🤔。