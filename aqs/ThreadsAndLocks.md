

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

	1. 阿松大
	2. 阿松大
	3. 阿松大
	4. 阿松大