**ForkJoinTask#fork()**

```java
Thread t;
if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
    ((ForkJoinWorkerThread)t).workQueue.push(this);
return this;
```

**WorkQueue#push(ForkJoinTask<?> task)**

```java
ForkJoinTask<?>[] a; ForkJoinPool p;
int b = base, s = top, n;
if ((a = array) != null) {
    int m = a.length - 1;
  	//任务放到top的位置,然后top + 1
    U.putOrderedObject(a, ((m & s) << ASHIFT) + ABASE, task);
    U.putOrderedInt(this, QTOP, s + 1);
  	/**
  	 * 如果n<=1,很可能当前任务刚执行fork方法
  	 * 所以这时可以尝试唤醒等待线程栈栈顶WorkQueue参与竞争,提升整体并行性
  	 */
    if ((n = s - b) <= 1) {
        if ((p = pool) != null)
            p.signalWork(p.workQueues, this);
    }
}
```

