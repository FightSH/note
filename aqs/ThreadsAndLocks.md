

# Threads and Locks

[深入分析 java 8 编程语言规范](https://www.javadoop.com/post/Threads-And-Locks-md)

内容主要来自上面的文章，这里只是总结

## 前言

线程由Thread类表示，每个线程都和Thread的一个类实例关联。用户创建线程的方式是创建Thread的类实例

## 17.1同步（synchronization）

>  synchronize 的锁是基于 Java 对象的监视器 monitor，所以任何对象都可以用来做锁。
>
> **对 Class 对象加锁、对对象加锁，它们之间不构成同步**。synchronized 作用于静态方法时是对 **Class 对象**加锁，作用于实例方法时是对实例加锁。

Java中提供的多线程之间通信的机制，最基本的就是同步（synchronization），使用监视器（monitor）实现(Java中每个对象都关联了一个监视器)。在同一时间，只有一个线程可以拿到对象上的监视器锁。并且监视器锁是可重入的。

## 17.2等待集合和唤醒（Wait Sets and Notification）



### 17.2.1等待（wait）



在线程 t 中对对象 m 调用 m.wait()方法，n 代表加锁编号，同时还没有相匹配的解锁操作，下面的情况之一会发生

- 如果 n = 0，即线程t没有持有对象 m 的锁，那么会抛出 IllegalMonitorStateException 异常。

	> 如果没有获取到监视器锁，wait 方法是会抛异常的，而且注意这个异常是IllegalMonitorStateException 异常。

- 如果线程 t 调用的是 m.wait(millisecs) 或m.wait(millisecs, nanosecs)，形参 millisecs 不能为负数，nanosecs 取值应为 [0, 999999]，否则会抛出 IllegalArgumentException 异常。

- 如果线程 t 被中断( **interrupt** )，此时中断状态为 true，则 wait 方法将抛出 InterruptedException 异常，并将中断状态设置为 false。

- 否则，下面的情况会发生

  1. 线程 t 会加入到对象 m 的**等待集合**中，执行 **加锁编号 n 对应的解锁操作**

  2. 线程 t 不会执行任何进一步的指令，直到它从 m 的等待集合中移出（也就是等待唤醒）。在发生以下操作的时候，线程 t 会从 m 的等待集合中移出，然后在之后的某个时间点恢复，并继续执行之后的指令。

  	- 在 m上执行了 notify 操作，**而且线程 t 被选中**从等待集合中移除。

  	- 在 m 上执行了 notifyAll 操作，那么线程 t 会从等待集合中移除。

  	- 线程 t 发生了 interrupt 操作。

  	- 如果线程 t 是调用 wait(millisecs) 或者 wait(millisecs, nanosecs) 方法进入等待集合的，那么过了millisecs 毫秒或者 (millisecs*1000000+nanosecs) 纳秒后，线程 t 也会从等待集合中移出。

  	- JVM 的“假唤醒”，虽然这是不鼓励的，但是这种操作是被允许的，这样 JVM 能实现将线程从等待集合中移出，而不必等待具体的移出指令。

  		注意，良好的 Java 编码习惯是，只在循环中使用 wait 方法，这个循环等待某些条件来退出循环。

  		> ~~~java
  		>  synchronized(m) {
  		>      while(!canExit) {
  		>        m.wait(10); // 等待10ms; 当然中断也是常用的
  		>        canExit = something();  // 判断是否可以退出循环
  		>      }
  		>  }
  		>  // 2 个知识点：
  		>  // 1. 必须先获取到对象上的监视器锁
  		>  // 2. wait 有可能被假唤醒
  		> ~~~

  		每个线程在一系列 **可能导致它从等待集合中移出的事件** 中必须决定一个顺序

  		

  3. 线程 t 执行编号为 n 的加锁操作

  4. 如果线程 t 在 2 的时候由于中断而从 m 的等待集合中移出，那么它的中断状态会重置为 false，同时 wait 方法会抛出 InterruptedException 异常。

### 17.2.2通知（Notification）

notify和notifyAll

我们在线程 t 中对对象 m 调用 m.notify() 或 m.notifyAll() 方法，n 代表加锁编号，同时对应的解锁操作没有执行，则下面的其中之一会发生：

- 如果 n 等于 0，抛出 IllegalMonitorStateException 异常，因为线程 t 还没有获取到对象 m 上的锁。

	> 这一点很关键，只有获取到了对象上的监视器锁的线程才可以正常调用 notify，前面我们也说过，调用 wait 方法的时候也要先获取锁

- 如果 n 大于 0，而且这是一个 notify 操作，如果 m 的等待集合不为空，那么等待集合中的线程 u 被选中从等待集合中移出。

	对于哪个线程会被选中而被移出，虚拟机没有提供任何保证，从等待集合中将线程 u 移出，可以让线程 u 得以恢复。注意，恢复之后的线程 u 如果对 m 进行加锁操作将不会成功，直到线程 t 完全释放锁之后。

	> 因为线程 t 这个时候还持有 m 的锁。这个知识点在 17.2.4 节我还会重点说。这里记住，被 notify 的线程在唤醒后是需要重新获取监视器锁的。

- 如果 n 大于 0，而且这是一个 notifyAll 操作，那么等待集合中的所有线程都将从等待集合中移出，然后恢复。

	注意，这些线程恢复后，只有一个线程可以锁住监视器。



#### 17.2.3 中断（Interruptions）

中断发生于 Thread.interrupt 方法的调用。

令线程 t 调用线程 u 上的方法 u.interrupt()，其中 t 和 u 可以是同一个线程，这个操作会将 u 的中断状态设置为 true。

> 顺便说说中断状态吧，初学者肯定以为 thread.interrupt() 方法是用来暂停线程的，主要是和它对应中文翻译的“中断”有关。中断在并发中是常用的手段，请大家一定好好掌握。可以将中断理解为线程的状态，它的特殊之处在于设置了中断状态为 true 后，这几个方法会感知到：
>
> 1. wait(), wait(long), wait(long, int), join(), join(long), join(long, int), sleep(long), sleep(long, int)
>
> 	这些方法都有一个共同之处，方法签名上都有`throws InterruptedException`，这个就是用来响应中断状态修改的。
>
> 2. 如果线程阻塞在 InterruptibleChannel 类的 IO 操作中，那么这个 channel 会被关闭。
>
> 3. 如果线程阻塞在一个 Selector 中，那么 select 方法会立即返回。
>
> 如果线程阻塞在以上3种情况中，那么当线程感知到中断状态后（此线程的 interrupt() 方法被调用），会将中断状态**重新设置为 false**，然后执行相应的操作（通常就是跳到 catch 异常处）。
>
> 如果不是以上3种情况，那么，线程的 interrupt() 方法被调用，会将线程的中断状态设置为 true。
>
> 当然，除了这几个方法，我知道的是 LockSupport 中的 park 方法也能自动感知到线程被中断，当然，它不会重置中断状态为 false。我们说了，只有上面的几种情况会在感知到中断后先重置中断状态为 false，然后再继续执行。

另外，如果有一个对象 m，而且线程 u 此时在 m 的等待集合中，那么 u 将会从 m 的等待集合中移出。这会让 u 从 wait 操作中恢复过来，u 此时需要获取 m 的监视器锁，获取完锁以后，发现线程 u 处于中断状态，此时会抛出 InterruptedException 异常。

> 这里的流程：t 设置 u 的中断状态 => u 线程恢复 => u 获取 m 的监视器锁 => 获取锁以后，抛出 InterruptedException 异常。
>
> 这个流程在前面 **wait** 的小节已经讲过了，这也是很多人都不了解的知识点。如果还不懂，可以看下一小节的结束，我的两个简单的例子。
>
> 一个小细节：u 被中断，wait 方法返回，并不会立即抛出 InterruptedException 异常，而是在重新获取监视器锁之后才会抛出异常。

实例方法 thread.isInterrupted() 可以知道线程的中断状态。

调用静态方法 Thread.interrupted() 可以返回当前线程的中断状态，同时将中断状态设置为false。

> 所以说，如果是这个方法调用两次，那么第二次一定会返回 false，因为第一次会重置状态。当然了，前提是两次调用的中间没有发生设置线程中断状态的其他语句。



#### 17.2.4 等待、通知和中断 的交互（Interactions of Waits, Notification, and Interruption）

如果一个线程在等待期间，**同时发生了通知和中断**，他将发生：

- 从wait方法中正常返回，同时不改变中断状态
- 由于抛出了InterruptedException异常而从wait方法中返回的，中断状态设置为false

线程可能没有重置它的中断状态，同时从 wait 方法中正常返回，即第一种情况。

> 也就是说，线程是从 notify 被唤醒的，由于发生了中断，所以中断状态为 true

同样的，通知也不能由于中断而丢失。

> 这个要说的是，线程其实是从中断唤醒的，那么线程醒过来，同时中断状态会被重置为 false。

假设 m 的等待集合为 线程集合 s，并且在另一个线程中调用了 m.notify(), 那么将发生：

- 至少有集合 s 中的一个线程正常从 wait 方法返回，或者
- 集合 s 中的所有线程由抛出 InterruptedException 异常而返回。

> 考虑是否有这个场景：x 被设置了中断状态，notify 选中了集合中的线程 x，那么这次 notify 将唤醒线程 x，其他线程（我们假设还有其他线程在等待）不会有变化。
>
> 答案：存在这种场景。因为这种场景是满足上述条件的，而且此时 x 的中断状态是 true。

注意，如果一个线程同时被中断和通知唤醒，同时这个线程通过抛出 InterruptedException 异常从 wait 中返回，那么等待集合中的某个其他线程一定会被通知。