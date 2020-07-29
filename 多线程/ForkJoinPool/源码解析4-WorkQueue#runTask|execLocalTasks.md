**WorkQueue#runTask(ForkJoinTask<?> task)**

```java
//scanState最低位变成0,表示WorkQueue正在执行任务
scanState &= ~SCANNING;
(currentSteal = task).doExec();
/**
 * 为什么不使用currentSteal = null?
 * currentSteal使用volatile修饰
 * putOrderedObject保证指令不重排序,但是不保证可见性,所以这样写入的性能更高
 */
U.putOrderedObject(this, QCURRENTSTEAL, null);
execLocalTasks();
//任务执行完毕,恢复scanState
scanState |= SCANNING;
↓
↓
//ForkJoinTask#doExec()
completed = exec();
↓
↓
//RecursiveAction#exec()
compute();
```

**WorkQueue#execLocalTasks()**

```java
int b = base, m, s;
ForkJoinTask<?>[] a = array;
//本地任务数组中存在任务
if (b - (s = top - 1) <= 0 && a != null && (m = a.length - 1) >= 0) {
    //默认是LIFO_QUQUE(栈)
  	if ((config & FIFO_QUEUE) == 0) {
    	for (ForkJoinTask<?> t;;) {
        //top - 1位置的任务出队,更新top的值
        if ((t = (ForkJoinTask<?>) U.getAndSetObject(a, ((m & s) << ASHIFT) + ABASE, null)) == null)
        	break;
    		U.putOrderedInt(this, QTOP, s);
    		t.doExec();
        //本地任务数组中不存在任务
    		if (base - (s = top - 1) > 0)
        	break;
			}
    }
  	...
}
```

