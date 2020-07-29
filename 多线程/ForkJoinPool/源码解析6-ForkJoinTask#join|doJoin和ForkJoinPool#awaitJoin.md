1.**ForkJoinTask#join()**

```java
int s;
if ((s = doJoin() & DONE_MASK) != NORMAL)
    reportException(s);
return getRawResult();
```

2.**ForkJoinTask#doJoin()**

```java
//我只关心这块逻辑,也是最难理解的部分
((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
    (w = (wt = (ForkJoinWorkerThread)t).workQueue).tryUnpush(this) && (s = doExec()) < 0 ? 
  	s : wt.pool.awaitJoin(w, this, 0L)
↓
↓
//重写一下这块逻辑
if((t = Thread.currentThread()) instanceof ForkJoinWorkerThread){
  if((w = (wt = (ForkJoinWorkerThread)t).workQueue).tryUnpush(this)
     && (s = doExec()) < 0) {
    return s;
  } else {
    return wt.pool.awaitJoin(w, this, 0L); //3
  }
}
↓
↓
//进入WorkQueue#tryUnpush
ForkJoinTask<?>[] a; int s;
if ((a = array) != null && (s = top) != base &&
    //任务如果在top位置
    //更新top位置为null
    //同时top - 1
    U.compareAndSwapObject(a, (((a.length - 1) & --s) << ASHIFT) + ABASE, t, null)) {
    U.putOrderedInt(this, QTOP, s);
    return true;
}
return false;
```

**3.ForkJoinPool#awaitJoin(WorkQueue w, ForkJoinTask<?> task, long deadline)**

```java
int s = 0;
if (task != null && w != null) {
  	//记录上一个join的任务
    ForkJoinTask<?> prevJoin = w.currentJoin;
  	//设置当前任务为join的任务
    U.putOrderedObject(w, QCURRENTJOIN, task);
    for (;;) {
      	//任务结束
        if ((s = task.status) < 0)
            break;
      	//第一个条件:当前任务队列中的任务全被窃取走了
      	//第二个条件:在任务队列中查找这个任务,如果找到了就执行它
        else if (w.base == w.top || w.tryRemoveAndExec(task))
            //帮助窃取者执行任务
          	helpStealer(w, task);
      	//再次检查任务是否结束
        if ((s = task.status) < 0)
            break;
        long ms, ns;
        if (deadline == 0L)
            ms = 0L;
        else if ((ns = deadline - System.nanoTime()) <= 0L)
            break;
        else if ((ms = TimeUnit.NANOSECONDS.toMillis(ns)) <= 0L)
            ms = 1L;
      	//补偿一个线程执行任务,自己等待任务结束
      	//如果满足补偿条件,线程阻塞等待任务完成
      	//如果不满足补偿条件,重新走一遍上面的流程
      	if (tryCompensate(w)) {
            task.internalWait(ms);
            U.getAndAddLong(this, CTL, AC_UNIT);
        }
    }
    U.putOrderedObject(w, QCURRENTJOIN, prevJoin);
}
return s;
```

