---
layout: post
title: "GCD学习笔记"
date: 2014-02-25 13:58:49 +0800
comments: true
categories: iOS
---

本文是个人学习GCD后所做的笔记，如有错漏欢迎指正。

###GCD简介

GCD（Grand Central Dispatch）是一种简单易用的异步执行任务的技术，一般来说GCD会将任务封装到block中然后提交给dispatch queue，dispatch queue会根据任务提交的顺序串行或者并行的执行任务。

###Dispatch Queue的分类

* Serial Dispatch Queue：添加到queue中的block按FIFO的顺序串行处理。
* Concurrent Dispatch Queue：添加到queue中的block按FIFO顺序调度处理，前一个block调度后无须等block执行完毕就可以调度下一个block，所以不同的block是并发执行的，具体并发执行的线程数GCD会根据系统负载自动调整。

###Main Dispatch Queue和Global Dispatch Queue

* Main Dispatch Queue：通过`dispatch_get_main_queue()`获取，在主线程中执行，属于Serial Dispatch Queue。
* Global Dispatch Queue：通过`dispatch_get_global_queue(priority,0)`获取，属于全局的Concurrent Dispatch Queue，其中priority指定queue的优先级，优先级高的queue中的block会优先调度。

###自定义Dispatch Queue

* 通过`dispatch_queue_t queue = dispatch_queue_create("queueName", attr);`创建自定义dispatch queue，attr为NULL或DISPATCH_QUEUE_SERIAL则创建的是Serial Dispatch Queue，attr为DISPATCH_QUEUE_CONCURRENT则创建的是Concurrent Dispatch Queue。自定义的dispatch queue需要在结束时调用`dispatch_release(queue)`释放。
* 自定义dispatch queue的优先级相当于DISPATCH_QUEUE_PRIORITY_DEFAULT,使用`dispatch_set_target_queue(queue,globalQueue)`可以将queue设置成与globalQueue相同优先级。

###GCD相关的API

* `dispatch_sync(queue,block)`：  
将block同步添加到queue中，并等待block执行完毕再返回。慎用这个方法因为容易引起死锁，比如在主线程中调用dispatch_sync(dispatch_get_main_queue(),^{});
* `dispatch_async(queue,block)`：  
将block异步添加到queue中并直接返回，无须等待。
* `dispatch_after(delayTime,queue,block)`：  
在delayTime后将block添加到queue中执行。
* `dispatch_once(&onceToken,block)`：  
保证应用程序中只执行一次block。
* `dispatch_barrier_sync(queue,block)`和`dispatch_barrier_async(queue,block)`：  
类似于dispatch_sync和dispatch_async添加block到queue中，只是这个block被标记为barrier，意思是barrier前添加的所有block执行完毕后才执行barrier，barrier执行完毕后才执行后面添加的block。
* `dispatch_suspend(queue)`和`dispatch_resume(queue)`：  
挂起和恢复指定queue，对queue中已经执行的block没影响。
* `dispatch_apply(count,queue,^(size_t idx){})`：  
将block重复添加到queue中count次，block有一个参数指明重复次数，取值为0~count-1，该方法需要等所有block执行完毕才返回。
* dispatch_group：

        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0); 
        dispatch_group_t group = dispatch_group_create(); 
        dispatch_group_async(group, queue, block1); 
        dispatch_group_async(group, queue, block2); 
        dispatch_group_async(group, queue, block3); 
        dispatch_group_notify(group, queue, block4); 
        long result = dispatch_group_wait(group,timeout);
`dispatch_group_notify(group,queue,block4)`会等到queue中所有block全部执行完毕后再异步添加block4到queue中，这样就保证了block4在block1、block2、block3全部执行完毕后再执行。
`dispatch_group_wait(group,timeout)`会同步等待group中所有block执行完毕，或者等到指定timeout时间为止，返回result为0则表示所有block全部执行完毕，非0则表示还有block在执行。
