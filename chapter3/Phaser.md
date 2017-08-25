`Phaser`是jdk 1.7引入的线程同步辅助类，`Phaser`代表一个可重复使用的同步屏障(barrier)，功能类似`CyclicBarrier`和`CountDownLatch`，但是比他们更强大灵活。
<br />
首先解释下这个英文单词的意思，Phase翻译成中文就是阶段的意思。Phaser类机制是在每一个阶段结束的位置对线程进行同步，当所有的线程都该阶段，才能进入下一个阶段。 这个机制刚好说明Phaser侧重在“重用”二字上。

当我们有并发任务并且需要分解成几步执行的时候，这种机制就非常适合。 

相对于CyclicBarrier CountDownLatch来说，他们都只能在构造时指定成员参与数，而Phaser可以动态的增减参与数。

Phase提供的方法
* arriveAndAwaitAdvance()：等待参与者达到指定数量，才开始运行下面的代码
* arrive()：这个方法通知phase对象一个参与者已经完成了当前阶段，但是它不应该等待其他参与者都完成当前阶段，必须小心使用这个方法，因为它不会与其他线程同步。
* awaitAdvance(int phase)：如果传入的阶段参数与当前阶段一致，这个方法会将当前线程至于休眠，直到这个阶段的所有参与者都运行完成。如果传入的阶段参数与当前阶段不一致，这个方法立即返回。
* awaitAdvanceInterruptibly(int phaser):这个方法跟awaitAdvance(int phase)一样，不同处是：该访问将会响应线程中断。会抛出interruptedException异常。
* register():将一个新的参与者注册到phase中，这个新的参与者将被当成没有执完本阶段的线程。
* bulkRegister(int parties)：将指定数目的参与者注册到phaser中，所有这些新的参与者都将被当成没有执行完本阶段的线程。
* arriveAndDeregister():减少参与者 。告知phaser对应的线程已经完成了当前阶段，并它不会参与到下一阶段的操作中，也就是从phaser的数量中减少
* forceTermination()：强制终止。当一个phaser没有参与者的时候，它就处于终止状态，使用forceTermination()方法来强制phaser进入终止状态，不管是否存在未注册的参与线程，当一个线程出现错误时，强制终止phaser是很有意义的。



