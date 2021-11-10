# BlockingQueue

BlockingQueue即阻塞队列，是一个先进先出 ( FIFO ) 的队列 。作为一个阻塞队列，**当入队列时，若队列已满，则阻塞调用者；当出队列时，若队列为空，则阻塞调用者。**



BlockingQueue为入队提供了三个方法，分别是 add()，offer()，和 put()。他们的区别就是 add（）和offer（）的返回值是布尔类型，而put无返回值，还会抛出中断异常，所以add（）和offer（）是无阻塞的，也是Queue本身定义的接口，而put（）是阻塞式的。add（）和offer（）的区别不大，当队列为满的时候，前者会抛出异常，后者则直接返回false。

出队也是类似。其实 BlockingQueue 对入队，出队，获取元素都提供了不同的方法在不同的场景中使用，如下表格

|             | *Throws exception* | *Special value* | *Blocks*         | *Times out*          |
| ----------- | ------------------ | --------------- | ---------------- | -------------------- |
| **Insert**  | add(e)             | offer(e)        | **put(e)**       | offer(e, time, unit) |
| **Remove**  | remove()           | poll()          | **take()**       | poll(time, unit)     |
| **Examine** | element()          | peek()          | *not applicable* | *not applicable*     |

- 抛出异常；
- 返回特殊值（null 或 true/false，取决于具体的操作）；
- 阻塞等待此操作，直到这个操作成功；
- 阻塞等待此操作，直到成功或者超时指定时间

