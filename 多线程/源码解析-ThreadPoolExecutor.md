1.ThreadPoolExecutor#execute

```java
int c = ctl.get();
//如果线程池中线程数量小于核心线程数
if (workerCountOf(c) < corePoolSize) {
  if (addWorker(command, true))
  	return;
}
//如果线程数量大于核心线程数,任务放到阻塞队列中
if (isRunning(c) && workQueue.offer(command)) {
}
//如果阻塞队列满了,线程池中的线程数量小于最大线程数
else if (!addWorker(command, false))
```

2.ThreadPoolExecutor#addWorker

```java
for (;;) {
  int c = ctl.get();
  for (;;) {
    int wc = workerCountOf(c);
    //判断当前线程池中的线程数量是否大于corePoolSize
    if (wc >= (core ? corePoolSize : maximumPoolSize))
      return false;
    //更新线程池中线程数量+1
    if (compareAndIncrementWorkerCount(c))
      break retry;
}
...
boolean workerStarted = false;
boolean workerAdded = false;
Worker w = null;
try {
  //Runnable对象当成Worker的第一个任务传入
  //Worker的构造函数中会创建一个线程Thread
	w = new Worker(firstTask);
  final Thread t = w.thread;
  ...
  if (rs < SHUTDOWN || (rs == SHUTDOWN && firstTask == null)) {
    workerAdded = true;
  ...
  if (workerAdded) {
  	//线程执行
    t.start();
	}
}
```

3.Worker#run

```java
runWorker(this);
↓
↓
//拿到第一个任务
Runnable task = w.firstTask;
w.firstTask = null;
//第一次就拿firstTask去执行
//之后从阻塞队列中拿任务执行
while (task != null || (task = getTask()) != null) {  
	try {
    //任务执行
		task.run();
  } finally {
  	task = null;
    w.completedTasks++;
  }
}
↓
↓
//从阻塞队列中获取任务
Runnable r = timed ? workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) : workQueue.take();
if (r != null)
  return r;
```

---

