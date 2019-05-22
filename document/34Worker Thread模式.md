# 概览

最经典的Worker Thread模式其实就是线程池模式

## 1线程池介绍

```
线程池有很多优点，能避免创建、销毁线程，同时能够限制创建线程的上限等等。
```

## 2正确的创建线程池

```
Java的线程池既能够避免无限制的创建线程导致OOM，也能够避免无限制  
的接收任务导致OOM，所以建议如下:
1.在创建线程池的时候使用有界队列来接收任务  
2.在创建线程池时，清晰的知名拒绝策略
3.给与线程和业务相关的命名
```

```
ExecutorService es = new ThreadPoolExecutor(
  50, 500,
  60L, TimeUnit.SECONDS,
  // 注意要创建有界队列
  new LinkedBlockingQueue<Runnable>(2000),
  // 建议根据业务需求实现 ThreadFactory
  r->{
    return new Thread(r, "echo-"+ r.hashCode());
  },
  // 建议根据业务需求实现 RejectedExecutionHandler
  new ThreadPoolExecutor.CallerRunsPolicy());
```

## 3避免线程死锁

```
提交到线程池执行的任务尽量是独立的，不然一定要考虑锁竞争的关系导致死锁
```