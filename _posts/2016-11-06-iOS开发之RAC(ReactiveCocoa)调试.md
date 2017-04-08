---
layout: post
title: 2016-11-06-iOS开发之RAC(ReactiveCocoa)调试
date: 2016-11-06
categories: blog
tags: [iOSDev]
description: 2016-11-06-iOS开发之RAC(ReactiveCocoa)调试

---


#RAC调试

##1.RAC 源码下有 instruments 的两个插件，方便大家使用。

- signalEvents 这个可以看到流动的信号的发出情况，对于时序的问题可以比较好的解决。

- diposable 可以检查信号的 disposable 是否正常

##2.打印别名给信号一个名字，然后通过下面的打印方法来进行调试


```
/// Additional methods to assist with debugging.
@interface RACSignal (Debugging)

/// Logs all events that the receiver sends.
- (RACSignal *)logAll;

/// Logs each `next` that the receiver sends.
- (RACSignal *)logNext;

/// Logs any error that the receiver sends.
- (RACSignal *)logError;

/// Logs any `completed` event that the receiver sends.
- (RACSignal *)logCompleted;

```


 ```
  增加log方法
 DExecute(({
    setenv("RAC_DEBUG_SIGNAL_NAMES", "RAC_DEBUG_SIGNAL_NAMES", 0);
    [signalUserGeo setNameWithFormat:@"signalUserGeo"];
    signalUserGeo = [signalUserGeo logAll];
}));


```

- 如果是性能调试，主要是经验 +Instruments，经验类似于：少用 RACCommand、RACSequence 这样的，Instruments 可以用它的 Time Profile 来看。


- 如果是 Bug 调试，主要还是靠 Log，配合一些 Xcode 插件，比如 MCLog(可以很方便地过滤日志)，如果要还原堆栈的话，就加一个断点

- 保证一个 RACSignal 只会给订阅者 send 一种类型的 value，所以就手动给 signal 加了部分泛型支持

- Sequence 的性能很差,最好不用


