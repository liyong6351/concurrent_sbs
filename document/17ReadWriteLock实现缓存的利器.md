<!-- TOC -->

- [概览](#概览)
    - [1 什么是读写锁](#1-什么是读写锁)
    - [2 快速实现一个缓存](#2-快速实现一个缓存)
    - [3 读写锁的升级与降级](#3-读写锁的升级与降级)
    - [4 总结](#4-总结)

<!-- /TOC -->

# 概览

在读多写少的应用场景下，ReadWriteLock可以提供很大的帮助，并且性能很好

## 1 什么是读写锁

读写锁遵循以下三条基本原则:

* 允许多个想成同时读共享变量
* 只允许一个线程写共享变量
* 如果一个写线程正在执行写操作，此时禁止度线程读共享变量

读写锁与互斥锁的一个重要区别就是读写锁允许多个线程同时读共享变量，而互斥锁是不允许的，这是读写锁在读多写少场景下性能优于互斥锁的关键

## 2 快速实现一个缓存

```
class Cache<K,V> {
  final Map<K, V> m = new HashMap<>();
  final ReadWriteLock rwl = new ReentrantReadWriteLock();
  // 读锁
  final Lock r = rwl.readLock();
  // 写锁
  final Lock w = rwl.writeLock();
  // 读缓存
  V get(K key) {
    r.lock();
    try { return m.get(key); }
    finally { r.unlock(); }
  }
  // 写缓存
  V put(String key, Data v) {
    w.lock();
    try { return m.put(key, v); }
    finally { w.unlock(); }
  }
}
```

* 缓存数据的初始化可以采用一次性全量load，也可以采用增量load，这要看缓存数据的大小。在增量load的方式就是懒加载

按需加载的示例:

```
class Cache<K,V> {
  final Map<K, V> m =
    new HashMap<>();
  final ReadWriteLock rwl = 
    new ReentrantReadWriteLock();
  final Lock r = rwl.readLock();
  final Lock w = rwl.writeLock();
 
  V get(K key) {
    V v = null;
    // 读缓存
    r.lock();         ①
    try {
      v = m.get(key); ②
    } finally{
      r.unlock();     ③
    }
    // 缓存中存在，返回
    if(v != null) {   ④
      return v;
    }  
    // 缓存中不存在，查询数据库
    w.lock();         ⑤
    try {
      // 再次验证
      // 其他线程可能已经查询过数据库
      v = m.get(key); ⑥
      if(v == null){  ⑦
        // 查询数据库
        v= 省略代码无数
        m.put(key, v);
      }
    } finally{
      w.unlock();
    }
    return v; 
  }
}
```

## 3 读写锁的升级与降级

* 在获取了读锁之后再获取写锁，叫做锁的升级，当前是不允许的
* 在获取了写锁之后获取读锁，叫做锁的降级，当前是可以的

```
class CachedData {
  Object data;
  volatile boolean cacheValid;
  final ReadWriteLock rwl =
    new ReentrantReadWriteLock();
  // 读锁  
  final Lock r = rwl.readLock();
  // 写锁
  final Lock w = rwl.writeLock();
  
  void processCachedData() {
    // 获取读锁
    r.lock();
    if (!cacheValid) {
      // 释放读锁，因为不允许读锁的升级
      r.unlock();
      // 获取写锁
      w.lock();
      try {
        // 再次检查状态  
        if (!cacheValid) {
          data = ...
          cacheValid = true;
        }
        // 释放写锁前，降级为读锁
        // 降级是可以的
        r.lock(); ①
      } finally {
        // 释放写锁
        w.unlock(); 
      }
    }
    // 此处仍然持有读锁
    try {use(data);} 
    finally {r.unlock();}
  }
}
```

## 4 总结

读写锁类似于ReenTrantLock，也支持公平模式和非公平模式。但是只有写锁支持条件变量，读锁不支持条件变量，会抛错