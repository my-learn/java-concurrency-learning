线程的消亡不能通过调用stop()命令，而是让run()方法自然结束。** stop()方法是不安全的，已经废弃。**
停止线程推荐的方式：设定一个标志变量，在run()方法中是一个循环，由该标志变量控制循环是继续执行还是跳出；循环跳出，则线程结束。
如代码例子中所示：
```java
class MyThreadClass implements Runnable
{
    private boolean flag = true;
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

