# 概览

Semaphore也就是信号量，是由大名鼎鼎的迪杰斯特拉于1965年提出来的。  
在管程出来之前，信号量都是并发编程领域的终结者

## 1 信号量模型

信号量模型还是很简单的，可以概括为三部分: 计数器、等待队列、三个方法 init() down()和up()

* 信号量伪代码如下:

```
class Semaphore{
  // 计数器
  int count;
  // 等待队列
  Queue queue;
  // 初始化操作
  Semaphore(int c){
    this.count=c;
  }
  // 
  void down(){
    this.count--;
    if(this.count<0){
      // 将当前线程插入等待队列
      // 阻塞当前线程
    }
  }
  void up(){
    this.count++;
    if(this.count<=0) {
      // 移除等待队列中的某个线程 T
      // 唤醒线程 T
    }
  }
}
```

## 2 Java中的信号量

Java中的信号量对应的类为Semaphore,对应的方法为acquire()和release()方法

* 信号量demo

```
static final count;
static final Semaphore s = new Semaphore(1);

static void addOne(){
    s.acquire();
    try{
    //dosomebusiness
    }finally{
        s.release();
    }
}
```