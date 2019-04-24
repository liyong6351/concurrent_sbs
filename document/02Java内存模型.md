<!-- TOC -->

- [概览](#%E6%A6%82%E8%A7%88)
  - [1 volatile](#1-volatile)
  - [2 Happens-Before规则](#2-happens-before%E8%A7%84%E5%88%99)
    - [2.1 程序的顺序性规则](#21-%E7%A8%8B%E5%BA%8F%E7%9A%84%E9%A1%BA%E5%BA%8F%E6%80%A7%E8%A7%84%E5%88%99)
    - [2.2 volatile变量规则](#22-volatile%E5%8F%98%E9%87%8F%E8%A7%84%E5%88%99)
    - [2.3 传递性](#23-%E4%BC%A0%E9%80%92%E6%80%A7)
    - [2.4 管程中锁的规则](#24-%E7%AE%A1%E7%A8%8B%E4%B8%AD%E9%94%81%E7%9A%84%E8%A7%84%E5%88%99)
    - [2.5 线程 start() 规则](#25-%E7%BA%BF%E7%A8%8B-start-%E8%A7%84%E5%88%99)
    - [2.6 线程join规则](#26-%E7%BA%BF%E7%A8%8Bjoin%E8%A7%84%E5%88%99)
- [3 final](#3-final)
  - [优化特点](#%E4%BC%98%E5%8C%96%E7%89%B9%E7%82%B9)

<!-- /TOC -->

# 概览

Java内存模型规范了JVM如何提供按需禁用缓存和编译优化的方法。具体方法包括: volatile、synchronize和final三个关键字以及六项Happens-Before规则

## 1 volatile

volatile目的就是禁用CPU缓存，告诉编译器，每次对这个变量的读写都不能使用CPU缓存，必须从内存中读取或者写入

## 2 Happens-Before规则

* happens-before表达的是前一个操作的结果对后续操作的可见性，也就是说Happens-Before约束了编译器优化之后一定要遵守Happens-Before原则

### 2.1 程序的顺序性规则

* 在一个线程中，按照程序的顺序操作

### 2.2 volatile变量规则

* 对一个volatile变量的写操作对后续操作的读可见

### 2.3 传递性

* A Happens-Before B，B Happens-Before C，那么A Happens-Before C

### 2.4 管程中锁的规则

* 对一个锁的解锁Happens-Before于后续对这个锁的加锁
* 管程是一种通用的同步原语，在Java中指的就是synchronize

### 2.5 线程 start() 规则

* 主线程A启动子线程B后，子线程B能够看到主线程在启动子线程B前的操作

### 2.6 线程join规则

* 这条是关于线程等待的。意思是主线程A等待子线程B的完成，组线程能够看到子线程的操作，指的是共享变量的操作

# 3 final

final 关键字告诉编译器，这个变量生而不变，可以随意优化

## 优化特点

* final对变量的重排进行了约束，对于正确的构造函数就没有“逸出”