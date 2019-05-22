# 概览

Balking模式本质上是一个多线程的if条件判断，比如  
我们实现编辑器的自动保存，那么是否改变是一个共享  
变量，如何保证isChange这个共享变量的安全呢?第一个  
方法就是对于其修改都使用synchronized关键字，但是  
这种方式的效率极低。

## 1 Balking模式的经典实现  

```
Balking模式本质上是一种规范化解决“多线程版本的if”的  
方案，如果使用Balking设计模式的话，简化后的代码如下:  
简单说就是把共享变量使用单一的方法来维护，这样的好处  
就是讲并发处理的逻辑和业务逻辑分开
```

<pre>
boolean changed=false;
// 自动存盘操作
void autoSave(){
  synchronized(this){
    if (!changed) {
      return;
    }
    changed = false;
  }
  // 执行存盘操作
  // 省略且实现
  this.execSave();
}
// 编辑操作
void edit(){
  // 省略编辑逻辑
  ......
  change();
}
// 改变状态
void change(){
  synchronized(this){
    changed = true;
  }
}
</pre>

## 2 Balking模式和单例模式的讨论

在实现单例模式的时候可以采取以下方案:  

```
class Singleton{
  private static
    Singleton singleton;
  // 构造方法私有化  
  private Singleton(){}
  // 获取实例（单例）
  public synchronized static 
  Singleton getInstance(){
    if(singleton == null){
      singleton=new Singleton();
    }
    return singleton;
  }
}
```

但是上述方案的效率很低，因为synchronized同步的操作范围太大  
使用Balking的模式写法如下:

<pre>
class Singleton{
  private static volatile 
    Singleton singleton;
  // 构造方法私有化  
  private Singleton() {}
  // 获取实例（单例）
  public static Singleton 
  getInstance() {
    // 第一次检查
    if(singleton==null){
      synchronize{Singleton.class){
        // 获取锁后二次检查
        if(singleton==null){
          singleton=new Singleton();
        }
      }
    }
    return singleton;
  }
}
</pre>

这样的操作效率就会高出很多

## 3 总结

```
Balking模式和Guarded Suspension模式要解决的问题都是一样的，  
即在多线程情况下对于if的判断。不同之处在于Guarded Suspension模式  
需要等待直到条件满足，线程被执行。而Balking模式不会等待而直接失败
```