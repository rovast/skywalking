# 服务自动探针

服务自动探指的是基于语言本身的一些自动注入机制。这种类型的探针，一般使用的是语言各自的特性，一般都是基于虚拟机的语言。
## 什么是自动？

很多人都是从这样了解到这类探针的“不需要修改任何一行代码”，SkyWalking 以前也是把这个写到了 README 文档里。
但怎么说呢，这句话是既对又不对。
对于终端用户来说，答案是：对。他们确实不需要修改代码，至少大部分情况下是这样的。
但是同时，这个答案又是：不对。因为你的运行时代码需要修改。
就底层而言，你需要基于虚拟机的接口来修改代码，比如说在 Java 语言装哪个，你需要通过 `javaagent premain` 来修改 Java class。

同样，我们说自动的探针一般是基于虚拟机语言的，但同样的，你也可以自己在编译阶段构建相关动作。

## 局限性？

自动的语言探针听起来还是很不错的，但有是否有些局限性？答案当然是：**是**，主要体现在：

- **多数情况下需要通过进程来传播数据**。
很多高级语言被用来构建业务系统，比如：Java 和 .NET。
一般而言，在一个请求中，这些业务逻辑代码运行在同一个线程中，一般是基于 thread id 来进行传播，这类技术栈用来保证上下文安全。

- **只对 frameworks 或 libraries 生效**。
由于代码的修改是由探针完成的，也就是说这些修改是插件开发者来完成的。所以，支持的场景有限，如下所示：[SkyWalking Java 探针支持列表](../setup/service-agent/java-agent/Supported-list.md).

- **跨线程支持有限**。
正如我们之前说到的进程传播一样，在一个请求中，大部分的代码是运行在独立的线程中，尤其是业务代码。
但是在有些场景中，代码是运行在不同的线程里，比如：任务派发、任务池或者批量的进程。
还有一些语言提供了类似携程的特性，比如 `Goroutine`，在这种情况下，开发者可以运行异步进程，而且这些都是很常见的。
在这种情况下，自动探针就会遇到一些问题。

至此，我们已经解开了自动探针的神秘面纱，简而言之，自动语言探针是因为有探针的开发者在背后默默支撑（而不是说啥都不用修改）。

## 接下来

如果你想知道手动探针，查阅 [手动探针 SDK](manual-sdk.md) 章节。