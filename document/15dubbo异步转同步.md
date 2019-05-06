# 概览

在dubbo中使用到了异步转同步的操作，让我们一起来看看源代码

## 1 总结

在dubbo使用rpc调用之后，会执行到dubboInvoker的doInvoke方法，这块实际上使用的是Future编程模式，即在请求发出去之后，通过Callable将线程阻塞掉，然后监听是否由返回数据，如果超时那么就直接抛错，如果未超时，则future返回数据就直接返回掉了

## 2 使用锁的情况

```
private final Lock lock = new ReentrantLock();
private final Condition done = lock.newCondition();

private void doReceived(Response res) {
    lock.lock();
    try {
        response = res;
        if (done != null) {
            done.signal();
        }
    } finally {
        lock.unlock();
    }
    if (callback != null) {
        invokeCallback(callback);
    }
}
```