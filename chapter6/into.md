Fork/Join框架是用来解决能够通过分治技术将一个问题拆分成子问题来解决的技术框架。

ForkJoinPool继承自ExecutorService，提供了多个重载的submit方法，计算任务，其中一个submit方法接收一个ForkJoinTask类型参数。
它默认的并行线程数通过使用`Runtime.getRuntime().availableProcessors()`获取的。

ForkJoinTask继承自Future的，它有两个实现类RecursiveTask、RecursiveAction，他们都是抽象类，需要子类实现compute()方法

RecursiveTask:用于任务没有返回结果的场景
RecursiveAction:用于任务有返回结果的场景




