我们先来了解一下进程和线程的概念
# 进程的概念

很简单的道理嘛，你看电影的时候，又想聊qq，又想迅雷下载，这相当于有3个任务在同事进行。对电脑而言，就是3个应用程序，用进程来对应一个程序，每个进程对应一定的内存地址空间，并且只能使用它自己的内存空间，各个进程间互不干扰。并且进程保存了程序每个时刻的运行状态，这样就为进程切换提供了可能。当进程暂时时，它会保存当前进程的状态（比如进程标识、进程的使用的资源等），在下一次重新切换回来时，便根据之前保存的状态进行恢复，然后继续执行。
** 从宏观上看有多个任务在同时执行，但在微观上看，在一个时间段内，只有一个任务能抢占CPU资源(针对单核CPU) **

一个进程可以包含一个或多个线程。
一个进程至少要包含一个线程(主线程)

# 线程的概念
一般一个程序不可能只干一件事吧。一个任务会有多个子任务，比如聊qq的时候，我发消息的同时，也在接收消息，也可能同时在传输文件。
针对这中情况，出现了线程的概念，任务对应进程，子任务对应线程，即一个进程有多个线程。
+ ** 虽然一个进程包括多个线程，但是这些线程是共享进程占有的资源和地址空间的。进程是操作系统进行资源分配的基本单位，而线程是操作系统进行调度的基本单位。** 
+ ** 线程本身不能运行，它只能用于程序中。** 
