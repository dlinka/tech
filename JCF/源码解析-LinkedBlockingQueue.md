**1.LinkedBlockingQueue构造函数**

```java
this.capacity = capacity;
//初始化头尾节点
last = head = new Node<E>(null);
```

**2.put**

```java
//创建新节点
Node<E> node = new Node<E>(e);

final ReentrantLock putLock = this.putLock;
final AtomicInteger count = this.count;

//拿到put锁
putLock.lockInterruptibly();
try {
    //容器元素到达容量限制,等待
  	while (count.get() == capacity) {
  			notFull.await();
  	}
  
    enqueue(node); //2.1
  
  	//增加集合中元素数量
  	c = count.getAndIncrement();	
  
  	//这一步很关键因为消费者只会唤醒一个生产者
    //如果容器未到容量限制可以继续唤醒生产者生产
  	if (c + 1 < capacity)
    		notFull.signal();

} finally {
    putLock.unlock();
}

//唤醒一个消费者
if (c == 0)
    signalNotEmpty(); //2.2
```

**2.1enqueue**

```java
//最后一个节点指向node
last.next = node;
//最后一个节点变成node
last = last.next;
```

**2.2signalNotEmpty**

```java
final ReentrantLock takeLock = this.takeLock;
takeLock.lock();
try {
    notEmpty.signal();
} finally {
    takeLock.unlock();
}
```

**3.take**

```java
E x;
final AtomicInteger count = this.count;
final ReentrantLock takeLock = this.takeLock;

//拿到take锁
takeLock.lockInterruptibly();
try {
  	//容器没有元素,等待
    while (count.get() == 0) {
        notEmpty.await();
    }
  
    x = dequeue(); //3.1
  
    //减少集合中元素数量
    c = count.getAndDecrement();
  
    //这一步很关键因为生产者只会唤醒一个消费者
    //如果元素个数大于1就可以唤醒其他消费者
    if (c > 1)
        notEmpty.signal();
} finally {
    takeLock.unlock();
}

//唤醒一个生产者
if (c == capacity)
    signalNotFull();

return x;
```

**3.1dequeue**

```java
Node<E> h = head;
Node<E> first = h.next;

//注释上说这行有助于help GC,有知道的可以讲讲怎么有助于的
h.next = h;

//头节点变成下一个节点
head = first;
//返回值节点里面的值
E x = first.item;
//节点里面的值变成null
first.item = null;

return x;
```

**3.2signalNotFull**

```java
final ReentrantLock putLock = this.putLock;
putLock.lock();
try {
    notFull.signal();
} finally {
    putLock.unlock();
}
```

---

```java
通过上面的源码知道LinkedBlockingQueue使用了两把锁,一把是对生产者,一把是对消费者
最开始我也疑问这么设计不会产生线程不安全吗
其实并不会因为在put的时候使用的是last变量,而在take的时候使用的是head变量,很巧妙的设计
```



