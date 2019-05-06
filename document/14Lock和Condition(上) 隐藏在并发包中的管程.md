# 概览

在并发编程的两个核心问题就是: 互斥和同步。

Java SDK包通过Lock和Condition两个接口来实现管程，其中Lock用于解决互斥的问题，Condition用于解决同步的问题

## 再造管程的理由

* 在jdk中的synchronized方法已经实现了管程，但是对于打破死锁释放资源这块无法解决，所以需要再造管程

## 1 Condition

我们知道任何一个Java对象，都拥有一组监视器方法，主要包括wait()、notify()、notifyAll()方法，这些方法与synchronized关键字配合使用可以实现等待/通知机制。类似地，Condition接口也提供类似的Object的监视器的方法，主要包括await()、signal()、signalAll()方法，这些方法与Lock锁配合使用也可以实现等待/通知机制。

相比Object实现的监视器方法，Condition接口的监视器方法具有一些Object所没有的特性：

* Condition接口可以支持多个等待队列，在前面已经提到一个Lock实例可以绑定多个Condition，所以自然可以支持多个等待队列了
* Condition接口支持响应中断，前面已经提到过
* Condition接口支持当前线程释放锁并进入等待状态到将来的某个时间，也就是支持定时功能

Lock接口有三个方法:

```
// 支持中断的 API
void lockInterruptibly() throws InterruptedException;
// 支持超时的 API
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
// 支持非阻塞获取锁的 API
boolean tryLock();
```

## 2 如何保证可见性

Java中的可见性是通过happen-before实现的。Java SDK中的可见性是利用volatile相关的Happens-Before规则,Java SDK中的ReentrantLock内部持有一个volatile的成员变量state，获取锁的时候回读写state的值

## 3 什么是可重入锁

* 线程可重复获取同一把锁，例如在处理一个任务的时候首先需要获取锁，在处理过程中如果还需要获取锁，那么获取肯定能成功，而不是阻塞线程去获取锁
* 可重入函数表示多个线程可以同时调用该函数(也就是线程安全的函数)

## 4 公平锁与非公平锁

在使用ReentranLock时，其有两个构造函数，一个是无参构造函数，一个是有参构造函数，有参构造函数的参数如果为true表示需要构造一个公平锁，反之则是构造一个非公平锁

```
在管程中，如果没有获取到锁的线程会进入到等待队列，对于公平锁来说，如果执行完成，那么唤醒的是等待时间最长的线程。对于非公平锁来说可能随机唤醒一个线程
```

## 5 锁的最佳实践

1. 永远只在更新对象的成员变量时加锁
2. 永远只在访问可变的成员变量时加锁
3. 永远不再调用其他对象的方法时加锁
4. 减少持有锁的时间
5. 减小锁的粒度