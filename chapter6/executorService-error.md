你可能会用以下命令停止tomcat
```shell
./shutdown.sh
```
然后tomcat服务不能访问了，但是通过ps -ef | grep java发现进程还存在。
产生这种现象的原因：当有非守护线程存在时，jvm不会退出，刚好executorService就有会造成这种现象。

比如创建了ScheduledExecutorService的定时任务，就会导致jvm不会退出，正确的退出容器方式应该是在系统停止前调用shutdown/shutdownNow方法

