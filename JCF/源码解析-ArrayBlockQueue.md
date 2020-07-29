1.put

```java
final ReentrantLock lock = this.lock;
lock.lockInterruptibly();
try {
    //如果集合中元素个数等于数组长度就阻塞等待
  	while (count == items.length)
        notFull.await();
    enqueue(e);
} finally {
    lock.unlock();
}
↓
↓
final Object[] items = this.items;
//把元素放入指定位置
items[putIndex] = x;
//putIndex++
//长度等于数组长度从0开始
if (++putIndex == items.length)
    putIndex = 0;
count++;
//通知消费者
notEmpty.signal();
```

2.take

```java
final ReentrantLock lock = this.lock;
lock.lockInterruptibly();
try {
  	//如果集合元素为空就阻塞等待
    while (count == 0)
        notEmpty.await();
    return dequeue();
} finally {
    lock.unlock();
}
↓
↓
final Object[] items = this.items;
E x = (E) items[takeIndex];
items[takeIndex] = null;
//takeIndex++
if (++takeIndex == items.length)
    takeIndex = 0;
count--;
//通知生产者
notFull.signal();
return x;
```

