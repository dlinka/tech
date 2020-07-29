**任务state**

```java
0=NEW //任务未执行
1=COMPLETING //任务完成了或者异常了,但是还没有保存任务结果或者异常信息
2=NORMAL //任务完成结果已保存
3=EXCEPTIONAL //任务异常结果已保存
4=CANCELLED //任务未执行或者任务执行中,调用cancel(false),取消任务但是不中断任务
5=INTERRUPTING //任务未执行或者任务执行中,调用cancel(true),取消任务并且准备中断任务
6=INTERRUPTED //中断任务
```

---

**1.FutureTask的构造函数**

```java
this.callable = callable;
//state(volatile修饰)赋值操作写在后面可以保证其他线程对callable的可见性
this.state = NEW;
```

**2.run方法**

```java
//当前线程赋值给runner属性,保证只有一个线程执行下面的代码
if (state != NEW ||
    !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
	return;
try {
  Callable<V> c = callable;
  if (c != null && state == NEW) {
  	try {
      //执行任务
    	result = c.call();
    	ran = true;
		} catch (Throwable ex) {}
		if (ran)
      //保存结果
    	set(result);
  }
} finally {
  //runner必须在设置了state之后再置空
  //如果不这样做,可能存在并发问题,另外一个线程通过方法开头的if判断
  runner = null;
  //假设线程池中一个线程执行到这里,任务状态此时是INTERRUPTING
  //现在假设方法直接结束了,这个线程去做下一个任务,同时cancle方法这时调用到t.interrupt(),这样就会打断线程当前执行的任务
	int s = state;
  if (s >= INTERRUPTING)
  	handlePossibleCancellationInterrupt(s);
}
```

**2.1set**

```java
if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
  outcome = v;
  //此时对业务来讲只需要获取执行结果就可以了,不需要再关心state状态,所以使用putOrderedInt方法提高性能
  UNSAFE.putOrderedInt(this, stateOffset, NORMAL);
  finishCompletion();
}
```

**3.get**

```java
int s = state;
if (s <= COMPLETING)
    s = awaitDone(false, 0L);
return report(s);
↓
↓
//进入awaitDone方法
WaitNode q = null;
boolean queued = false;
for (;;) {
  //当前线程被打断,移除节点
  if (Thread.interrupted()) {
      removeWaiter(q);
      throw new InterruptedException();
	}
  
  int s = state;
  if (s > COMPLETING) {
    //任务已经结束,直接返回结果
    return s;
  }
  else if (s == COMPLETING) {
    //任务已经结束,但是还没有保存任务结果
  	Thread.yield();
  }
  else if (q == null) {
    //构建等待节点
    q = new WaitNode();
  }
  else if (!queued) {
    //把上一步构建的等待节点放到队首
    queued = UNSAFE.compareAndSwapObject(this, waitersOffset, q.next = waiters, q);
  }
  else {
    //阻塞等待
    LockSupport.park(this);
  }
}
```

**4.finishCompletion方法**

```java
for (WaitNode q; (q = waiters) != null;) {
  if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
  	for (;;) {
    	Thread t = q.thread;
      if (t != null) {
      	q.thread = null;
        LockSupport.unpark(t); //唤醒这个线程
      }
      //下一个节点
      WaitNode next = q.next;
			//最后一个节点结束循环
      if (next == null)
         break;
      //当前节点的next赋值为null,帮助GC
      q.next = null;
      //以下一个节点开始循环
      q = next;
    }
    break;
  }
}
//如果多个FutureTask使用同一个callable,这一步是减少引用
callable = null;
```

**5.cancel**

```java
//任务未执行或者任务执行中
//调用cancel(false),取消任务但是不中断任务
//调用cancel(true),取消任务并且准备中断任务
if (!(state == NEW && UNSAFE.compareAndSwapInt(this, stateOffset, NEW, mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
	return false;
try {
    if (mayInterruptIfRunning) {
        try {
            Thread t = runner;
            if (t != null)
                t.interrupt();
        } finally {
            UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
        }
    }
} finally {
    finishCompletion();
}
return true;
```

---



