
Fork/Join框架思想就是充分利用多核CPU的优势把计算任务拆分成多个子任务，再把多个子任务的计算结果合并，提高cpu利用率，有点类似MapReduce思想。



Fork/Join提供的接口很简洁，核心只有三个类/接口，你不需要太多时间关心并行时线程的通信，死锁问题，线程同步

Fork/Join核心由以下几个类接口组成
* `ForkJoinPool`:继承自ExecutorService，提供了多个重载的submit方法计算任务，其中一个submit方法接收一个ForkJoinTask类型参数。
它默认的并行线程数通过使用`Runtime.getRuntime().availableProcessors()`获取的。<br />
ForkJoinPool几个重要的方法
`void execute(ForkJoinTask<?> task)`: 以同步的方式执行一个任务
`void execute(Runnable task)`:同上
`submit(...)`：多个重载方法，执行一个任务
`shutdown()`:
`boolean awaitTermination(long timeout, TimeUnit unit)`:阻塞指定的时间，等待任务完成

* `ForkJoinTask`:这个类是ForkJoinPool中执行的任务得基类，继承自Future的，它有两个实现类RecursiveTask、RecursiveAction，他们都是抽象类，需要子类实现compute()方法。一个可以Fork/Join的任务必须实现这个两个类之一

* RecursiveAction:用于任务没有返回结果的场景
* RecursiveTask:用于任务有返回结果的场景

ForkJoinTask几个重要的方法
`boolean isCompletedNormally()`:任务是否完成并且没有抛出异常没有被取消
`boolean cancle()`：取消任务
`ForkJoinTask<V> fork()`
`V join()`
`void invokeAll(ForkJoinTask<?>... tasks)`
`boolean isDone()`
`invokeAll(...)`：多个重载方法，用来执行一个主任务所创建的多个子任务。这是一个同步调用，这个任务将等待子任务完成，然后继续执行（也可能是结束）


