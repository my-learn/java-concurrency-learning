Fork/Join框架是用来解决能够通过分治技术将一个问题拆分成子问题来解决的技术框架。

ForkJoinPool继承自ExecutorService，使用起来还是非常容易的，它默认的并行线程数通过使用`Runtime.getRuntime().availableProcessors()`获取的。

