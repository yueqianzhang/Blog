---

souce: http://reactivex.io/intro.html

created at: 2021-03-21

---

## ReactiveX

ReactiveX 是一个通过使用可观察序列来构建异步、基于事件的程序的库。 

它扩展了观察者模式来支持数据和/或事件的序列，并添加了一组允许你声明式的把序列组合到一起的操作符，同时通过抽象使你不用考虑低级线程、同步、线程安全、并发数据结构、非阻塞 IO。 

Observables 通过理想的方式填平了访问异步多项序列的鸿沟。

|     | 单项  | 多项  |
| --- | --- | --- |
| 同步  | `T getData()` | `Iterable<T> getData()` |
| 异步  | `Future<T> getData()` | `Observable<T> getData()` |

这种方式有时被叫做“函数响应式编程”，但是这种说法是不恰当的。ReactiveX 可能是函数式也可能是响应式，但是它和“函数响应式编程”是不同的物种。一个主要的区别就是函数响应式编程在随时间连续变化的值上运行，而 ReactiveX 是在随时间推移发出的离散值上运行。（进一步了解函数响应式编程：[Conal Elliott’s work for more-precise information on functional reactive programming](https://github.com/conal/essence-and-origins-of-frp).) 

## 为什么使用 Observables？

ReactiveX Observables 模型使你可以使用类似用于数组之类的数据项集合相同的简单、可组合的操作来处理异步事件流。它将您从复杂的回调网络中解放出来，从而使您的代码更具可读性，并且不易出现错误。

### Observables 是可组合的

像[ JavaFuture](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html) 的这一类技术，可以直接用于[单项级别的异步执行](https://gist.github.com/4670979)，但当它们多层嵌套来表达多项时，就会变得[很复杂](https://gist.github.com/4671081)。 

[使用 Future 来较好的组成有条件的异步执行流是很困难的](https://gist.github.com/4671081#file-futuresb-java-L163)（或者根本不可能，因为在运行的程序中每个请求的等待时间会有所不同）。当然[可以这样做](http://www.amazon.com/gp/product/0321349601?ie=UTF8&tag=none0b69&linkCode=as2&camp=1789&creative=9325&creativeASIN=0321349601)，但是很快就会变得很复杂（因为很容易出现错误）或者过早的阻塞了 `Future.get()`，从而丢失了异步执行的优点。 

换句话说，ReactiveX Observables 可以 [组合异步数据的流和序列](https://github.com/Netflix/RxJava/wiki/How-To-Use#composition)。

### Observables 很灵活

ReactiveX Observables 不仅支持单个标量的发出（就像 Futures 那样的功能），而且支持值序列甚至无限数据流。`Observable` 是一个能用于任何用例的独立抽象。Observable 具有和 Iterable 相同的所有灵活性和优雅性。 

Observable 异步/push 数据 ”对应“ 同步/pull 相似的 Iterable

| 事件  | Iterable（pull） | Observable（push） |
| --- | --- | --- |
| 检索数据 | `T next()` | `onNext(T)` |
| 发现错误 | throws `Exception` | `onError(Exception)` |
| 完成  | `!hasNext()` | `onCompleted()` |

### Observables 可以用各种风格实现

ReactiveX 并不偏向某些特定的并发或异步源。 Observables 可以用线程池、事件循环、非阻塞 I/O、actors（比如来自 Akka）实现，或者任何适合你需求、风格或专长的方式实现。无论你的底层实现选择阻塞还是非阻塞的方式，客户端代码都会将其与 Observables 的交互视为异步。 

Observable 怎么实现呢？ 

`public Observable<data> getData();` 

- 这在同步在相同线程调用时，还有效吗？
  
- 这在异步不同线程时，还有效吗？
  
- 这在多个线程以任意顺序将数据返回给调用方时，还有效吗？
  
- 这在使用 Actor（或多个 Actor）代替线程池时，还有效吗？
  
- 这在使用带有事件循环的非阻塞 I/O 上进行异步网络访问时，还有效吗？
  
- 这在使用事件循环从回调线程分离出工作线程时，还有效吗？
  

在 Observer 的角度来说，这些都是不无所谓的！ 

更牛的是：借助 ReactiveX，你可以从根本上改变你的 Observable 的底层实现，而你的 Observer 的使用者有无需修改任何代码。

### 回调有自己的缺点

回调通过不允许任何事件阻塞来解决 `Future.get()` 上过早阻塞的问题。回调自然高效，因为在响应代码准备就绪时执行。 

但是和 Futures 一样，尽管回调很容易在单个的异步执行中使用，但是在嵌套使用时，他们会[变得很复杂](https://gist.github.com/4677544)。

### ReactiveX 有多种编程语言的实现

RectiveX 当前已在多种语言中实现，实现的方式遵守了这些语言的习惯用法，并且更多语言的实现将会很快被添加。

## 响应式编程

ReactiveX 提供一组操作符，你可以使用它们来过滤、搜索、转换、结合和组成 Observables，从而允许高效的执行和构建。 

你可以把 Observable 的 ”pull“ 视为 Iterable 的 ”push“。在 Iterable 中，消费者从生产者或线程回调中同步获取值，相比之下，使用 Observable 时，只要值可用，生产者就会推送值给消费者。这种方式更加灵活，因为值既可以同步到达也可以异步到达。 

实例代码展示了如何将相似的高阶函数应用于 Iterable 和 Observable。 

```java
// Iterable
getDataFromLocalMemory()
  .skip(10)
  .take(5)
  .map({ s -> return s + " transformed" })
  .forEach({ println "next" => " + it})

// Observable
getDataFromNetwork()
  .skip(10)
  .take(5)
  .map({ s -> return s + " transformed" })
  .subscribe({ println "onNext => " + it })
```

Observable 类型添加了两个缺失的语义到 [the Gang of Four 总结的观察者模式](http://en.wikipedia.org/wiki/Observer_pattern)，来匹配在 Iterable 类型中可用的那些功能。

1. 生产者向消费者发出一个没有更多可用数据信号的能力（在这种情况下，Iterable 上的 foreach 循环会完成并正常返回；Observable 调用它观察者的 `onCompleted` 方法）
  
2. 生产者向消费者发出一个错误已发生信号的能力（如果在迭代期间发生错误，Iterable 会抛出异常；一个 Observable 会调用他的观察者的 `onError` 方法）
  

有了这些添加，ReactiveX 得以协调 Iterable 类型和 Observable 类型。它们之间唯一的不同是数据流的方向。这是非常重要的，因为现在你能执行的在一个 Iterable 上的任何操作，你都可以在 Observable 上执行。
