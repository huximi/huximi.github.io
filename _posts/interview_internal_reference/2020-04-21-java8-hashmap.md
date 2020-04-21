---
layout: post
title: "Java 8系列之重新认识HashMap"
subtitle: 'HashMap原理及分析'
author: "Jian Shen"
header-style: text
tags:
  - HashMap
  - Java
---

### 摘要

HashMap是Java程序员使用频率最高的用于映射（键值对）处理的数据类型。随着JDK(Java Development Kit)版本的更新，JDK1.8对HashMap底层的实现进行了优化，例如引入红黑树的数据结构和扩容的优化等。本文结合JDK1.7和JDK1.8的区别，深入探讨HashMap的结构实现和功能原理。

### 简介

Java为数据结构中的映射定义了一个接口java.util.Map,此接口主要有四个常用的实现类，分别是HashMap、Hashtable、LinkedHashMap、TreeMap，类继承关系如下图所示：

<img src = "/img/java/map.png" />

下面针对各个实现类的特点做一些说明：

 - (1) HashMap: 它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。HashMap最多只允许一条记录的键为null,允许多条记录的值为null。HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。
 - (2) Hashtable Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写Hashtable，并发性不如ConcurrentHashMap，因为ConcurrentHashMap引入了分段锁。Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。
 - (3) LinkedHashMap: LinkedHashMap是HashMap的一个子类，保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的，也可以在构造时带参数，按照访问次序排序。
 - (4) TreeMap：TreeMap实现SortedMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。如果使用排序的映射，建议使用TreeMap。在使用TreeMap时，key必须实现Comparable接口或者在构造TreeMap传入自定义的Comparator，否则会在运行时抛出java.lang.ClassCastException类型的异常。
 
对于上述四种Map类型的类，要求映射中的key是不可变对象。不可变对象是该对象在创建后它的哈希值不会被改变。如果对象的哈希值发生变化，Map对象很可能就定位不到映射的位置了。

通过上面的比较，我们知道了HashMap是Java的Map家庭中一个普通成员，鉴于它可以满足大多数场景的使用条件，所以是使用频度最高的一个。下文我们主要结合原码，从存储结构、常用方法分析、扩容以及安全性等方面深入讲解HashMap的工作原理。

### 内部实现

搞清楚HashMap，首先需要知道HashMap是什么，即它的存储结构-字段；其次弄明白它能干什么，即它的功能实现-方法。下面我们针对这两个方面详细展开讲解。

### 存储结构-字段

从结构实现来讲，HashMap是数组+链表+红黑树（JDK1.8增加了红黑树部分）实现的，如下所示：

 <img src = "/img/java/hashMap.png" />

这里需要讲明白两个问题：数据底层具体存储的是什么？这样的存储方式有什么优点呢？

 - (1) 从源码可知，HashMap类中有一个非常重要的字段，就是Node[] table,即哈希桶数组，明显它是一个Node的数组。我们来看Node[JDK1.8]是何物。
 
     ```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash; //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next; //链表的下一个node
        
        Node(int hash,K key,V value,Node<K,V> next) { /*...*/ }
        public final K getKey() {/*...*/}
        public final V getValue() {/*...*/}
        public final String toString() {/*...*/}
        public final int hashCode() {/*...*/}
        public final V setValue(V newValue) {/*...*/}
        public final boolean equals(Object o) {/*...*/}  
    }
    ```

   `Node`是`HashMap`的一个内部类，实现了`Map.Entry`接口，本质是一个映射（键值对）。上图中的每个黑色圆点就是一个Node对象。

 - (2) HashMap就是使用哈希表来存储的。哈希表为解决冲突，可以采用开放地址法和链地址法等来解决问题，Java中HashMap采用了链地址法。链地址法，简单来说，就是数组加链表的结合。在每个数组元素上都有一个链表结构，当数据被Hash后，得到数组下标，把数据放在对应下标元素的链表上。例如程序执行下面代码：
    ```
    map.put("姓名","小米");
    ```
    
   系统将调用`"姓名"`这个key的`hashCode()`方法得到其`hashCode`值（该方法适用于每个Java对象），然后再通过Hash算法的后两步运算（高位运算和取模运算，下文有介绍）来定位该键值对的存储位置，有时两个key会定位到相同的位置，表示发生了Hash碰撞。当然Hash算法计算结果越分散均匀，Hash碰撞的概率就越小，map的存取效率就会越高。
  
    如果哈希桶数组很大，即使较差的Hash算法也会比较分散，如果哈希桶数组很小，即使好的Hash算法也会出现较多碰撞，所以就需要在空间成本和时间成本之间权衡，其实就是在根据实际情况确定哈希桶数组的大小，并在此基础上设计好的Hash算法减少Hash碰撞。那么通过什么方式来控制map使得Hash碰撞的概率又小，哈希桶数组（Node[] table)占用空间又少呢？答案就是好的Hash算法和扩容机制。
  
    在理解Hash和扩容流程之前，我们得先了解下HashMap的几个字段。从HashMap的默认构造函数源码可知，构造函数就是对下面几个字段进行初始化，源码如下：
      ```
      int threshold; //所能容纳的key-value对极限
      final float loadFactor;  //负载因子
      int modCount;
      int size;
      ```
      
    首先，`Node[] table`的初始化长度`length`(默认值是16)，`loadFactor`为负载因子（默认值是0.75），`threshold`是`HashMap`所能容纳的最大数据量的`Node`(键值对)个数。`threshold = length * loadFactor` 。也就是说,在数组定义好长度之后，负载因子越大，所能容纳的键值对个数越多。
    
    结合负载因子的定义公式可知，threshold就是在此loadFactor和length(数组长度)对应下允许的最大元素数目，超过这个数目就重新resize(扩容)，扩容后的HashMap容量是之前容量的两倍。默认的负载因子0.75是对空间和时间效率的一个平衡选择，建议大家不要修改，除非在时间和空间比较特殊的情况下，如果内存空间很多而又对时间效率要求很高，可以降低负载因子loadFactor的值; 相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1.
    
     size这个字段其实很好理解，就是HashMap中实际存在的键值对数量。注意和table的长度length、容纳最大键值对数量threshold的区别。而modCount字段主要用来记录HashMap内部结构发生变化的次数，主要用于迭代的快速失败。强调一点，内部结构发生变化指的是结构发生变化，例如put新键值对，但是某个key对应的value值被覆盖不属于结构变化。
     
     在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计，常规的设计是把桶的大小设计为素数。相对来说素数导致冲突的概率要小于合数，具体证明可以参考[为什么一般hashtable的桶数会取一个素数](https://blog.csdn.net/liuqiyao_01/article/details/14475159) ,Hashtable初始化桶大小为11，就是桶大小设计为素数的应用（Hashtable扩容后不能保证还是素数）。HashMap采用这种非常规设计，主要是为了在取模和扩容时做优化，同时为了减少冲突，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。
     
     这里存在一个问题，即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，则会严重影响HashMap的性能，于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能，其中会用到红黑树的插入、删除、查找等算法。本文不再对红黑树展开讨论，想了解更多红黑树数据结构的工作原理可以参考[教你初步了解红黑树](https://blog.csdn.net/v_july_v/article/details/6105630) 
     
### 功能实现-方法


     
  
 
  
  
  






