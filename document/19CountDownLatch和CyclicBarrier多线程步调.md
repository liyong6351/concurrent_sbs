# 概览

CountDownLatch和CyclicBarrier主要作用是用于协调不同的线程执行的步调

## 1 示例场景

```bash
用户通过在线商城下单，会生成电子订单，保存在订单库；之后物流会生成派送单给用户发货，派送单保存在派送单库。
为了防止漏派或者重复派送，对账系统每天还会校验石村存在异常订单
```

## 2 第一个版本

```
while(存在未对账订单){
  // 查询未对账订单
  pos = getPOrders(); 1
  // 查询派送单
  dos = getDOrders();  2
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
}
```

## 3 利用并行优化对账系统

上述代码的问题在于订单量和派送单巨大，导致步骤1和2运行缓慢，那需要通过并行优化这个操作

```
while(存在未对账订单){
  // 查询未对账订单
  Thread T1 = new Thread(()->{
    pos = getPOrders();
  });
  T1.start();
  // 查询派送单
  Thread T2 = new Thread(()->{
    dos = getDOrders();
  });
  T2.start();
  // 等待 T1、T2 结束
  T1.join();
  T2.join();
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
}
```

## 4 用CountDownLatch实现线程等待

上述问题在并行这个方面是已经OK了，但是每次都要实例化一个线程，这是比较费性能的，那么使用线程池进行优化  
但是在使用线程池的时候无法获知线程是否运行完成，那么这个时候CountDownLatch就派上用处啦

```
// 创建 2 个线程的线程池
Executor executor = Executors.newFixedThreadPool(2);
while(存在未对账订单){
  // 计数器初始化为 2
  CountDownLatch latch = new CountDownLatch(2);
  // 查询未对账订单
  executor.execute(()-> {
    pos = getPOrders();
    latch.countDown();
  });
  // 查询派送单
  executor.execute(()-> {
    dos = getDOrders();
    latch.countDown();
  });
  
  // 等待两个查询操作结束
  latch.await();
  
  // 执行对账操作
  diff = check(pos, dos);  1
  // 差异写入差异库
  save(diff);   2
}
```

## 5 进一步优化性能

在上述案例中步骤1和步骤2还是串行的，那么可以继续并行起来

```
// 订单队列
Vector<P> pos;
// 派送单队列
Vector<D> dos;
// 执行回调的线程池
Executor executor = Executors.newFixedThreadPool(1);
final CyclicBarrier barrier =
  new CyclicBarrier(2, ()->{
    executor.execute(()->check());
  });
  
void check(){
  P p = pos.remove(0);
  D d = dos.remove(0);
  // 执行对账操作
  diff = check(p, d);
  // 差异写入差异库
  save(diff);
}
  
void checkAll(){
  // 循环查询订单库
  Thread T1 = new Thread(()->{
    while(存在未对账订单){
      // 查询订单库
      pos.add(getPOrders());
      // 等待
      barrier.await();
    }
  });
  T1.start();  
  // 循环查询运单库
  Thread T2 = new Thread(()->{
    while(存在未对账订单){
      // 查询运单库
      dos.add(getDOrders());
      // 等待
      barrier.await();
    }
  });
  T2.start();
}
```

## 6 总结

CountDownLatch和CyclicBarrier是Java并发包中提供的两个非常易用的线程同步工具类，这两个工具类方法的区别在于:

* CountDownLatch主要用来解决一个线程等待多个线程的场景，就像旅行团长需要等到多有的游客到齐才能去下一个景点一样
* CyclicBarrier是一组线程之间互相等待
* CountDownLatch的计数器不能循环，但是CyclicBarrier的计数器是可以循环利用的
* CountDownLatch没有回调函数，但是CyclicBarrier可设置回调函数