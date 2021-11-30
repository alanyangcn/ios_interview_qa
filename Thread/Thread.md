## 多线程有哪几种？你更倾向于哪一种？

* NSThread
* Cocoa NSOperation （使用NSOperation和* NSOperationQueue）
* GCD （Grand Central Dispatch）

### 1.NSThread:(两种创建方式)

```
[NSThread detachNewThreadSelector:@selector(doSomething:) toTarget:self withObject:nil];

NSThread *myThread = [[NSThread alloc] initWithTarget:self selector:@selector(doSomething:) object:nil];

[myThread start];
```

优点：NSThread 比其他两个轻量级。 缺点：需要自己管理线程的生命周期，线程同步，线程同步时对数据的加锁会有一定的系统开销。

### 2.Cocoa Operation

```
NSOperationQueue*oprationQueue= [[NSOperationQueuealloc] init];

oprationQueueaddOperationWithBlock:^{

//这个block语句块在子线程中执行

}
```

优点：不需要关心线程管理，数据同步的事情。 Cocoa Operation 相关的类是 NSOperation ，NSOperationQueue。NSOperation是个抽象类，使用它必须用它的子类，可以实现它或者使用它定义好的两个子类：NSInvocationOperation 和 NSBlockOperation。创建NSOperation子类的对象，把对象添加到NSOperationQueue队列里执行，我们会把我们的执行操作放在NSOperation中main函数中。

3.GCD Grand Central Dispatch (GCD)是Apple开发的一个多核编程的解决方法，GCD是一个替代诸如NSThread, NSOperationQueue, NSInvocationOperation等技术的很高效和强大的技术。它让程序平行排队的特定任务，根据可用的处理资源，安排他们在任何可用的处理器核心上执行任务，一个任务可以是一个函数(function)或者是一个block。 dispatch queue分为下面三种： private dispatch queues，同时只执行一个任务，通常用于同步访问特定的资源或数据。 global dispatch queue，可以并发地执行多个任务，但是执行完成的顺序是随机的。 Main dispatch queue 它是在应用程序主线程上执行任务的。
