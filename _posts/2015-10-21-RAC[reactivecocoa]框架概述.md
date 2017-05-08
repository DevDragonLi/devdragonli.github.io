---
layout: post
title: 2015-10-21-RAC[reactivecocoa]框架概述
date: 2015-10-21
categories: blog
tags: [iOSDev]
description: RAC[reactivecocoa]框架概述

---
##1.比较适合用RAC开发中场景： 
对于一个应用来说，绝大部分的时间都是在等待某些事件的发生或响应某些状态的变化，比如用户的触摸事件、应用进入后台、网络请求成功刷新界面等等，而维护这些状态的变化，常常会使代码变得非常复杂，难以扩展。而 ReactiveCocoa 给出了一种非常好的解决方案，它使用信号来代表这些异步事件，提供了一种统一的方式来处理所有异步的行为，包括代理方法、block 回调、target-action 机制、通知、KVO 等

一、UI 操作，连续的动作与动画部分，例如某些控件跟随滚动。

二、网络库，因为数据是在一定时间后才返回回来，不是立刻就返回的。

三、刷新的业务逻辑，当触发点是多种的时候，业务往往会变得很复杂，用 delegate、notification、observe 混用，难以统一。这时用 RAC 可以保证上层的高度一致性，从而简化逻辑上分层
#2.框架概述
ReactiveCocoa 主要由以下四大核心组件构成：

-信号源：RACStream 及其子类；RACStream 就是以 Monad 的概念为依据进行设计的，它代表的就是一个 Monad,有了 Monad 作为基石后，许多基于流的操作就可以被建立起来了，比如 map 、filter 、zip 等

* RACStream

* RACSignal

* RACSubject

* RACSequence

-订阅者：RACSubscriber 的实现类及其子类；

- RACSubscriber

- RACMulticastConnection

-调度器：RACScheduler 及其子类；

-清洁工：RACDisposable 及其子类

##1Streams

由RACStream抽象类来表示的Steam是任意对象的值的一个序列(any series of object values)。

值可以立即使用也可以在之后使用，但是必须按照顺序进行检索。不能在没有获取或者等待(without evaluating or waiting for the first value)第一个值的情况下获取第二个值。

Streams是单子（monads），除其他事项外，这使得复杂的操作必须建立在一些基本的原函数(primitives)上（特别是-bind:）。RACSteam也实现了Haskell中Monoid与MonadZip类型的等效(the equivalent of the Monoid and MonadZip typeclasses from Haskell)。

RACStream自身并不是十分有用。大多数stream被当作信号和序列来使用(treated as signals or sequences instead)。
##1.1流的解释
一个流就是一个将要发生的以时间为序的事件序列。它能发射出三种不同的东西：一个数据值（data value）(某种类型的)，一个错误（error）或者一个“完成（completed）”的信号。比如说，当前按钮所在的窗口或视图关闭时，“单击”事件流也就“完成”了
ReactiveCocoa 中的值流只包含正常的值，即通过 next 事件传送的值，并不包括 error 和 completed 事件，它们需要被特殊处理。通常情况下，一个信号的生命周期是由任意个 next 事件和一个 error 事件或一个 completed 事件组成的

##1.2事件：Events
一个事件, 用 Event类型表示, 表示某些事情已经发生。 在RAC中事件是传播（center-piece of communication）的核心。 一个事件可能是button的一次点击，从API返回的一些信息，一个错误的发生，或者一个长时间操作完成了。无论如何，一些东西产生事件，然后通过signal发送给每个订阅这个signal的观察者。

 Event是一个枚举类型，可能有四种值（Next中有值，其他三种表示结束）：

 Next代表有一个新的值从源产生。(可以为nil)
 Failed说明在信号源完成前发生了一个错误。事件会被当做一个类型为 ErrorType的参数，一种在事件中声明过的表示已知错误的类型。如果这个错误没有被声明许可过，可以用 NoError表示。
 Completed说明事件已经成功结束。不会再有值发送出来。
 Interrupted说明事件被取消了，意味着操作既没有成功也没有失败。

##2 Signals--信号
信号源代表的是随着时间而改变的值流
###2.1RACSignal 还有五个用来实现不同功能的私有子类：

-RACEmptySignal ：空信号，用来实现 RACSignal 的 +empty 方法；

-RACReturnSignal ：一元信号，用来实现 RACSignal 的 +return: 方法；

-RACDynamicSignal ：动态信号，使用一个 block 来实现订阅行为，我们在使用 RACSignal 的 +createSignal: 方法时创建的就是该类的实例；

+RACErrorSignal ：错误信号，用来实现 RACSignal 的 +error: 方法；

+RACChannelTerminal ：通道终端，代表 RACChannel 的一个终端，用来实现双向绑定

###2.2什么是冷信号与热信号
1.代表的是未来将会被传送的值，它是一种 push-driven 的流。RACSignal 可以向订阅者发送三种不同类型的事件()
RACSignal:代表的是未来将会被传送的值

冷热信号的概念源于C#的MVVM框架Reactive Extensions中的Hot Observables和Cold Observables:

Hot Observables和Cold Observables的区别：

Hot Observables是主动的，尽管你并没有订阅事件，但是它会时刻推送，就像鼠标移动；而Cold Observables是被动的，只有当你订阅的时候，它才会发布消息。
Hot Observables可以有多个订阅者，是一对多，集合可以与订阅者共享信息；而Cold Observables只能一对一，当有不同的订阅者，消息是重新完整发送。
这里面的Observables可以理解为RACSignal

Signal有RACSignal类描述。它是一个push-driven的流。

信号一般表示将要传递的数据，随着工作（方法）的执行或者数据的接受，数据通过信号得到传递到订阅者(subscribers)处。用户必须订阅(subscribe)一个信号来访问它的值。

在三种不同类型的事件下信号会被发送给它的订阅者：
(Signals send three different types of events to their subscribers:)？？
信号为它的订阅者发送三种不同类型的事件：

Steam中的下个(next)事件提供了新的值。RACSteam方法只会在这种类型的事件时执行。不想Cocoa中的collections，在信号中使用nil也是有效的。
错误事件(error evenet)表示在信号结束前发生了一个错误。这个事件可能包含了一个NSError对象，指示什么地方除了错误。错误必须得到特殊的处理(handled specially)，因为它们不包含在steam的值中。
完成(complete)事件表示信号成功的结束了，并且没有更多的值被添加到steam中。完成后也必须进行特殊处理，原因相同。
一个信号的生命周期包括任何数量的next事件，以及一个error或者completed事件（两者不需要都有）。

##3Subscription--订阅

订阅者(subscriber)是正在等待或者能够等待来自信号的事件的任何对象。在RAC中，订阅者被表示为一个符合RACSubscriber协议的任何对象。

订阅可以通过调用-subscribeNext:error:completed:或者相应的适宜的方法来创建。从技术上讲，大部分RACStream和RACSignal方法能够很好的创建一个订阅，但是，这些中间订阅(intermediate subscriptions)通常是一个实现细节(implementation detail)。

订阅保留(retain)他们的信号，并在信号完成或者出错时被处理。订阅也可以被手动地处理。

##4Subjects

subject是一个可以手动控制的信号。它由RACSubject类进行描述。

subject可以看作是信号的一个“可变”变体(variant)，就像NSMutableArray之于NSArray。它对于桥接(bridging)非RAC代码到信号中去(into the world of signals)非常有用。

举例来说，block可以简单的发送一个事件(event)给共享的subject来替代在block的回调中处理应用程序逻辑。之后subject可以返回一个RACSignal，隐藏回调的实现细节。

一些subject提供了额外的方法(behaviors)。尤其像RACReplaySubject能够被用来为之后的订阅者缓冲事件(buffer events)，比如，一个网络请求在其他准备好处理请求结果之前完成。(when a network request finishes before anything is ready to handle the result)

##5Commands

command创建并订阅一个信号以响应一些动作(action)。这使得它很容易处理UI与应用之间的side-effecting work。

通常触发command的动作是由UI驱动的，比如按钮的点击。命令也可以根据信号被自动禁用，并且这种禁用状态能过通过禁用任何与command相关的控件来表现在UI上。

在OS X上，RAC为NSButton增加了一个rac_command属性来自动设置上述行为(behaviors)。

##6Connections

connection是一个在任意数量订阅者之间共享的一个订阅(subscription)。它由RACMulticastConnection类表示。

信号在默认情况下是“冷(code)”的，意思是，每当有一个新的订阅被添加了，信号就开始工作。这意味着每个订阅者的数据(data)会被重新计算。这种行为在一般情况下是可取的，但是如果信号具有side effects或者所要做的工作代价是昂贵的（比如发送一个网络请求）时，这就会有一些问题。

connection时通过RACSignal的-publish或者-muticast:方法创建的，并确保无论connection被订阅了多少次，只有一个相关的订阅被创建了。一旦连接了，连接的信号就被认为是“热(hot)”的，并且那个相关订阅会被保持活跃(remain active)直到所有connection上的订阅都被处理。

##7Sequences--序列
RACUnarySequence ：一元序列，用来实现 RACSequence 的 +return: 方法；

RACIndexSetSequence ：用来遍历索引集；

RACEmptySequence ：空序列，用来实现 RACSequence 的 +empty 方法；

RACDynamicSequence ：动态序列，使用 blocks 来动态地实现一个序列；

RACSignalSequence ：用来遍历信号中的值；

RACArraySequence ：用来遍历数组中的元素；

RACEagerSequence ：非懒计算的序列，在初始化时立即计算所有的值；

RACStringSequence ：用来遍历字符串中的字符；

RACTupleSequence ：用来遍历元组中的元素

序列是个pull-driven的流，它由RACSequence类描述。

序列是一种集合，与NSArray类似。但是与数组不同的是，sequence中的值在默认情况下是延迟计算(evaluated lazily)的，如果只有一部分的sequence被使用，这会提高一定的性能。与Cocoa中的集合一样，序列中不允许包含nil。

序列与Clojure中的序列（特别是lazy-seq）以及Haskell中的List类型相似。

RAC给大多数Cocoa的集合类添加了-rac_sequence方法，允许它们使用作为RACSequences。

##8Disposables
RACDisposables用于取消订阅与资源清理(cancellation and resource cleanup)。

Disposables最常用来退订(unsubscribe)一个信号。当一个订阅被处理之后，相应的订阅者将不会从信号中接受任何事件。另外，任何与订阅相关的的工作（后台进程、网络请求等等）会被取消，结果也不再需要。

##9Schedulers
Scheduler是信号执行工作以及返回它们的结果的一个串行执行队列。它由RACScheduler类描述。

Scheduler类似于GCD的队列(queue)，但是scheduler支持取消操作，并且始终串行执行。唯一的例外是+immediateScheduler，schedulers不提供同步执行。这有助于避免死锁，并促使(encourages)用信号操作来替代block。

RACScheduler有时候也像NSOperationQueue，但是schedulers不允许任务重新排序与相互依赖。

##10Value types
为了方便表示流(steam)中的值，RAC提供了一些杂类(miscellaneous classes):

RACTuple是一个小型的、固定大小的集合，可以包括nil(由RACTupleNil表示)。它一般表示多个流的组合值(combined values)。
##11RACUnit
RACUnit是一个单例(singleton)的“空”值。它被用来表示那些在留中不存在的更有意义的数据。
RACEvent把任何信号事件(signal event)表示为信号值(signal value)。它主要通过RACSignal的-materialize来使用。
Asynchronous Backtraces
因为基于RAC的代码往往设计到异步的工作以及队列跳转(queue-hopping)。ReactiveCocoa框架支持支持捕获异步回溯，使得调试更加容易。

在OS X中，回溯会自动从任何代码捕捉，包括系统的库。

在iOS中，只有队列在RAC内跳转你的项目才会捕捉到（但是信息任然是有效的）。


#补充版本

##1 一个信号用Signal类型表示,是一连串随着时间发出的可以被观察的事件。

信号通常用来表示事件流正在发出，比如通知，用户的输入等。每当动作被执行或者数据已经接受，事件们就会通过signal发出，signal会把它们推送给每个观察者。所有的观察者都会同时接受到事件。

用户如果想要接收它们的事件必须observe（观察）这个signal。观察一个信号不会触发其他副作用。换句话说，事件是源驱动，基于推送，观察者在整个生命周期里不会受到到任何影响。当观察一个信号时，用户只能按照顺序处理信号里的事件。不能随意访问信号里的事件。

信号可以被操作符操作。常用的操作一个信号的有filter，map和reduce，zip可以一次处理多个信号源。操作符只能在Next事件中才能使用。

信号的整个生命周期有一组Next事件组成，最后是一个终结事件，可能是Failed, Completed, 或者Interrupted中的任一个。终结事件没有被包含在事件的值里，他们需要被单独处理。
##2管道：Pipes
一个管道，通过 Signal.pipe()创建。一个可以被手动控制的信号。
这个方法返回一个信号和一个observer。可以控制信号发送事件给观察者。这个在将非RAC的代码转变到信号世界里特别有用。

比如，可以不用在block的回调里处理业务逻辑，将blocks简化成发送事件给观察者。同时信号可以被返回，隐藏回调里的具体实现。

##3信号生产者：Signal Producers
一个信号生产者,以  SignalProducer类型表示,创建信号并且产生副作用。

可以用来表示一组操作或者任务，比如网络请求，每次 start()调用后会创建一个新的操作，允许发起者观察结果。通过 startWithSignal()可以访问到产生的信号，允许被多次观察。

因为 start()这种行为的不同，每次从同一个信号生产者可能会得到不同顺序或者版本的事件，甚至整个流可能完全不同。不像一个普通的信号，直到有一个观察者被添加才会开始启动，每次都会为新添加的观察者重新工作一次。

开启一个信号生产者会返回一个 disposable，用了中断或者取消（interrupt/cancel）这个信号生产者的工作。
和信号一样，信号生产者可以被操作符比如map，filter等操作。每个信号的操作符都可以通过“lifted”迁移后在信号生产者上使用。而且，还有几个特有的操作符用了控制工作什么时候开始和怎么运行，比如 times。

##4缓冲：Buffers
一个缓冲通过  SignalProducer.buffer() 创建,是一个事件的队列（通常指定数量），当新信号产生时，会重新执行队列里的事件。

和 pipe相似，这个方法返回一个观察者。每个发给这个观察者的事件会被加入队列。如果这个缓冲区已经达到创建时预定的数量，当新的事件发来时，最早的一个会被移出队列。

##5观察者：Observers
 Observer是指任何等待从信号中接收事件的东西。

Observers可以通过 Signal.observe或者 SignalProducer.start隐式获得。

##6动作：Actions
动作用  Action类型表示，指当有输入时会做一些工作。当动作执行时，会有0个或者多个值输出；或者会产生一个失败。

Action用来处理用户交互时做一些处理很方便，比如当一个按钮点击时这种动作。Action也可以和一个属性自动关联disabled。比如当一个UI控件的关联Action被设置成disabled时，这个控件也会disabled。

为了和NSControl和UIControl交互，RAC提供了 CocoaAction类型可以桥接到OC下使用

##7属性：Properties
一个属性表现为  PropertyType协议（protocol）, 保存一个值，并且会将将来每次值的变化通知给观察者们。

property的当前值可以通过获取 value获得。 producer返回一个会一直发送值变化信号生成者（signal producer ），

 <~运算符是提供了几种不同的绑定属性的方式。注意这里绑定的属性必须是 MutablePropertyType类型的。

 property <~ signal 将一个属性和信号绑定在一起，属性的值会根据信号送过来的值刷新。
 property <~ producer 会启动这个producer，并且属性的值也会随着这个产生的信号送过来的值刷新。
 property <~ otherProperty将一个属性和另一个属性绑定在一起，这样这个属性的值会随着源属性的值变化而变化。
 DynamicProperty 类型用于桥接OC的要求KVC或者KVO的API，比如 NSOperation。要提醒的是大部分AppKit和UIKit的属性都不支持KVO，所以要观察它们值的变化需要通过其他的机制。相比 DynamicProperty要优先使用  MutablePropertyType类型。

##8销毁：Disposables
disposable表现为 Disposable 协议,用于内存管理和释放销毁。

当你启动一个signal producer，一个disposable会被返回。可以用于被调起者取消已经启动的signal producer（比如后台线程的处理，网络请求等），清除临时资源，发送一个最终的 Interrupted事件给它创建的信号。

观察一个信号也会返回一个disposable。调用后就不会再收到这个信号发过来变化的值，但是这对信号本身不会产生影响。

更多关于销毁的信息查看这份文档：RAC Design Guidelines.

##9调度器：Schedulers
调度器，类型是 SchedulerType 协议, 是一个序列化的要被执行的任务队列或者是一组向外输出的结果。

信号和信号生成者可以按照安排好的次序发送事件到一个指定的 scheduler。信号生产者还可以在指定的调度器上被启动。

This helps avoid deadlocks, and encourages the use of signal and signal producer primitives instead of blocking work.
scheduler很像GCD，但是scheduler可以被销毁（通过Disposable），而且总是连续执行。由于 ImmediateScheduler 会引发异常, scheduler不提供同步的操作。这样可以避免出现死锁，还鼓励使用信号的操作符而不是blocking work。
scheduler也有点像NSOperationQueue, 但是scheduler不允许任务根据另一个调度器而改变顺序。

--------------------
-










