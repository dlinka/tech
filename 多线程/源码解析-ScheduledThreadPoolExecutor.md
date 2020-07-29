**核心概念**

```java
DelayedWorkQueue(一个无界阻塞队列)
  队列中的元素类型是接口RunnableScheduledFuture,实现类为ScheduledFutureTask
```

[源码解析-DelayedWorkQueue](../JCF/源码解析-DelayedWorkQueue.md)

---

**1.ScheduledThreadPoolExecutor构造方法**

```java
public ScheduledThreadPoolExecutor(int corePoolSize) {
	//ScheduledThreadPoolExecutor继承了ThreadPoolExecutor
  super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue());
}
```

**2.scheduleAtFixedRate**

```java
ScheduledFutureTask<Void> sft = new ScheduledFutureTask<Void>(
  																command,
  																null,
  																triggerTime(initialDelay, unit), //2.1
                                  unit.toNanos(period));
RunnableScheduledFuture<Void> t = decorateTask(command, sft);
sft.outerTask = t;
delayedExecute(t); //2.2
return t;
```

**2.1triggerTime**

```java
return triggerTime(unit.toNanos((delay < 0) ? 0 : delay));
↓
↓
//返回当前时间 + 延迟时间
return now() + ((delay < (Long.MAX_VALUE >> 1)) ? delay : overflowFree(delay));
```

**2.2delayedExecute**

```java
//先把任务放入阻塞队列
super.getQueue().add(task);
else
    ensurePrestart();
↓
↓
int wc = workerCountOf(ctl.get());
if (wc < corePoolSize)
  //源码解析-ThreadPoolExecutor
  addWorker(null, true);
```

**3.ScheduledFutureTask#run**

```java
//判断是否是周期性任务
boolean periodic = isPeriodic();
//周期性任务
else if (ScheduledFutureTask.super.runAndReset()) {
    //设置下一次任务的执行时间
    setNextRunTime(); //3.1
    reExecutePeriodic(outerTask); //3.2
}
```

**3.1setNextRunTime**

```java
long p = period;
if (p > 0)
	//下次运行时间 = 开始时间 + 间隔时间
  time += p;
else
  //scheduleWithFixedDelay方法构造ScheduledFutureTask时会传入period为负数,所以这里写-p
  //这里就会重新计算执行时间 = 返回当前时间 + 延迟时间
  time = triggerTime(-p);
```

**3.2reExecutePeriodic**

```java
//重新加入队列
super.getQueue().add(task);
else
    //上面2.2中有这个方法的解析
    ensurePrestart();
```

