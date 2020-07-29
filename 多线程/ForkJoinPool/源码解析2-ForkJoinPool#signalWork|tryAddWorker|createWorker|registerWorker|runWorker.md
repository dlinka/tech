**signalWork(WorkQueue[] ws, WorkQueue q)**

```java
long c; int sp, i; WorkQueue v; Thread p;
//当前活动线程数不够
while ((c = ctl) < 0L) {
  	/**
  	 * 当前没有空闲中的线程
  	 * 1.一个线程都没有创建
  	 * 2.一个空闲线程都没有
  	 */
    if ((sp = (int)c) == 0) {
        //当前总线程数不够
        if ((c & ADD_WORKER) != 0L)
            tryAddWorker(c);
        break;
    }
    if (ws.length <= (i = sp & SMASK))
        break;
    if ((v = ws[i]) == null)                  
        break;
    /**
     * 计算等待线程栈栈顶WorkQueue激活后的scanState
     * 1.增加一次版本变化(SS_SEQ)
     * 2.修改WorkQueue的扫描状态(INACTIVE)
     */
    int vs = (sp + SS_SEQ) & ~INACTIVE;
  	/**
  	 * 这个过程可能存在ABA问题,具体可以看scan方法
  	 * 所以需要判断等待线程栈栈顶WorkQueue的scanState这时有没有发生过变化
  	 */
    int d = sp - v.scanState;
    long nc = (UC_MASK & (c + AC_UNIT)) | (SP_MASK & v.stackPred);
    if (d == 0 && U.compareAndSwapLong(this, CTL, c, nc)) {
        v.scanState = vs;
        if ((p = v.parker) != null)
            U.unpark(p);
        break;
    }
 	  ...
}
```

**tryAddWorker(long c)**

```java
boolean add = false;
do {
  	//活动线程数+1,总线程数+1
    long nc = ((AC_MASK & (c + AC_UNIT)) | (TC_MASK & (c + TC_UNIT)));
    if (ctl == c) {
        int rs, stop;
        if ((stop = (rs = lockRunState()) & STOP) == 0)
            add = U.compareAndSwapLong(this, CTL, c, nc);
        unlockRunState(rs, rs & ~RSLOCK);
      	...
        if (add) {
            createWorker();
            break;
        }
    }
} while (((c = ctl) & ADD_WORKER) != 0L && (int)c == 0);
```

**createWorker()**

```java
ForkJoinWorkerThreadFactory fac = factory;
Throwable ex = null;
ForkJoinWorkerThread wt = null;
try {
  	//创建线程
  	//factory的默认实现类为CommonPoolForkJoinWorkerThreadFactory
    if (fac != null && (wt = fac.newThread(this)) != null) {
        //启动线程
      	wt.start();
        return true;
    }
} catch (Throwable rex) {
    ex = rex;
}
...
return false;
```

**registerWorker(ForkJoinWorkerThread wt)**

```java
...
//这里创建的WorkQueue与线程绑定
WorkQueue w = new WorkQueue(this, wt);
int i = 0;
int mode = config & MODE_MASK;
int rs = lockRunState();
try {
    WorkQueue[] ws; int n; 
    if ((ws = workQueues) != null && (n = ws.length) > 0) {
      	//第一次s等于0x9e3779b9
      	int s = indexSeed += SEED_INCREMENT;
        int m = n - 1;
      	//计算WorkQueue在WorkQueue数组的下标,得到一个奇数
        i = ((s << 1) | 1) & m;
      	...
        w.hint = s;
      	//奇数数组索引｜任务出队模式(默认LIFO_QUEUE)
        w.config = i | mode;
        w.scanState = i;
        ws[i] = w;
    }
} finally {
    unlockRunState(rs, rs & ~RSLOCK);
}
//设置线程名
wt.setName(workerNamePrefix.concat(Integer.toString(i >>> 1)));
return w;
```

**runWorker(WorkQueue w)**

```java
//初始化WorkQueue#array
w.growArray();
int seed = w.hint;
int r = (seed == 0) ? 1 : seed;
for (ForkJoinTask<?> t;;) {
  	//窃取任务
    if ((t = scan(w, r)) != null)
      	//执行任务
        w.runTask(t);
    else if (!awaitWork(w, r))
        break;
  	//一种伪随机算法(XORSHIFT)
    r ^= r << 13; r ^= r >>> 17; r ^= r << 5;
}
```

---



