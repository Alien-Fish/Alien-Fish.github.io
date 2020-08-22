---
layout: post
title:  java面试题2-线程、数据库、缓存、协议
date:   2019-11-18 20:02:00 +0800
categories: java面试
tag: interview
---

* content
{:toc}


### 1.讲一下项目

### 2.线程池由哪些组件组成？
线程池的基本思想是一种对象池，在程序启动时就开辟一块内存空间，里面存放了众多(未死亡)的线程，池中线程执行调度由池管理器来处理。当有线程任务时，从池中取一个，执行完成后线程对象归池，这样可以避免反复创建线程对象所带来的性能开销，节省了系统的资源。  

一个线程池包括以下四个基本组成部分：  
(1)、线程池管理器（ThreadPool）：用于创建并管理线程池，包括 创建线程池，销毁线程池，添加新任务；  
(2)、工作线程（WorkThread）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；  
(3)、任务接口（Task）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等；  
(4)、任务队列（taskQueue）：用于存放没有处理的任务。提供一种缓冲机制。  

3.有哪些线程池，分别怎么使用？拒绝策略有哪些？
线程池
1、newCachedThreadPool

作用：创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们，并在需要时使用提供的 ThreadFactory 创建新线程。

特征： 
（1）线程池中数量没有固定，可达到最大值（Interger. MAX_VALUE） 
（2）线程池中的线程可进行缓存重复利用和回收（回收默认时间为1分钟） 
（3）当线程池中，没有可用线程，会重新创建一个线程

创建方式： Executors.newCachedThreadPool();

2、newFixedThreadPool

作用：创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数 nThreads 线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务，则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务（如果需要）。在某个线程被显式地关闭之前，池中的线程将一直存在。

特征： 
（1）线程池中的线程处于一定的量，可以很好的控制线程的并发量 
（2）线程可以重复被使用，在显示关闭之前，都将一直存在 
（3）超出一定量的线程被提交时候需在队列中等待

创建方式： 
（1）Executors.newFixedThreadPool(int nThreads)；//nThreads为线程的数量 
（2）Executors.newFixedThreadPool(int nThreads，ThreadFactory threadFactory)；//nThreads为线程的数量，threadFactory创建线程的工厂方式

3、newScheduleThreadPool

作用： 创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。

特征： 
（1）线程池中具有指定数量的线程，即便是空线程也将保留 
（2）可定时或者延迟执行线程活动

创建方式： 
（1）Executors.newScheduledThreadPool(int corePoolSize)；// corePoolSize线程的个数 
（2）newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)；// corePoolSize线程的个数，threadFactory创建线程的工厂
 
4、newSingleThreadScheduledExecutor

作用： 创建一个单线程执行程序，它可安排在给定延迟后运行命令或者定期地执行。

特征： 
（1）线程池中最多执行1个线程，之后提交的线程活动将会排在队列中以此执行 
（2）可定时或者延迟执行线程活动

创建方式： 
（1）Executors.newSingleThreadScheduledExecutor() ； 
（2）Executors.newSingleThreadScheduledExecutor(ThreadFactory threadFactory) ；//threadFactory创建线程的工厂

### 4.什么时候多线程会发生死锁，写一个例子？
死锁发生的原因：

1、系统资源有限

2、进程或线程推进顺序不恰当

3、资源分配不当

死锁发生的四个条件：

1、互斥条件：一份资源每次只能被一个进程或线程使用（在Java中一般体现为，一个对象锁只能被一个线程持有）

2、请求与保持条件：一个进程或线程在等待请求资源被释放时，不释放已占有资源

3、不可剥夺条件：一个进程或线程已经获得的资源不能被其他进程或线程强行剥夺

4、循环等待条件：形成一种循环等待的场景

下面看代码：

    public class A {
        public synchronized void waitMethod(B b) throws InterruptedException{
            System.out.println(Thread.currentThread().getName() + "：正在执行a的等待方法，持有a的对象锁");
            Thread.sleep(2000L);
            System.out.println(Thread.currentThread().getName() + "：试图调用b的死锁方法，尝试获取b的对象锁");
            b.deadLockMethod();
        }
        
        public synchronized void deadLockMethod(){
            System.out.println(Thread.currentThread().getName() + "：正在执行a的死锁方法，持有a的对象锁");
        }
    }

    public class B {
        public synchronized void waitMethod(A a) throws InterruptedException{
            System.out.println(Thread.currentThread().getName() + "：正在执行b的等待方法，持有b的对象锁");
            Thread.sleep(2000L);
            System.out.println(Thread.currentThread().getName() + "：试图调用a的死锁方法，尝试获取a的对象锁");
            a.deadLockMethod();
        }
        
        public synchronized void deadLockMethod(){
            System.out.println(Thread.currentThread().getName() + "：正在执行B的死锁方法，持有B的对象锁");
        }
    }

    public class TestDeadLock implements Runnable {
     
        A a = new A();
        B b = new B();
        
        public void init() throws InterruptedException{
            Thread.currentThread().setName("主线程");
            a.waitMethod(b);
        }
        
        @Override
        public void run() {
            Thread.currentThread().setName("副线程");
            try {
                b.waitMethod(a);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
        public static void main(String[] args) throws InterruptedException{
            TestDeadLock testDeadLock = new TestDeadLock();
            Thread thread = new Thread(testDeadLock);
            thread.start(); 
            testDeadLock.init();
        }
    }

通过sleep()方法，可以实现主线程持有a的对象锁并请求b的对象锁、副线程持有b的对象锁并请求a的对象锁的场景，即发生死锁。

### 5.Redis的数据结构是什么？ 线程模型说一下？
Redis作为Key-Value型的内存数据库, 其Value有多种类型.

    String
    Hash
    List
    Set
    ZSet

Redis对使用者暴露了五种Value Type, 其底层实现的数据结构有8种, 分别是:

    SDS - simple synamic string - 支持自动动态扩容的字节数组
    list - 平平无奇的链表
    dict - 使用双哈希表实现的, 支持平滑扩容的字典
    zskiplist - 附加了后向指针的跳跃表
    intset - 用于存储整数数值集合的自有结构
    ziplist - 一种实现上类似于TLV, 但比TLV复杂的, 用于存储任意数据的有序序列的数据结构
    quicklist - 一种以ziplist作为结点的双链表结构, 实现的非常苟
    zipmap - 一种用于在小规模场合使用的轻量级字典结构


### 6.讲讲Redis的数据淘汰机制？
配置redis.conf中的maxmemory这个值来开启内存淘汰功能，

至于这个值有什么意义，我们可以通过了解内存淘汰的过程来理解它的意义：

    客户端发起了需要申请更多内存的命令（如set）。

    Redis检查内存使用情况，如果已使用的内存大于maxmemory
    则开始根据用户配置的不同淘汰策略来淘汰内存（key），从而换取一定的内存。

    如果上面都没问题，则这个命令执行成功。

maxmemory为0的时候表示我们对Redis的内存使用没有限制。

Redis提供了下面几种淘汰策略供用户选择，其中默认的策略为noeviction策略：

· noeviction：当内存使用达到阈值的时候，所有引起申请内存的命令会报错。

· allkeys-lru：在主键空间中，优先移除最近未使用的key。

· volatile-lru：在设置了过期时间的键空间中，优先移除最近未使用的key。

· allkeys-random：在主键空间中，随机移除某个key。

· volatile-random：在设置了过期时间的键空间中，随机移除某个key。

· volatile-ttl：在设置了过期时间的键空间中，具有更早过期时间的key优先移除。


### 7.说说Redis的数据一致性问题？


### 8.Redis的分布式怎么做？


### 9.RPC讲一下？使用了什么协议？


### 10.三次握手和四次挥手？如果没有三次握手有问题吗?


### 11.Http请求过程，DNS解析的过程？


### 12.InnoDB支持的四种事务隔离级别名称是什么？有什么区别？说说MySQL隔离级别？事务的特性及慢查询？


### 13.BTree机制说一下？


### 14.说说MySQL常用的优化方法？

