# 概览

本节讲解线程池相关知识

线程的创建需要调用操作系统的API，然后操作系统要为线程分配一系列的资源，所以线程是一个重量级的对象，应该避免频繁创建和销毁

## 1 线程池是一种 生产者-消费者 模式

```
线程池的使用方式生产者，线程池本身是消费者
```

## 2 如何使用Java中的线程池

线程池最核心的是ThreadPoolExecutor，其最复杂的构造函数是包括七个参数:

* corePoolSize: 线程池保有的最小线程数
* maximumPoolSize: 线程池创建的最大线程数
* keepAliveTime & unit: 线程最长的空闲时间
* workQueue: 工作队列
* threadFactory: 自定义如何创建线程
* handler: 定义拒绝策略，包括以下四种: 1. CallRunsPolicy  2. AbortExecutionExceptions 3. DiscardPolicy 4. DiscardOldestPolicy

## 使用线程池需要注意什么

考虑到使用ThreadPoolExecutor的构造函数是在有些复杂，所以Java并发包里提供了一个线程池的静态工厂类Executors，利用Executors可以快速创建线程池，不过现在大厂基本都不建议使用Executors了。

因为Executors一般都使用无界的LinkedBlockingQueue，高负载情况下容易导致OOM，OOM会导致整个应用都无法使用，所以建议使用有界队列

同时在使用线程池时候，需要考虑拒绝策略，这经常和降级一起处理