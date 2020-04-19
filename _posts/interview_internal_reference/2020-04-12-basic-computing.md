---
layout: post
title: "计算机基本知识"
subtitle: '计算机基本知识'
author: "GuYang"
header-img: "img/post-bg-2018.jpg"
tags:    
    - 面试题     
---

####  关于并行计算的一些基础开放问题。

◼ 如何定义并计算，请分别阐述分布式内存到共享内存模式行编程的区别和实现（例子代码）？

◼ 请使用 MPI 和 OpenMP 分别实现 N 个处理器对 M 个变量的求和？

◼ 请说明 SIMD 指令在循环中使用的权限？向量化优化有哪些手段？

◼ 请用 Amdahl 定律说明什么是并行效率以及并行算法的扩展性？并说明扩展性的性能指标和限制因素，最后请说明在共享内存计算机中，共享内存的限制？OpenMP 是怎样实现共享内存编程环境的？MPI 阻塞和非阻塞读写的区别？


（简要答案，但必须触及，可以展开）
◼ 同时执行多个/算法/逻辑操作/内存访问/IO，相互独立同时运行，分三个层次：进程级，多个节点分布式内存通过MPI通信并行；线程级，共享内存的多路机器，通过OpenMP实现多线程并行；指令集：通过SIM指令实现单指令多数据。。。。举例吧啦吧啦。

◼ MPI代码，，，OpenMP代码，分别写出来 M个元素，N个处理器的累加，后者注意private 参数。

◼ SIMD在循环中的应用，限制在于 SIMD指令处理的每一个数组的长度，cache line利用，内部循环间的依赖和条件调用等。

◼ 向量化，主要看SSE和AVX指令占比率，通过编译器优化...... 在loop代码中使用。

◼ 性能和计算规模随处理器增加的变化曲线，实测HPL和峰值HPL比率，能用用Amdahl定律表达Tpar(N) = (an + (1-a)n/N )t + C (n,N), 能够讲明白串行部分对整个并行的天花板效应，扩展性能够解释清楚算法的扩展性=并行效率随处理器数目的变化关系，画出来。

◼ 共享内存计算机OpenMP对变量的限制描述，EREW，CREW，ERCW，CRCW等区别，NUMA概念，如何保持coherent等。

◼ 写出OpenMP和MPI的核心函数，回答问题即可。

#### 什么是数据库



数据库可以看成是一个电子化的文件柜,用户可以对文件中的数据运行新增、检索、更新、删除等操作。数据库是一个所有集合的容器，在文件系统中每一个数据库都有一个相关的物理文件。

#### 什么是集合



集合就是一组 MongoDB 文档。它相当于关系型数据库（RDBMS）中的表这种概念。集合位于单独的一个数据库中。一个集合内的多个文档可以有多个不同的字段。一般来说，集合中的文档都有着相同或相关的目的。

#### 什么是文档



文档由一组key value组成。文档是动态模式,这意味着同一集合里的文档不需要有相同的字段和结构。在关系型数据库中table中的每一条记录相当于MongoDB中的一个文档。

#### 什么是非关系型数据库



非关系型数据库是对不同于传统关系型数据库的统称。非关系型数据库的显著特点是不使用SQL作为查询语言，数据存储不需要特定的表格模式。由于简单的设计和非常好的性能所以被用于大数据和Web Apps等

#### 非关系型数据库有哪些类型



* Key-Value 存储 Eg:Amazon S3

* 图表 Eg:Neo4J

* 文档存储 Eg:MongoDB

 * 基于列存储 Eg:Cassandra
 
 
#### 如何实现两金额数据相加（最多小数点两位）？

其实问题并不难，就是考察候选人对 JavaScript 数据运算上的认知以及考虑问题的缜密程度，有很多坑，可以用在笔试题，如果用在面试，回答过程中还可以随机加入有很多计算机基础的延伸。

回到这个问题，由于直接浮点相与加会失精，所以要转整数；（可以插入问遇到过吗？是否可以举个例子？）。

转整数是第一个坑，虽然只有两位可以通过乘以100转整数，但由于乘以一百和除以一百都会出现浮点数的运算，所以也会失精，还是要通过字符串来转；（可以插入问字符串转整数有几种方式？）字符串转整是第二个坑，因为最后要对齐计算，如果没考虑周全先toFixed(2)，对于只有一位小数点数据进入计算就会错误；转整数后的计算是个加分点，很多同学往往就是直接算了，如果可以考虑大数计算的场景，恭喜同学进入隐藏关卡，这就会涉及如何有效循环、遍历、算法复杂度的问题。
 
 ##### **问题**：关于 epoll 和 select 的区别，哪些说法是正确的？（多选）
 A. epoll 和 select 都是 I/O 多路复用的技术，都可以实现同时监听多个 I/O 事件的状态。
 
 B. epoll 相比 select 效率更高，主要是基于其操作系统支持的I/O事件通知机制，而 select 是基于轮询机制。
 
 C. epoll 支持水平触发和边沿触发两种模式。
 
 D. select 能并行支持 I/O 比较小，且无法修改。
 
 #
 A，B，C
 
 
 **【延伸】那在高并发的访问下，epoll使用那一种触发方式要高效些？当使用边缘触发的时候要注意些什么东西？**

#### 请评估一下程序的执行结果？
```
public class SynchronousQueueQuiz {
    public static void main(String[] args) throws Exception {
        BlockingQueue<Integer> queue = new
        SynchronousQueue<>();
        System. out .print(queue.offer(1) + " ");
        System. out .print(queue.offer(2) + " ");
        System. out .print(queue.offer(3) + " ");
        System. out .print(queue.take() + " ");
        System. out .println(queue.size());
    }
}

```
A. true true true 1 3

B. true true true (阻塞)

C. false false false null 0

D. false false false (阻塞)


D

#### 最大频率栈。
实现 FreqStack，模拟类似栈的数据结构的操作的一个类。FreqStack 有两个函数： push(int x)，将整数 x 推入栈中。pop()，它移除并返回栈中出现最频繁的元素。如果最频繁的元素不只一个，则移除并返回最接近栈顶的元素。
◼ 示例：
push [5,7,5,7,4,5]
pop() -> 返回 5，因为 5 是出现频率最高的。 栈变成
[5,7,5,7,4]。
pop() -> 返回 7，因为 5 和 7 都是频率最高的，但 7 最接近栈
顶。 栈变成 [5,7,5,4]。
pop() -> 返回 5 。 栈变成 [5,7,4]。
pop() -> 返回 4 。 栈变成 [5,7]。



令 freq 作为 x 的出现次数的映射 Map。

此外 maxfreq，即栈中任意元素的当前最大频率，因为我们必须弹出频率最高的元素。

当前主要的问题就变成了：在具有相同的（最大）频率的元素中，怎么判断那个元素是最新的？我们可以使用栈来查询这一信息：靠近栈顶的元素总是相对更新一些。

为此，我们令 group 作为从频率到具有该频率的元素的映射。到目前，我们已经实现了 FreqStack 的所有必要的组件。

算法：

实际上，作为实现层面上的一点细节，如果 x 的频率为 f，那么我们将获取在所有 group[i] (i <= f) 中的 x,而不仅仅是栈顶的那个。这是因为每个 group[i] 都会存储与第 i 个 x 副本相关的信息。

最后，我们仅仅需要如上所述维持 freq，group，以及 maxfreq。

**参考代码***：
```
class FreqStack {
    Map<Integer, Integer> freq;
    Map<Integer, Stack<Integer>> group;
    int maxfreq;

    public FreqStack() {
        freq = new HashMap();
        group = new HashMap();
        maxfreq = 0;
    }
    
    public void push(int x) {
        int f = freq.getOrDefault(x, 0) + 1;
        freq.put(x, f);
        if (f > maxfreq) maxfreq = f;
        group.computeIfAbsent(f, z-> new Stack()).push(x);
    }
    
    public int pop() {
        int x = group.get(maxfreq).pop();
        freq.put(x, freq.get(x) - 1);
        if (group.get(maxfreq).size() == 0)
        maxfreq--;
        return x;
    }
}
```

#### 从 innodb 的索引结构分析，为什么索引的 key 长度不能太长？


key 太长会导致一个页当中能够存放的 key 的数目变少，间接导致索引树的页数目变多，索引层次增加，从而影响整体查询变更的效率。


#### 给定一个链表，删除链表的倒数第 N 个节点，并且返回链表的头结点。

◼ 示例：
给定一个链表: 1->2->3->4->5, 和 n = 2.
当删除了倒数第二个节点后，链表变为 1->2->3->5.
说明：
给定的 n 保证是有效的。
要求：
只允许对链表进行一次遍历。



我们可以使用两个指针而不是一个指针。第一个指针从列表的开头向前移动 n+1n+1 步，而第二个指针将从列表的开头出发。现在，这两个指针被 nn 个结点分开。我们通过同时移动两个指针向前来保持这个恒定的间隔，直到第一个指针到达最后一个结点。此时第二个指针将指向从最后一个结点数起的第 n 个结点。我们重新链接第二个指针所引用的结点的 next 指针指向该结点的下下个结点。

**参考代码**：

```
public ListNode removeNthFromEnd(ListNode head, int n)
{
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode first = dummy;
    ListNode second = dummy;
    // Advances first pointer so that the gap between first
    and second is n nodes apart
    for (int i = 1; i <= n + 1; i++) {
        first = first.next;
    }
    // Move first to the end, maintaining the gap
    while (first != null) {
        first = first.next;
        second = second.next;
    }
    second.next = second.next.next;
    return dummy.next;
}
```

**复杂度分析：**
* 时间复杂度：O(L)，该算法对含有 L 个结点的列表进行了一次遍历。因此时间复杂度为 O(L)。

* 空间复杂度：O(1)，我们只用了常量级的额外空间。

#### 现有一批邮件需要发送给订阅顾客，且有一个集群（集群的节点数不定，会动态扩容缩容）来负责具体的邮件发送任务，如何让系统尽快地完成发送？请详述技术方案！

A. 借助消息中间件，通过发布者订阅者模式来进行任务分配

B. master-slave 部署，由 master 来分配任务

C. 不借助任何中间件，且所有节点均等。通过数据库的 update-returning，从而实现节点之间任务的互斥

#### 请解释下为什么鹿晗发布恋情的时候，微博系统会崩溃，如何解决？

A. 获取微博通过 pull 方式还是 push 方式

B. 发布微博的频率要远小于阅读微博

C. 流量明星的发微博，和普通博主要区分对待，比如在 sharding的时候，也要考虑这个因素

#### 有一批气象观测站，现需要获取这些站点的观测数据，并存储到 Hive 中。但是气象局只提供了 api 查询，每次只能查询单个观测点。那么如果能够方便快速地获取到所有的观测点的数据？

A. 通过 shell 或 python 等调用 api，结果先暂存本地，最后将本地文件上传到 Hive 中。

B. 通过 datax 的 httpReader 和 hdfsWriter 插件，从而获取所需的数据。

C. 比较理想的回答，是在计算引擎的 UDF 中调用查询 api，执行UDF 的查询结果存储到对应的表中。一方面，不需要同步任务的导出导入；另一方面，计算引擎的分布式框架天生提供了分布式、容错、并发等特性。

#### 一颗现代处理器，每秒大概可以执行多少条简单的MOV指令，有哪些主要的影响因素？


**及格：**
每执行一条mov指令需要消耗1个时钟周期，所以每秒执行的mov指令和CPU主频相关。

**加分：**
在CPU微架构上，要考虑数据预取，乱序执行，多发射，内存stall(前端stall和后端stall)等诸多因素，因此除了cpu主频外，还和流水线上的效率(IPC)强相关，比较复杂的一个问题。

#### Memcache与Redis的区别都有哪些？


1)、存储方式

Memecache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。

Redis有部份存在硬盘上，这样能保证数据的持久性。

2)、数据支持类型

Memcache对数据类型支持相对简单。

Redis有复杂的数据类型。

3)、使用底层模型不同

它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。

Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。

4），value大小

redis最大可以达到1GB，而memcache只有1MB


















 
 