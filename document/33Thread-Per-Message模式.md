# 概览

Thread-Per-Message 设计模式其实就是一种委托实现的解决方案  
例如在Tomcat处理的示例中，HttpServer只能响应主线程的操作  
那么如果HttpServer也进行业务逻辑处理的话会导致HttpServer  
彻底变成串行操作，那么这将导致性能急剧下降  
Thread-Per-Message模式是一种分工的模式，实现起来也很简单

## 1 用Thread实现Thread-Per-Message模式

Thread-Per-Message模式的最经典应用场景就是网络服务器里服务端  
的实现。下例是使用Thread-Per-Message实现的Echo程序

```
final ServerSocketChannel ssc = 
  ServerSocketChannel.open().bind(
    new InetSocketAddress(8080));
// 处理请求    
try {
  while (true) {
    // 接收请求
    SocketChannel sc = ssc.accept();
    // 每个请求都创建一个线程
    new Thread(()->{
      try {
        // 读 Socket
        ByteBuffer rb = ByteBuffer
          .allocateDirect(1024);
        sc.read(rb);
        // 模拟处理请求
        Thread.sleep(2000);
        // 写 Socket
        ByteBuffer wb = 
          (ByteBuffer)rb.flip();
        sc.write(wb);
        // 关闭 Socket
        sc.close();
      }catch(Exception e){
        throw new UncheckedIOException(e);
      }
    }).start();
  }
} finally {
  ssc.close();
}
```

以上的代码从服务端实现方案来看是不具备可行性的，因为  
线程创建是一个重量级的操作，Java中线程创建直接对应着  
服务器上线程的创建，这样的有点是稳定可靠，但是弊端就是  
线程的操作性能消耗太大。  
业界有另外一种方案，叫做轻量级线程，这个在GO或Lua上叫做  
协程，创建的成本和创建本地类的消耗类似，但是当前Java是不支持  
但是Java也在做这方面的改善，比如当前OpenJDK中的loom项目  
就是针对这个情况的

## 2 使用Fiber实现Thread-Per-Message模式

```
final ServerSocketChannel ssc = 
  ServerSocketChannel.open().bind(
    new InetSocketAddress(8080));
// 处理请求
try{
  while (true) {
    // 接收请求
    final SocketChannel sc = 
      serverSocketChannel.accept();
    Fiber.schedule(()->{
      try {
        // 读 Socket
        ByteBuffer rb = ByteBuffer
          .allocateDirect(1024);
        sc.read(rb);
        // 模拟处理请求
        LockSupport.parkNanos(2000*1000000);
        // 写 Socket
        ByteBuffer wb = 
          (ByteBuffer)rb.flip()
        sc.write(wb);
        // 关闭 Socket
        sc.close();
      } catch(Exception e){
        throw new UncheckedIOException(e);
      }
    });
  }//while
}finally{
  ssc.close();
}
```