# 作用
在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
它初始化时需要传入一个整数，这个整数就是线程要等待完成的操作的数目。

CountDownLatch类的方法
+ CountDownLatch(int count) 构造函数，需要传入一个初始值，即定义必须等待的先行完成的操作的数目
+ await()方法，需要等待其他事件先完成的线程调用，一直阻塞
+ await(long timeout, TimeUnit unit)方法，可以指定等待时间，否则线程被中断
+ countDown()方法，每个被等待的事件在完成的时候调用

当调用countDown()方法后，内部计数器将减1，当计数器到达0的时候，CountDownLatch对象将唤起所有在awati()方法上等待的线程。

