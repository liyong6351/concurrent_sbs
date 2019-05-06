<!-- TOC -->

- [概览](#概览)
    - [StampedLock支持的三种锁模式](#stampedlock支持的三种锁模式)
    - [进一步理解乐观锁](#进一步理解乐观锁)
    - [StampedLock使用注意事项](#stampedlock使用注意事项)

<!-- /TOC -->

# 概览

同样的在读多写少的场景中，除了ReadWriteLock之外，还有StampedLock可以实现，并且性能比ReadWriteLock的性能还要好

## StampedLock支持的三种锁模式

ReadWriteLock支持两种模式: 读锁和写锁

StampedLock支持三种模式: 写锁、悲观锁和乐观锁。其中写锁和悲观锁和ReadWriteLock中的写锁和读锁类似，允许多个线程获取到悲观读锁，但是只有一个线程获取写锁，读锁和写锁是互斥的。不同的是，StampedLock里的写锁和悲观读锁加锁成功之后，都会返回一个stamp；然后解锁的时候需要传递这个stamp，示例如下:

```
final StampedLock sl = 
  new StampedLock();
  
// 获取 / 释放悲观读锁示意代码
long stamp = sl.readLock();
try {
  // 省略业务相关代码
} finally {
  sl.unlockRead(stamp);
}

// 获取 / 释放写锁示意代码
long stamp = sl.writeLock();
try {
  // 省略业务相关代码
} finally {
  sl.unlockWrite(stamp);
}
```

StampedLock的性能之所以比ReadwriteLock好，其关键是StampedLock支持乐观读的方式。ReadWriteLock支持多个线程同时读，但是多个线程同时读的时候，所有的写操作都会被阻塞。StampedLock提供的乐观读，是允许一个线程获取写锁的，也就是说不是所有的写操作都会被阻塞。

示例:

```
class Point {
  private int x, y;
  final StampedLock sl = 
    new StampedLock();
  // 计算到原点的距离  
  int distanceFromOrigin() {
    // 乐观读
    long stamp = 
      sl.tryOptimisticRead();
    // 读入局部变量，
    // 读的过程数据可能被修改
    int curX = x, curY = y;
    // 判断执行读操作期间，
    // 是否存在写操作，如果存在，
    // 则 sl.validate 返回 false
    if (!sl.validate(stamp)){
      // 升级为悲观读锁
      stamp = sl.readLock();
      try {
        curX = x;
        curY = y;
      } finally {
        // 释放悲观读锁
        sl.unlockRead(stamp);
      }
    }
    return Math.sqrt(
      curX * curX + curY * curY);
  }
}
```

## 进一步理解乐观锁

* 数据库中的乐观锁

<pre>
通过version，每次更新的时候version+1。这个version其实就类似于StampedLock中的stamp
</pre>

## StampedLock使用注意事项

StampedLock的功能仅仅是ReadWriteLock的子集。在使用的时候需要注意以下事项:
1. StampedLock没有reen的标识，那么他们不支持可重入
2. Stamped的悲观读锁和写锁都不支持条件变量
3. 使用StampedLock千万不要调用中断操作，否则可能导致CPU飙升到100%。如果需要支持中断操作，需要使用悲观读锁和写锁