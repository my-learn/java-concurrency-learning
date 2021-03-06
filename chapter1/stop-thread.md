在java中，并未提供一种安全的抢占式方法来停止线程，只能通过一些协作方式来结束线程。

终止一个线程有四种方法
1. 使用退出状态标志。该方式使线程正常退出
2. 使用stop方法强行终止线程。** 这个方法是不安全的，已经废弃不推荐使用 ，** 这些方法`stop`、`stop(Throwable)`、`suspend`、`resume`、`destroy`一样，都是废弃的，因为他们都可能发生不可预料的结果
3. 使用interrupt中断机制中断线程
4. FutureTask和Executor框架带有管理线程的功能，包括取消线程

正确终止线程的方式应该是让run()方法自然结束，而不是暴力的方式终止，类似强制关机一样，说不定电脑就崩了。
既然stop方法不再使用，那就只能使用退出状态标志的方式了，线程中断将在下节中介绍

使用退出状态标志的方式其实也很简单，即：设定一个标志变量，在run()方法中是一个循环，由该标志变量控制循环是继续执行还是跳出；循环跳出，则线程结束。
但是这种实现还是有弊端，如果while循环中存在阻塞代码，那么任务永远也不可能去检查标志位，导致线程永远不可能结束。

实现代码如下：
```java
class MyThreadClass implements Runnable
{
    private volatile boolean flag = true;
    @Override
    public void run()
    {
        while (flag)
        {
            System.out.println("Do something.");
        }
    }
    public void stopRunning()
    {
        flag = false;
    }
}
```

# 为什么stop方法被废弃
参考[Why Are Thread.stop, Thread.suspend, Thread.resume and Runtime.runFinalizersOnExit Deprecated?](http://docs.oracle.com/javase/1.5.0/docs/guide/misc/threadPrimitiveDeprecation.html)


