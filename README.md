《java并发编程学习手册》本书主要介绍java多线程编程基础，以及java并发库的使用，涉及到的知识点尽量做到简单、全面。同时，不会将非相关的类容涵盖进来而占用篇幅

---

**目录**

* [关于本书](README.md)
* [前言 java并发框架简介](java_concurrent_intro.md)
* [第1章 线程管理](chapter1.md)
  * [1.1 进程和线程](chapter1/process-thread.md)
  * [1.2 java 中创建进程](chapter1/1-2.md)
  * [1.3 java 中创建线程](chapter1/1-3.md)
  * [1.4 停止线程](chapter1/stop-thread.md)
  * [1.5 线程中断机制](chapter1/interrupt-thread.md)
  * [1.6 线程中不可控异常的处理](chapter1/uncaught-exception-in-thread.md)
  * [1.6 守护线程](chapter1/daemon-thread.md)
  * [1.5 Thread类详解](chapter1/1-4.md)
  * [1.6 线程的生命周期](chapter1/1-5.md)
  * [1.7 线程组](chapter1/ThreadGroup.md)
  * [1.8 多线程中访问成员变量与局部变量](chapter1/1-7.md)
  * [1.9 线程间的通信-wait及notify方法](chapter1/wait-notify.md)
  * [1.10 ThreadLocal类](chapter1/ThreadLocal.md)
* [第2章 多线程同步](chapter2.md)
  * [2.1 简介](chapter2/intro.md)
  * [2.2 锁的相关概念介绍](chapter2/Lock-concepts.md)
  * [2.3 synchronized](chapter2/synchronized.md)
  * [2.4 Lock](chapter2/Lock.md)
  * [2.5 可重入ReentrantLock](chapter2/ReentrantLock.md)
  * [2.6 读写锁ReadWriteLock](chapter2/ReadWriteLock.md)
  * [2.7 修改锁的公平性](chapter2/lock-fair.md)
  * [2.8 Lock和synchronized的选择](chapter2/Lock-vs-synchronized.md)
  * [2.9 LockSupport](chapter2/LockSupport.md)
  * [2.10 在锁中使用多条件\(Condition\)](chapter2/Condition.md)
* [第3章 线程同步辅助类](chapter3.md)
  * [3.1 简介](chapter3/intro.md)
  * [3.2 信号量Semaphore](chapter3/Semaphore.md)
  * [3.3 CountDownLatch](chapter3/CountDownLatch.md)
  * [3.4 CyclicBarrier](chapter3/CyclicBarrier.md)
  * [3.5 Phaser](chapter3/Phaser.md)
  * [3.6 Exchanger](chapter3/Exchanger.md)
* [第4章 Java Executor并发框架](chapter4.md)
  * [4.1 简介](chapter4/4-1.md)
  * [4.2 Executor接口](chapter4/4-2.md)
  * [4.3 ExecutorService接口](chapter4/4-3.md)
  * [4.4 ScheduledExecutorService接口](chapter4/4-4.md)
  * [4.5 Executors静态工厂方法](chapter4/4-5.md)
  * [4.6 ThreadPoolExecutor ](chapter4/4-6.md)
  * [4.7 可以返回结果的线程](chapter4/4-7.md)
  * [4.8 CompletionService](chapter4/4-8.md)
  * [4.9 Executors得到当前活动的线程数](chapter4/4-9.md)
* [第5章 并发集合](chapter5.md)
  * [5.1 简介](chapter5/5-1.md)
  * [5.2 阻塞队列](chapter5/5-2.md)
  * [5.3 同步阻塞队列SynChronousQueue](chapter5/5-3.md)
  * [5.4 非阻塞队列](chapter5/5-4.md)
  * [5.5 ConcurrentMap](chapter5/5-5.md)
  * [5.6 CopyOnWrite机制实现](chapter5/5-6.md)
  * [5.7 支持排序的并发集合](chapter5/5-7.md)
  * [5.8 LinkedTransferQueue](chapter5/LinkedTransferQueue.md)
* [第6章 Fork/Join框架](chapter6.md)
  * [6.1 简介](chapter6/into.md)
  * [6.2 RecursiveAction](chapter6/RecursiveAction.md)
  * [6.3 RecursiveTask](chapter6/RecursiveTask.md)
* [第7章 原子变量](chapter7.md)
  * [7.1 简介](chapter7/into.md)
  * [7.2 AtomicInteger](chapter7/AtomicInteger.md)
  * [7.3 AtomicReference](chapter7/AtomicReference.md)
  * [7.4 原子数组](chapter7/AtomicIntegerArray.md)
* [第8章 实用篇](chapter8.md)
  * [8.1 时间工具类TimeUnit](chapter8/TimeUnit.md)
  * [8.2 volatile](chapter8/volatile.md)

* [第9章 深入篇](chapter9.md)
  * [9.1 指令重排 and 多级存储](chapter9/memory-reordering.md)
  * [9.2 CAS](chapter9/cas.md)
  * [9.3 AbstractQueuedSynchronizer](chapter9/AbstractQueuedSynchronizer.md)
  * [9.4 Vector真的是安全的吗？](chapter9/Vector.md)
  * [9.5 Servlet的多线程同步问题](chapter9/Servlet.md)
  * [9.6 executorService停止线程的陷阱](chapter9/executorService-error.md)
* [说明](notice.md)


















