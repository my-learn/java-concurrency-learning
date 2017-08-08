终止一个线程有三种方法
1. 使用退出状态标志。该方式使线程正常退出
2. 使用stop方法强行终止线程。** 这个方法是不安全的，已经废弃不推荐使用 ，**因为`stop`和`suspend`、`resume`、`destroy`一样，也可能发生不可预料的结果
3. 使用interrupt方法中断线程

正确额终止线程的方式应该是让run()方法自然结束，而不是暴力的方式终止，类似强制关机一样，说不定电脑就崩了。
既然stop方法不再使用，那就只能使用退出状态标志的方式了，线程中断将在下节中介绍

使用退出状态标志的方式其实也很简单，即：设定一个标志变量，在run()方法中是一个循环，由该标志变量控制循环是继续执行还是跳出；循环跳出，则线程结束。
如代码例子中所示：
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

