---

created at: 2021-03-22 22:40:57

---

# Observable

在 ReactiveX 中，观察者订阅 Observable 对象。 观察者会响应任何 Observable 对象发出的信号或信号序列。这种模式加快了并发操作，因为在等待 Observable 发出信号的过程中不需要阻塞，而是以 Observer 的形式创建了一个”哨兵“，准备在未来任何时间当 Observable 发出信号时做出适当的响应。 

本页解释了什么是响应模式以及什么是 Observable 和 Observer（以及 Observer 如何订阅Observable）。其它页面展示了如何使用[各种 Observable 算子](http://reactivex.io/documentation/operators.html)将 Observable 连接到一起并修改其行为。 

本文档在解释时附带”弹珠图“，下图展示了弹珠图是如何表现 Observable 和 Observable 转换的。

// FIXME

参见：

- [Single](http://reactivex.io/documentation/single.html) — a specialized version of an Observable that emits only a single item
  
- [Rx Workshop: Introduction](http://channel9.msdn.com/Series/Rx-Workshop/Rx-Workshop-Introduction)
  
- [Introduction to Rx: IObservable](http://www.introtorx.com/Content/v1.0.10621.0/02_KeyTypes.html#IObservable)
  
- [Mastering observables](http://docs.couchbase.com/developer/java-2.0/observables.html) (from the Couchbase Server documentation)
  
- [2 minute introduction to Rx](https://medium.com/@andrestaltz/2-minute-introduction-to-rx-24c8ca793877) by Andre Staltz (“Think of an Observable as an asynchronous immutable array.”)
  
- [Introducing the Observable](https://egghead.io/lessons/javascript-introducing-the-observable) by Jafar Husain (JavaScript Video Tutorial)
  
- [Observable object](http://xgrommx.github.io/rx-book/content/observable/index.html) (RxJS) by Dennis Stoyanov
  
- [Turning a callback into an Rx Observable](https://afterecho.uk/blog/turning-a-callback-into-an-rx-observable.html) by @afterecho
  

## 背景

在许多编程任务中，你或多或少希望编写的指令能在一次执行中按编写顺序逐步完成。

但在 ReactiveX 中，许多指令可以并行执行，并且其结果随后由 ”Observable“ 以任意顺序捕获。无需调用方法，而是以 “Observable" 的形式定义一种用于检索和转换数据的机制"，此时，先前定义的机制将在 Observer 站岗的情况下随时准备捕获并响应 Observable 发出的信号。 

这个方法的优点是，当你有一堆彼此不依赖的任务时，你可以同时启动所有任务，而不必等每个人物都完成才开始下一个任务 —— 这样，你完成任务集合所需的时间与完成集合中最长任务所需的时间一样长。 

有许多方式用来描述这种异步编程和设计模型。本文档将使用这种方式：Observer 订阅 Observable。Observable 发出信号或通过调用 Observer 的方法向其发送通知。        的上下文中，我们所谓的 ”observer" 有时也称为 “subscriber”、“watcher” 或 ”reactor“。这个模型同房被称为 ”reactor 模式“。 

## 建立观察者

本页的例子使用类似 Groovy 的伪代码，但需要注意的是， ReactiveX 已经在很多语言中实现。 

在普通方法调用中 —— 不是 ReactiveX 中典型的异步并行调用 —— 流程如下：

1. 调用一个方法
  
2. 将该方法返回值储存在一个变量中
  
3. 使用该变量执行一些有用的操作
  

或者像这样：

```java
// 进行调用，并将返回值分配给 `returnVal`
returnVal = somMethod(itsParameters);
// 用 returnVal 做一些有用的操作
```

在异步模型中，流程更像这样：

1. 定义一个对异步调用返回值做一些操作的方法；此方法是 Observer 的一部分。
  
2. 将异步调用本身定义为 Observable。
  
3. 通过订阅 Observable 的方式把 Observer 附加到 Observable 上。（这也会初始化 Observable 的动作）。
  
4. 继续你的业务逻辑；无论
