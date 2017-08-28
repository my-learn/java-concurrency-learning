在前面介绍了基本的同步机制

* synchronized
* Lock

本章将介绍使用更高级的同步机制来实现多线程间的同步

* 信号量（Semaphore）：对资源资源的并发访问控制
* CountDownLatch：用于等待多个并发事件的完成
* CyclicBarrier：功能跟CountDownLatch一样，但支持barrier重用
* Phaser：用于等待多个阶段并发事件的完成
。功能跟前面的CountDownLatch、CyclicBarrier功能一样，但是支持动态调整任务的数量。
* Exchanger:用户线程间数据交换




