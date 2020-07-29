**1.offer**

**put和add方法本质都是调用offer**

```java
final ReentrantLock lock = this.lock;
lock.lock();
try {
    //PriorityQueue队列中添加元素
    q.offer(e);
    //如果添加之后成为了第一个元素,唤醒一个等待池中的线程重新竞争leader
    if (q.peek() == e) {
        leader = null;
        available.signal();
    }
    return true;
} finally {
    lock.unlock();
}
```

**2.take**

```java
final ReentrantLock lock = this.lock;
lock.lockInterruptibly();
try {
    //自旋
    for (;;) {
        //拿到PriorityQueue队列第一个元素
        E first = q.peek();
        if (first == null)
            //如果PriorityQueue队列为空,则当前线程进入等待池
            available.await();
        else {
            long delay = first.getDelay(NANOSECONDS);
            if (delay <= 0)
                //如果delay小于等于0表示元素到期可以出队,元素出队
                return q.poll();
            first = null; //2.1
            if (leader != null)
                //如果leader不为空,则当前线程进入等待池,直到被唤醒
                available.await();
            else {
                Thread thisThread = Thread.currentThread();
								//将当前线程变成leader
              	leader = thisThread;
                try {
                    //leader线程进入等待池,等待到期时间唤醒重新竞争锁
                    available.awaitNanos(delay);
                } finally {
                    //如果leader线程没有被更改,就把leader置为null
                    if (leader == thisThread)
                        leader = null;
                }
            }
        }
    }
} finally {
    //如果leader等于null并且集合中还有元素,唤醒一个follower
    if (leader == null && q.peek() != null)
        available.signal();
    lock.unlock();
}
```

**2.1为什么这里要写first = null?**

```java
假设n个线程同时调用take,这时会同时持有first元素
DelayQueue中有个10s后过期的元素e1,e1到期被一个线程拿走
那此时其他阻塞的线程还会持有这个已经被拿走了的e1,是没有意义的行为
为了防止内存泄漏,这里强制first为null
```