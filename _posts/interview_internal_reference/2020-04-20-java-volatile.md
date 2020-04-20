---
layout: post
title: "Java并发机制底层实现原理之Volatile"
subtitle: 'Java并发机制底层实现原理之Volatile'
author: "Jian Shen"
header-style: text
tags:
  - volatile
  - java
---

如果一个字段被声明成volatile，Java内存模型保证所有线程看到这个变量的值是一致的。

### 1. 实现原理

有volatile修饰的变量进行写操作时会多出带有lock前缀的汇编指令，该指令在多核处理器中引发两件事情：

 - 将当前处理器缓存行的数据写回系统内存。通过总线锁定或缓存行锁定。
 - 这个写回内存的操作会使其他CPU中缓存了该内存地址的数据无效。其他处理器通过嗅探总线上传播过来的数据监测自己工作内存中缓存是否过期。过期，修改的时候则从主存中重新获取。
 
### 2. 内存语义层次理解

#### 2.1 volatile的特性

volatile变量自身具有以下特性：
   
  - 可见性：对一个volatile变量的读，总是能看到任意线程对这个volatile变量最后的写入
  - 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性，如
  ```
    volatile long v = 0L;
    v = 200L; //具有原子性
    v++; // 不具有原子性
  ```
ps: JDK5开始，JSR-133内存模型只允许64位long/double型的写操作拆分为两个32位的写操作来执行，任意的读操作在JSR-133中都必须具有原子性。

#### 2.2 volatile的写-读内存语义

 - 写内存语义：当写一个volatile变量时，JMM会把线程对应的本地内存的共享变量刷新到主内存
 - 读内存语义：当读一个volatile变量时，JMM会把线程对应的本地内存置于无效，线程接下来将从主内存读取共享变量
 
#### 2.3 重排序与volatile内存语义的实现

##### 重排序简介

为了提高性能，处理器与编译器会对指令做重排序，分为3种类型：
 - 1.编译器优化的重排序，不改变单线程程序语义，重新安排语句执行顺序
 - 2.指令级并行的重排序，若不存在数据依赖性，处理器可以改变语句对机器指令的执行顺序
 - 3.内存系统的重排序，处理器采用缓存和读、写缓冲区，使得加载与存储看上去是乱序执行
 
1 属于编译器重排序；
2、3属于处理器重排序。
为了实现volatile内存语义，需要JMM分别限制这两大类型的重排序。 

##### volatile内存语义具体实现

 - volatile写：前面插入StoreStore屏障（禁止该屏障前的普通写与volatile写重排序），后面插入StoreLoad屏障（防止volatile写与该屏障后的可能有的volatile读/写重排序）
 - volatile读：后面首先插入一个LoadLoad（禁止该屏障后所有的普通读操作与volatile读重排序），然后插入LoadStore屏障（禁止该屏障后所有的普通写操作与volatile读重排序）
 
### 3. 使用优化

JDK7并发包中新增了一个队列集合类LinkedTransferQueue，他在使用volatile变量时，用一种追加字节的方式来优化队列的入队与出队的性能。使用追加到64字节的方式填满同一个高速缓冲行，避免头结点与尾结点加载到同一个缓存行，使头、尾节点在修改时都不会相互锁定。

###  4. 实际使用场景-状态变量标记

一般业务场景中用于volatile static Boolean flag = true

在双重检查锁定中起到防止重排序作用

```java
public class SafeDoubleCheckedLocking{
    private volatile static Instance instance;
    
    public static Instance getInstance(){
        if(instance == null ){ // 第一次检查
            synchronized (SafeDoubleCheckedLocking.class){ // 加锁
                if(instance == null){ // 第二次检查
                    //该步骤有三小步：
                    // （1）分配对象的内存空间 
                    // （2）初始化对象 
                    // （3）设置instance指向刚分配的内存地址
                    // volatile 就是防止（2）与（3）重排序
                    instance = new Instance();
                }
            }
        }
        return instance;
    }
}
```  

ps: CPU术语定义

 - 内存屏障：是一组处理器指令，用于实现对内存操作的顺序控制
 - 缓存行：CPU高速缓存中可以分配的最小存储单位，处理器填写缓存行时会加载整个缓存行
 - 数据依赖性：如果两个操作同时访问一个变量，其中一个操作为写操作，则这两个操作存在数据依赖性