**核心概念**

```java
ScheduledThreadPool中定义,存储RunnableScheduledFuture接口的对象
  
这个队列核心思想和DelayQueue差不多,使用堆排序-小顶堆,Leader-Flower线程模型等思想,但是内部并没有使用PriorityQueue
```

**为什么有DelayQueue还要DelayedWorkQueue?**

```java
DelayQueue中如果要查找某个元素需要遍历整个队列,时间复杂度为O(n)
DelayedWorkQueue则优化了查找某个元素的效率,时间复杂度为O(1)
```

---

**1.offer**

```java
RunnableScheduledFuture<?> e = (RunnableScheduledFuture<?>)x;
final ReentrantLock lock = this.lock;
lock.lock();
try {
  int i = size;
	size = i + 1;
	if (i == 0) {
    queue[0] = e;
    //跟DelayQueue的不同就是这里元素会记录在队列中的位置
    setIndex(e, 0);
	} else {
    siftUp(i, e);
	}
	if (queue[0] == e) {
    leader = null;
    available.signal();
	}
} finally {
    lock.unlock();
}
return true;
↓
↓
//siftUp
while (k > 0) {
    int parent = (k - 1) >>> 1;
    RunnableScheduledFuture<?> e = queue[parent];
    if (key.compareTo(e) >= 0)
        break;
    queue[k] = e;
    //跟DelayQueue的不同就是这里元素会记录在队列中的位置
    setIndex(e, k);
    k = parent;
}
queue[k] = key;
//跟DelayQueue的不同就是这里元素会记录在队列中的位置
setIndex(key, k);
```

**2.take**

```java
final ReentrantLock lock = this.lock;
lock.lockInterruptibly();
try {
    for (;;) {
        RunnableScheduledFuture<?> first = queue[0];
        if (first == null)
            available.await();
        else {
            long delay = first.getDelay(NANOSECONDS);
            if (delay <= 0)
                //出队
                return finishPoll(first);
            first = null;
            if (leader != null)
                available.await();
            else {
                Thread thisThread = Thread.currentThread();
                leader = thisThread;
                try {
                    available.awaitNanos(delay);
                } finally {
                    if (leader == thisThread)
                        leader = null;
                }
            }
        }
    }
} finally {
    if (leader == null && queue[0] != null)
        available.signal();
    lock.unlock();
}
```

**2.1finishPoll**

```java
int s = --size;
RunnableScheduledFuture<?> x = queue[s];
queue[s] = null;
if (s != 0)
    siftDown(0, x);
//这里把元素在队列中的位置赋值为-1
setIndex(f, -1);
return f;
↓
↓
//siftDown
int half = size >>> 1;
while (k < half) {
    int child = (k << 1) + 1;
    RunnableScheduledFuture<?> c = queue[child];
    int right = child + 1;
    if (right < size && c.compareTo(queue[right]) > 0)
        c = queue[child = right];
    if (key.compareTo(c) <= 0)
        break;
    queue[k] = c;
    //跟DelayQueue的不同就是这里元素会记录在队列中的位置
    setIndex(c, k);
    k = child;
}
queue[k] = key;
//跟DelayQueue的不同就是这里会修改元素在队列中的位置
setIndex(key, k);
```

**3.setIndex**

```java
if (f instanceof ScheduledFutureTask)
	((ScheduledFutureTask)f).heapIndex = idx;
```

**4.indexOf**

```java
if (x instanceof ScheduledFutureTask) {
    int i = ((ScheduledFutureTask) x).heapIndex;
    if (i >= 0 && i < size && queue[i] == x)
        return i;
}
```

---