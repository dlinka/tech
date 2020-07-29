**externalSubmit(ForkJoinTask<?> task)**

```java
//线程随机数种子
int r;
if ((r = ThreadLocalRandom.getProbe()) == 0) {
    //初始化线程随机数种子
  	ThreadLocalRandom.localInit();
  	//第一次 = 0x9e3779b9
    r = ThreadLocalRandom.getProbe();
}
for (;;) {
		WorkQueue[] ws; WorkQueue q; int rs, m, k;
		boolean move = false;
  	//不考虑这种情况
  	if ((rs = runState) < 0) { ... }
  	//初始化WorkQueue数组
  	else if ((rs & STARTED) == 0 || ((ws = workQueues) == null || (m = ws.length - 1) < 0)) {
    		int ns = 0;
    		rs = lockRunState();
    		try {
      		  if ((rs & STARTED) == 0) {
         		   	U.compareAndSwapObject(this, STEALCOUNTER, null, new AtomicLong());
          		  //拿到parallelism,我的电脑parallelism = 7
          			int p = config & SMASK;
          			//简单理解就是通过parallelism计算一个2的幂
          			int n = (p > 1) ? p - 1 : 1;
            		n |= n >>> 1; n |= n >>> 2;  n |= n >>> 4; n |= n >>> 8; n |= n >>> 16; n = (n + 1) << 1;
            		//初始化一个长度为16的WorkQueue数组
            		workQueues = new WorkQueue[n];
            		ns = STARTED;
        		}
    		} finally {
      		  unlockRunState(rs, (rs & ~RSLOCK) | ns);
    		}
		}
		//将用户提交的task放到下面创建的WorkQueue的任务数组中
  	else if ((q = ws[k = r & m & SQMASK]) != null) {
    		if (q.qlock == 0 && U.compareAndSwapInt(q, QLOCK, 0, 1)) {
      			ForkJoinTask<?>[] a = q.array;
      			//第一次s = 4096
      			int s = q.top;
        		boolean submitted = false;
        		try {
          			//WorkQueue中的任务数组不等于null并且任务未满 || 任务数组扩容成功
          			//第一次执行growArray方法会得到一个长度为8192的任务数组
          	  	if ((a != null && a.length > s + 1 - q.base) || (a = q.growArray()) != null) {
                  	int j = (((a.length - 1) & s) << ASHIFT) + ABASE;
          	      	//任务放到任务数组的top的位置
          	    		U.putOrderedObject(a, j, task);
                  	//top = top + 1
          	      	U.putOrderedInt(q, QTOP, s + 1);
          	      	submitted = true;
          	  	}
        		} finally {
          		  U.compareAndSwapInt(q, QLOCK, 1, 0);
        		}
        		if (submitted) {
          		  signalWork(ws, q);
          	  	return;
        		}
    		}
      	...
		}
  	//创建WorkQueue并将其放到上面创建的WorkQueue数组中
  	else if (((rs = runState) & RSLOCK) == 0) {
    		q = new WorkQueue(this, null);
    		//hint = 线程随机数
      	q.hint = r;
	  		//config = 数组索引｜共享队列
    		q.config = k | SHARED_QUEUE;
    		q.scanState = INACTIVE;
    		rs = lockRunState();
    		if (rs > 0 &&  (ws = workQueues) != null && k < ws.length && ws[k] == null)
          	//索引位置为通过线程随机数种子计算出来的一个偶数
          	//第一次k = 8
        		ws[k] = q;
    		unlockRunState(rs, rs & ~RSLOCK);
		}
  	...
}
```

---

**ws[k = r & m & SQMASK]**

```java
r = 初始化线程随机数种子 = 0x9e3779b9
m = 数组长度-1 = 15
SQMARK = 1111110

//多少个线程可以把所有偶数索引遍历到?需要9个线程
我的电脑是8核,parallelism = 7
创建WorkQueue数组长度 = 16
↓
↓
8
2
10
4
12
6
14
8
0
  
```



