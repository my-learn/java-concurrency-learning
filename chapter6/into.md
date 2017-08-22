Fork/Join框架思想就是充分利用多核CPU把计算任务拆分成多个子任务，提高cpu利用率，有点类似MapReduce思想。

Fork/Join提供的接口很简洁，核心只有三个类/接口，你不需要太多时间关心并行时线程的通信，死锁问题，线程同步

ForkJoinPool继承自ExecutorService，提供了多个重载的submit方法，计算任务，其中一个submit方法接收一个ForkJoinTask类型参数。
它默认的并行线程数通过使用`Runtime.getRuntime().availableProcessors()`获取的。

ForkJoinTask继承自Future的，它有两个实现类RecursiveTask、RecursiveAction，他们都是抽象类，需要子类实现compute()方法

RecursiveTask:用于任务没有返回结果的场景
RecursiveAction:用于任务有返回结果的场景




