
Fork/Join框架思想就是充分利用多核CPU的优势把计算任务拆分成多个子任务，再把多个子任务的计算结果合并，提高cpu利用率，有点类似MapReduce思想。一个可以被拆分的任务要符合可以被递归拆分的要求

Fork/Join框架和执行器框架(Executor Framework)非常类似，利用线程池分发执行任务，他们主要的区别在于Fork/Join框架使用了工作窃取算法(Work-Stealing Algorithm)。所谓工作窃取，指的是对那些处理完自身任务的线程，会从其它线程窃取任务执行。

Fork/Join提供的接口很简洁，核心只有三个类/接口，你不需要太多时间关心并行时线程的通信，死锁问题，线程同步

Fork/Join核心由以下几个类接口组成
`ForkJoinPool`:继承自ExecutorService，提供了多个重载的submit方法计算任务，其中一个submit方法接收一个ForkJoinTask类型参数。
它默认的并行线程数通过使用`Runtime.getRuntime().availableProcessors()`获取的。<br />

ForkJoinPool几个重要的方法
* `void execute(ForkJoinTask<?> task)`: 以异步的方式执行一个任务
* `void execute(Runnable task)`:同上
* `T invoke(ForkJoinTask<T> task)`:同步方式执行一个任务
* `submit(...)`：多个重载方法，执行一个任务
* `shutdown()`:
* `boolean awaitTermination(long timeout, TimeUnit unit)`:阻塞指定的时间，等待任务完成

`ForkJoinTask`:这个类是ForkJoinPool中执行的任务得基类，继承自Future的，它有两个实现类RecursiveTask、RecursiveAction，他们都是抽象类，需要子类实现compute()方法。一个可以Fork/Join的任务必须实现这个两个类之一

* RecursiveAction:用于任务没有返回结果的场景
* RecursiveTask:用于任务有返回结果的场景

虽然ForkJoinPool是用来执行ForkJoinTask对象的，但是从submit等多个方法的重载形式看得出来，它也可以执行Runnable和Callable。当然，也可以使用ForkJoinTask类的adapt()方法将Runnable和Callable对象转换为一个ForkJoinTask对象，然后再去执行。

ForkJoinTask几个重要的方法
* `boolean isCompletedNormally()`:任务是否完成并且没有抛出异常没有被取消
* `ForkJoinTask<V> fork()`、`V join()`：异步的方式执行任务，
* `invokeAll(...)`：多个重载方法，用来执行一个主任务所创建的多个子任务。这是一个同步调用，这个任务将等待子任务完成，然后继续执行（也可能是结束）
* `void complete(V value)`:该方法也可以用来结束任务的执行并返回结果，它接收一个RecursiveTask类型的泛型参数
* `boolean isDone()`
* `boolean cancle()`：取消任务，不能取消已经被执行的任务，ForkJoinPool也未提供任务取消线程池中正在运行或等待运行的任务的方法。


基本使用方法
推荐的使用代码结构
```plain
if (problem size > default size){
  tasks  = divide(task); // 将任务一分为二
  execute(tasks);
}else{
  直接执行该任务
}
```

下面章节将介绍基本使用



