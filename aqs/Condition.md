# Condition

首先明确一点：**AQS中的Condition是基于ReentrantLock的。Condition不管是调用await进入等待还是signal唤醒，都需要获取到锁才能进行操作**



## 摘要

~~~java
 public static void main(String[] args) {
        ReentrantLock reentrantLock = new ReentrantLock(true);
        
        Condition condition = reentrantLock.newCondition();
		...
    }

    public Condition newCondition() {
        return sync.newCondition();
    }
~~~





~~~JAVA


package java.util.concurrent.locks;

import java.util.Date;
import java.util.concurrent.TimeUnit;


public interface Condition {

    void await() throws InterruptedException;

    void awaitUninterruptibly();

    long awaitNanos(long nanosTimeout) throws InterruptedException;

    boolean await(long time, TimeUnit unit) throws InterruptedException;

    boolean awaitUntil(Date deadline) throws InterruptedException;

    void signal();
    
    void signalAll();
}


   public class ConditionObject implements Condition, java.io.Serializable {
        private static final long serialVersionUID = 1173984872572414699L;
        /** First node of condition queue. */
        // 条件队列的第一个节点
        private transient Node firstWaiter;
        /** Last node of condition queue. */
        // 条件队列的最后一个节点
        private transient Node lastWaiter;

        /**
         * Creates a new {@code ConditionObject} instance.
         */
        public ConditionObject() { }

        // Internal methods
		......
       
    }

~~~



注意：在Condition中，有一个**条件队列**如下图。

![image-20210429155525341](img/image-20210429155525341.png)

- 条件队列和阻塞队列的节点，都是 Node 的实例，因为条件队列的节点是需要转移到阻塞队列中去的；
- 我们知道一个 ReentrantLock 实例可以通过多次调用 newCondition() 来产生多个 Condition 实例，这里对应 condition1 和 condition2。注意，ConditionObject 只有两个属性 firstWaiter 和 lastWaiter；
- 每个 condition 有一个关联的**条件队列**，如线程 1 调用 `condition1.await()` 方法即可将当前线程 1 包装成 Node 后加入到条件队列中，然后阻塞在这里，不继续往下执行，条件队列是一个单向链表；
- 调用`condition1.signal()` 触发一次唤醒，此时唤醒的是队头，会将condition1 对应的**条件队列**的 firstWaiter（队头） 移到**阻塞队列的队尾**，等待获取锁，获取锁后 await 方法才能返回，继续往下执行。



## await

### 加入条件队列(wait queue)

~~~java
   		/**
         * Implements interruptible condition wait.
         * <ol>
         * <li>If current thread is interrupted, throw InterruptedException.
         * <li>Save lock state returned by {@link #getState}.
         * <li>Invoke {@link #release} with saved state as argument,
         *     throwing IllegalMonitorStateException if it fails.
         * <li>Block until signalled or interrupted.
         * <li>Reacquire by invoking specialized version of
         *     {@link #acquire} with saved state as argument.
         * <li>If interrupted while blocked in step 4, throw InterruptedException.
         * </ol>
         */
		// 如果当前线程被中断，会抛出异常(awaitUninterruptibly()此方法不怕中断)
		// 阻塞线程直到signal或interrupt
        public final void await() throws InterruptedException {
            // 先判断中断状态
            if (Thread.interrupted())
                throw new InterruptedException();
            // 将线程添加到等待队列中
            Node node = addConditionWaiter();
            // await的前提是持有当前锁，进行await时需要释放锁并返回state值。
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            // 如果isOnSyncQueue(node)返回true，说明当前node已经到阻塞队列中了
            // 如果线程中断，会进行break( checkInterruptWhileWaiting(node) != 0)
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            // 线程被唤醒后，进入阻塞队列，等待获取锁
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }




        /**
         * Adds a new waiter to wait queue.
         * @return its new wait node
         */
		// 将当前节点加入到条件队列中，置于队尾
        private Node addConditionWaiter() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node t = lastWaiter;
            // If lastWaiter is cancelled, clean out.
            // 将状态不为Node.CONDITION的节点清除掉
            if (t != null && t.waitStatus != Node.CONDITION) {
                //具体的清除方法
                unlinkCancelledWaiters();
                t = lastWaiter;
            }
			// 新生成的节点的statu为Node.CONDITION
            Node node = new Node(Node.CONDITION);
			// 如果队列为空，当前节点为首节点。否则放在队尾
            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node;
            return node;
        }

		// 等待链表是单向链表
        private void unlinkCancelledWaiters() {
            Node t = firstWaiter;
            Node trail = null;
            while (t != null) {
                Node next = t.nextWaiter;
                if (t.waitStatus != Node.CONDITION) {
                    t.nextWaiter = null;
                    if (trail == null)
                        firstWaiter = next;
                    else
                        trail.nextWaiter = next;
                    if (next == null)
                        lastWaiter = trail;
                }
                else
                    trail = t;
                t = next;
            }
        }

~~~



### 释放锁

当前线程进入条件队列（wait queue）后，会进行释放锁的操作。**是完全释放**

~~~java
	/**
     * Invokes release with current state value; returns saved state.
     * Cancels node and throws exception on failure.
     * @param node the condition node for this wait
     * @return previous sync state
     */
	// 完全释放锁
    final long fullyRelease(Node node) {
        try {
            long savedState = getState();
            // 如果此方法执行失败，会将当前节点状态设置为Node.CANCELLED(取消)
            if (release(savedState))
                return savedState;
            throw new IllegalMonitorStateException();
        } catch (Throwable t) {
            node.waitStatus = Node.CANCELLED;
            throw t;
        }
    }

    /**
     * Releases in exclusive mode.  Implemented by unblocking one or
     * more threads if {@link #tryRelease} returns true.
     * This method can be used to implement method {@link Lock#unlock}.
     *
     * @param arg the release argument.  This value is conveyed to
     *        {@link #tryRelease} but is otherwise uninterpreted and
     *        can represent anything you like.
     * @return the value returned from {@link #tryRelease}
     */
	// 根据传入的参数，改别state值，并将当前阻塞队列头节点的后继节点唤醒
    public final boolean release(long arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

~~~

> 考虑一下这里的 savedState。如果在 condition1.await() 之前，假设线程先执行了 2 次 lock() 操作，那么 state 为 2，我们理解为该线程持有 2 把锁，这里 await() 方法必须将 state 设置为 0，然后再进入挂起状态，这样其他线程才能持有锁。当它被唤醒的时候，它需要重新持有 2 把锁，才能继续下去。
>
> 如果一个线程在不持有 lock 的基础上，就去调用 condition1.await() 方法，它能进入条件队列，但是在上面的这个方法中，由于它不持有锁，release(savedState) 这个方法肯定要返回 false，进入到异常分支，然后进入 finally 块设置 `node.waitStatus = Node.CANCELLED`，这个已经入队的节点之后会被后继的节点清除。

### 等待进入阻塞队列

~~~java
    ...
    // 此时已经释放锁了
    int interruptMode = 0;
	// 自旋判断节点是否在阻塞队列中
    while (!isOnSyncQueue(node)) {
        //如果不在阻塞队列中，将线程挂起
        LockSupport.park(this);
        // 线程被唤醒后，执行下列代码判断是否被中断过
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }


 /**
     * Returns true if a node, always one that was initially placed on
     * a condition queue, is now waiting to reacquire on sync queue.
     * @param node the node
     * @return true if is reacquiring
     */
	// 判断节点是否在阻塞队列中，如果节点在阻塞队列，返回true
	// 此方法若返回false，则将线程挂起
    final boolean isOnSyncQueue(Node node) {
        // node初始化时，waitStatus为Node.CONDITION。如果此时waitStatus还为Node.CONDITION，说明其还在条件队列中
        // 如果node的前驱节点为null，说明也没有在阻塞队列中。（条件队列是单向链表）
        if (node.waitStatus == Node.CONDITION || node.prev == null)
            return false;
        if (node.next != null) // If has successor, it must be on queue
            return true;
        /*
         * node.prev can be non-null, but not yet on queue because
         * the CAS to place it on queue can fail. So we have to
         * traverse from tail to make sure it actually made it.  It
         * will always be near the tail in calls to this method, and
         * unless the CAS failed (which is unlikely), it will be
         * there, so we hardly ever traverse much.
         */
        

        // 可以通过判断 node.prev() != null 来推断出 node 在阻塞队列吗？答案是：不能。
        // 这个可以看上篇 AQS 的入队方法，首先设置的是 node.prev 指向 tail，
        // 然后是 CAS 操作将自己设置为新的 tail，可是这次的 CAS 是可能失败的。
        // 此方法从阻塞队列的队尾开始从后往前遍历找，如果找到相等的，说明在阻塞队列，否则就是不在阻塞队列
        return findNodeFromTail(node);
    }

    /**
     * Returns true if node is on sync queue by searching backwards from tail.
     * Called only when needed by isOnSyncQueue.
     * @return true if present
     */

    private boolean findNodeFromTail(Node node) {
        // We check for node first, since it's likely to be at or near tail.
        // tail is known to be non-null, so we could re-order to "save"
        // one null check, but we leave it this way to help the VM.
        for (Node p = tail;;) {
            if (p == node)
                return true;
            if (p == null)
                return false;
            p = p.prev;
        }
    }


~~~



## signal

唤醒操作通常由另一个线程来进行操作。

~~~java
    /**
    * Moves the longest-waiting thread, if one exists, from the
    * wait queue for this condition to the wait queue for the
    * owning lock.
    *
    * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
    *         returns {@code false}
    */
	// 唤醒线程就是将线程对应的node从条件队列转到阻塞队列中。
    public final void signal() {
        // 只有持有独占锁的线程才能调用signal方法
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            doSignal(first);
    }



    /**
     * Removes and transfers nodes until hit non-cancelled one or
     * null. Split out from signal in part to encourage compilers
     * to inline the case of no waiters.
     * @param first (non-null) the first node on condition queue
     */
	// 从条件队列从头向尾遍历，找到第一个需要转移的node
    private void doSignal(Node first) {
        do {
            // first节点将要离开，所以让firstWaiter指向first的下个节点。如果为null，说明队列没node了
            if ( (firstWaiter = first.nextWaiter) == null){
				lastWaiter = null;      
            }
            // 断开first节点链
            first.nextWaiter = null;
            // 进行循环，如果唤醒失败，则更换节点继续唤醒
        } while (!transferForSignal(first) &&
                 (first = firstWaiter) != null);
    }


    /**
     * Transfers a node from a condition queue onto sync queue.
     * Returns true if successful.
     * @param node the node
     * @return true if successfully transferred (else the node was
     * cancelled before signal)
     */
    final boolean transferForSignal(Node node) {
        /*
         * If cannot change waitStatus, the node has been cancelled.
         */
        //CAS判断node状态是否为CONDITION。如果不是，说明节点被取消了。直接返回false，转移下一个节点
        if (!node.compareAndSetWaitStatus(Node.CONDITION, 0))
            return false;

        /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         */
        //尝试自旋进入阻塞队列的队尾。
        Node p = enq(node);
        // 节点p是node节点的前驱节点
        int ws = p.waitStatus;
        //ws>0代表p节点取消了等待锁，可直接唤醒node节点代表的线程
        if (ws > 0 || !p.compareAndSetWaitStatus(ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }

~~~

## 唤醒后检查中断状态



~~~java
int interruptMode = 0;
while (!isOnSyncQueue(node)) {
    // 线程挂起
    LockSupport.park(this);
	// 线程唤醒后，首先对线程的中断状态进行判断。是在signal之前进行中断还是之后进行中断
    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
        break;
}
~~~

> interruptMode 可以取值为 REINTERRUPT（1），THROW_IE（-1），0
>
> - REINTERRUPT： 代表 await 返回的时候，需要重新设置中断状态
> - THROW_IE： 代表 await 返回的时候，需要抛出 InterruptedException 异常
> - 0 ：说明在 await 期间，没有发生中断
>
> 有以下三种情况会让 LockSupport.park(this); 这句返回继续往下执行：
>
> 1. 常规路径。signal -> 转移节点到阻塞队列 -> 获取了锁（unpark）
> 2. 线程中断。在 park 的时候，另外一个线程对这个线程进行了中断
> 3. signal 的时候我们说过，转移以后的前驱节点取消了，或者对前驱节点的CAS操作失败了
> 4. 假唤醒。这个也是存在的，和 Object.wait() 类似，都有这个问题

~~~java
    /**
     * Checks for interrupt, returning THROW_IE if interrupted
     * before signalled, REINTERRUPT if after signalled, or
     * 0 if not interrupted.
     */
    private int checkInterruptWhileWaiting(Node node) {
        return Thread.interrupted() ?
            (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
        0;
    }


    /**
     * Transfers node, if necessary, to sync queue after a cancelled wait.
     * Returns true if thread was cancelled before being signalled.
     *
     * @param node the node
     * @return true if cancelled before the node was signalled
     */
	// 如果必要，将这个已取消等待的节点转移到阻塞队列
	// 如果节点在signal前就被取消，return true
    final boolean transferAfterCancelledWait(Node node) {
        // CAS设置node的ws为0。成功说明signal方法之前发生的中断
        if (node.compareAndSetWaitStatus(Node.CONDITION, 0)) {
            // 节点入阻塞队列
            enq(node);
            return true;
        }
        /*
         * If we lost out to a signal(), then we can't proceed
         * until it finishes its enq().  Cancelling during an
         * incomplete transfer is both rare and transient, so just
         * spin.
         */
        // 到这里是因为 CAS 失败，肯定是因为 signal 方法已经将 waitStatus 设置为了 0
        // signal 方法会将节点转移到阻塞队列，但是可能还没完成，这边自旋等待其完成
        // 当然，这种事情还是比较少的吧：signal 调用之后，没完成转移之前，发生了中断
        while (!isOnSyncQueue(node))
            Thread.yield();
        return false;
    }


~~~

这里描绘了一个场景，本来有个线程，它是排在条件队列的后面的，但是因为它被中断了，那么它会被唤醒，然后它发现自己不是被 signal 的那个，但是它会自己主动去进入到阻塞队列。

## 获取独占锁

从whlie中退出（要么中断，要么取消）后

~~~java
// acquireQueued(node, savedState) 的返回值就是代表线程是否被中断。如果返回 true，说明被中断了，而且 interruptMode != THROW_IE，说明在 signal 之前就发生中断了，这里将 interruptMode 设置为 REINTERRUPT，用于待会重新中断。
// accquireQueued详情可见RetreenLock
if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
	interruptMode = REINTERRUPT;
// signal时会将节点转移到阻塞队列中，并截断节点和条件队列的联系(node.nextWaiter=null)。
// 如果signal之前就已经中断，则node也会到阻塞队列中，但是并没有设置node.nextWaiter。因此进行unlinkCancelledWaiters()调用
if (node.nextWaiter != null) // clean up if cancelled
	unlinkCancelledWaiters();
if (interruptMode != 0)
	reportInterruptAfterWait(interruptMode);



private void reportInterruptAfterWait(int interruptMode)
    throws InterruptedException {
    if (interruptMode == THROW_IE)
        throw new InterruptedException();
    else if (interruptMode == REINTERRUPT)
        selfInterrupt();
}
~~~





## 超时机制的await

超时机制的await不难理解，是调用 parkNanos 方法来休眠指定的时间，醒来后判断是否 signal 调用了，调用了就是没有超时，否则就是超时了。超时的话，自己来进行转移到阻塞队列，然后抢锁。

~~~java
public final boolean await(long time, TimeUnit unit)
        throws InterruptedException {
    // 等待这么多纳秒
    long nanosTimeout = unit.toNanos(time);
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    // 当前时间 + 等待时长 = 过期时间
    final long deadline = System.nanoTime() + nanosTimeout;
    // 用于返回 await 是否超时
    boolean timedout = false;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        if (nanosTimeout <= 0L) {
            // 这里因为要 break 取消等待了。取消等待的话一定要调用 transferAfterCancelledWait(node) 这个方法
            // 如果这个方法返回 true，在这个方法内，将节点转移到阻塞队列成功
            // 返回 false 的话，说明 signal 已经发生，signal 方法将节点转移了。也就是说没有超时嘛
            timedout = transferAfterCancelledWait(node);
            break;
        }
        // spinForTimeoutThreshold 的值是 1000 纳秒，也就是 1 毫秒
        // 也就是说，如果不到 1 毫秒了，那就不要选择 parkNanos 了，自旋的性能反而更好
        if (nanosTimeout >= spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanosTimeout);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
        // 得到剩余时间
        nanosTimeout = deadline - System.nanoTime();
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
    return !timedout;
}
~~~



# AQS独占锁取消排队

如果我们要取消一个线程的排队，我们需要在另外一个线程中对其进行中断。比如某线程调用 lock() 老久不返回，我想中断它。一旦对其进行中断，此线程会从 `LockSupport.park(this);` 中唤醒，然后 `Thread.interrupted();` 返回 true。

即使是中断唤醒了这个线程，也就只是设置了 `interrupted = true` 然后继续下一次循环。而且，由于 `Thread.interrupted();`  会清除中断状态，第二次进 parkAndCheckInterrupt 的时候，返回会是 false。

在这个方法中，interrupted 只是用来记录是否发生了中断，然后用于方法返回值，其他没有做任何相关事情。

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}


private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);
    return Thread.interrupted();
}
```

lock() 方法处理中断的方法就是，你中断归中断，我抢锁还是照样抢锁，几乎没关系，只是我抢到锁了以后，设置线程的中断状态而已，也不抛出任何异常出来。调用者获取锁后，可以去检查是否发生过中断，也可以不理会。

~~~java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
~~~

我们来看 ReentrantLock 的另一个 lock 方法：

```java
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

方法上多了个 `throws InterruptedException` 

```java
public final void acquireInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (!tryAcquire(arg))
        doAcquireInterruptibly(arg);
}
```

继续往里：

```java
private void doAcquireInterruptibly(int arg) throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                // 就是这里了，一旦异常，马上结束这个方法，抛出异常。
                // 这里不再只是标记这个方法的返回值代表中断状态
                // 而是直接抛出异常，而且外层也不捕获，一直往外抛到 lockInterruptibly
                throw new InterruptedException();
        }
    } finally {
        // 如果通过 InterruptedException 异常出去，那么 failed 就是 true 了
        if (failed)
            cancelAcquire(node);
    }
}
```

既然到这里了，顺便说说 cancelAcquire 这个方法吧：

```java
private void cancelAcquire(Node node) {
    // Ignore if node doesn't exist
    if (node == null)
        return;
    node.thread = null;
    // Skip cancelled predecessors
    // 找一个合适的前驱。其实就是将它前面的队列中已经取消的节点都”请出去“
    Node pred = node.prev;
    while (pred.waitStatus > 0)
        node.prev = pred = pred.prev;
    // predNext is the apparent node to unsplice. CASes below will
    // fail if not, in which case, we lost race vs another cancel
    // or signal, so no further action is necessary.
    Node predNext = pred.next;
    // Can use unconditional write instead of CAS here.
    // After this atomic step, other Nodes can skip past us.
    // Before, we are free of interference from other threads.
    node.waitStatus = Node.CANCELLED;
    // If we are the tail, remove ourselves.
    if (node == tail && compareAndSetTail(node, pred)) {
        compareAndSetNext(pred, predNext, null);
    } else {
        // If successor needs signal, try to set pred's next-link
        // so it will get one. Otherwise wake it up to propagate.
        int ws;
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL ||
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {
            Node next = node.next;
            if (next != null && next.waitStatus <= 0)
                compareAndSetNext(pred, predNext, next);
        } else {
            unparkSuccessor(node);
        }
        node.next = node; // help GC
    }
}
```

其实这个方法没什么好说的，一行行看下去就是了，节点取消，只要把 waitStatus 设置为 Node.CANCELLED，会有非常多的情况被从阻塞队列中请出去，主动或被动。

# Java中的中断(Interrupt)

## 线程中断

中断代表线程状态，每个线程都关联了一个中断状态，是一个 true 或 false 的 boolean 值，初始值为 false。

关于中断状态，我们需要重点关注 Thread 类中的以下几个方法：

```java
public boolean isInterrupted() {}

public static boolean interrupted() {}

public void interrupt() {}
```

我们说中断一个线程，其实就是设置了线程的 interrupted status 为 true，至于说被中断的线程怎么处理这个状态，那是那个线程自己的事。如以下代码：

```java
while (!Thread.interrupted()) {
   doWork();
   System.out.println("我做完一件事了，准备做下一件，如果没有其他线程中断我的话");
}
```

当然，中断除了是线程状态外，还有其他含义，否则也不需要专门搞一个这个概念出来了。

如果线程处于以下三种情况，那么当线程被中断的时候，能自动感知到：

1. 来自 Object 类的 wait()、wait(long)、wait(long, int)，

	来自 Thread 类的 join()、join(long)、join(long, int)、sleep(long)、sleep(long, int)

	> 这几个方法的相同之处是，方法上都有: throws InterruptedException 
	>
	> 如果线程阻塞在这些方法上（我们知道，这些方法会让当前线程阻塞），这个时候如果其他线程对这个线程进行了中断，那么这个线程会从这些方法中立即返回，抛出 InterruptedException 异常，同时重置中断状态为 false。

2. 实现了 InterruptibleChannel 接口的类中的一些 I/O 阻塞操作，如 DatagramChannel 中的 connect 方法和 receive 方法等

	> 如果线程阻塞在这里，中断线程会导致这些方法抛出 ClosedByInterruptException 并重置中断状态。

3. Selector 中的 select 方法，参考下我写的 NIO 的文章

	> 一旦中断，方法立即返回

对于以上 3 种情况是最特殊的，因为他们能自动感知到中断（这里说自动，当然也是基于底层实现），**并且在做出相应的操作后都会重置中断状态为 false**。

那是不是只有以上 3 种方法能自动感知到中断呢？不是的，如果线程阻塞在 LockSupport.park(Object obj) 方法，也叫挂起，这个时候的中断也会导致线程唤醒，但是唤醒后不会重置中断状态，所以唤醒后去检测中断状态将是 true。



### InterruptedException 概述

它是一个特殊的异常，不是说 JVM 对其有特殊的处理，而是它的使用场景比较特殊。通常，我们可以看到，像 Object 中的 wait() 方法，ReentrantLock 中的 lockInterruptibly() 方法，Thread 中的 sleep() 方法等等，这些方法都带有 `throws InterruptedException`，我们通常称这些方法为阻塞方法（blocking method）。

阻塞方法一个很明显的特征是，它们需要花费比较长的时间（不是绝对的，只是说明时间不可控），还有它们的方法结束返回往往依赖于外部条件，如 wait 方法依赖于其他线程的 notify，lock 方法依赖于其他线程的 unlock等等。

当我们看到方法上带有 `throws InterruptedException` 时，我们就要知道，这个方法应该是阻塞方法，我们如果希望它能早点返回的话，我们往往可以通过中断来实现。 

除了几个特殊类（如 Object，Thread等）外，感知中断并提前返回是通过轮询中断状态来实现的。我们自己需要写可中断的方法的时候，就是通过在合适的时机（通常在循环的开始处）去判断线程的中断状态，然后做相应的操作（通常是方法直接返回或者抛出异常）。当然，我们也要看到，如果我们一次循环花的时间比较长的话，那么就需要比较长的时间才能**感知**到线程中断了。



### 处理中断

一旦中断发生，我们接收到了这个信息，然后怎么去处理中断呢？本小节将简单分析这个问题。

我们经常会这么写代码：

```java
try {
    Thread.sleep(10000);
} catch (InterruptedException e) {
    // ignore
}
// go on 
```

当 sleep 结束继续往下执行的时候，我们往往都不知道这块代码是真的 sleep 了 10 秒，还是只休眠了 1 秒就被中断了。这个代码的问题在于，我们将这个异常信息吞掉了。（对于 sleep 方法，我相信大部分情况下，我们都不在意是否是中断了，这里是举例）

AQS 的做法很值得我们借鉴，我们知道 ReentrantLock 有两种 lock 方法：

```java
public void lock() {
    sync.lock();
}

public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

前面我们提到过，lock() 方法不响应中断。如果 thread1 调用了 lock() 方法，过了很久还没抢到锁，这个时候 thread2 对其进行了中断，thread1 是不响应这个请求的，它会继续抢锁，当然它不会把“被中断”这个信息扔掉。我们可以看以下代码：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        // 我们看到，这里也没做任何特殊处理，就是记录下来中断状态。
        // 这样，如果外层方法需要去检测的时候，至少我们没有把这个信息丢了
        selfInterrupt();// Thread.currentThread().interrupt();
}
```

而对于 lockInterruptibly() 方法，因为其方法上面有 `throws InterruptedException` ，这个信号告诉我们，如果我们要取消线程抢锁，直接中断这个线程即可，它会立即返回，抛出 InterruptedException 异常。

在并发包中，有非常多的这种处理中断的例子，提供两个方法，分别为响应中断和不响应中断，对于不响应中断的方法，记录中断而不是丢失这个信息。如 Condition 中的两个方法就是这样的：

```java
void await() throws InterruptedException;
void awaitUninterruptibly();
```

> 通常，如果方法会抛出 InterruptedException 异常，往往方法体的第一句就是：
>
> ```java
> public final void await() throws InterruptedException {
>     if (Thread.interrupted())
>         throw new InterruptedException();
>      ...... 
> }
> ```

