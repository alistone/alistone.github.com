---
layout: post
title: "NSOperationQueue与GCD的区别"
date: 2014-10-01 17:10:09 +0800
comments: true
categories: iOS
---

NSOperationQueue与GCD都可以用于iOS的多线程编程，GCD是由底层C语言构成，NSOperationQueue则使用抽象的Objective C对象，其底层其实也是由GCD实现的，两者主要有以下不同：

+ GCD只支持不同队列间的优先级，NSOperationQueue则支持同一个队列里设置不同的优先级，而不同队列间的优先级则是隔离的。
+ NSOperationQueue可以方便的设置两个Operation的依赖关系，即时它们不在同一个Operation Queue里，但相同的功能GCD就很难实现。
+ NSOperationQueue可以取消添加到queue里还未执行的Operation，GCD则不行。
+ NSOperationQueue支持KVO，方便观察Operation的执行状态。
+ NSOperation做为Objective C对象可以方便的继承和复用，而GCD就很难做到。

总结起来使用NSOperationQueue有很多优势，但GCD也有它的优势就是简单和更好的性能，虽然两者之间的性能差距在很多情况下可以忽略，但是很多场景下简单就是最大的优势。
