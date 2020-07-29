**scan(WorkQueue w, int r)**

```java
WorkQueue[] ws; int m;
if ((ws = workQueues) != null && (m = ws.length - 1) > 0 && w != null) {
    //当前队列的扫描状态
  	int ss = w.scanState;
    //origin:循环WorkQueue数组的第一索引,用来判断是否循环一圈,让线程挂起的
    //oldSum和checkSum:每次扫描base总和,如果不相等,就没有必要让线程挂起,让线程继续运行
  	for (int origin = r & m, k = origin, oldSum = 0, checkSum = 0;;) {
        WorkQueue q; ForkJoinTask<?>[] a; ForkJoinTask<?> t;
        int b, n; long c;
        if ((q = ws[k]) != null) {
          	//判断这个队列是否有可以窃取的任务,判断的依据就是top和base的差值是否大于0
            if ((n = (b = q.base) - q.top) < 0 && (a = q.array) != null) {
                //计算base内存位置
              	long i = (((a.length - 1) & b) << ASHIFT) + ABASE;
              	//获取base内存位置的值
              	if ((t = ((ForkJoinTask<?>) U.getObjectVolatile(a, i))) != null && q.base == b) {
                  	//当前队列的扫描状态为激活状态
                  	if (ss >= 0) {
                        //出队
                        if (U.compareAndSwapObject(a, i, t, null)) {
                            q.base = b + 1;
                            if (n < -1)
                                signalWork(ws, q);
                            return t;
                        }
                    }
                    //当前队列的扫描状态为未激活,激活栈顶线程
                  	//为什么还要判断w.scanState < 0 ? 因为这个时候可能被其他WorkQueue线程激活了自己
                    else if (oldSum == 0 && w.scanState < 0)
                        tryRelease(c = ctl, ws[m & (int)c], AC_UNIT);
                }
              	//尝试刷新一下这个值
              	//这个值可能被当前线程改变,也可能被其他线程改变
                if (ss < 0)
                    ss = w.scanState;
              	//任务被其他线程拿走了,随机下一个位置,全部重新开始
                r ^= r << 1; r ^= r >>> 3; r ^= r << 10;
                origin = k = r & m;
                oldSum = checkSum = 0;
                continue;
            }
          	//叠加base值
            checkSum += b;
        }
      	//线性下一个位置判断是否回到最初位置
        if ((k = (k + 1) & m) == origin) {
            //如果两次扫描叠加base的结果一致,线程就可以尝试挂起
            if ((ss >= 0 || (ss == (ss = w.scanState))) && oldSum == (oldSum = checkSum)) {
                //循环出口
              	if (ss < 0 || w.qlock < 0)
                    break;
                int ns = ss | INACTIVE;
                long nc = ((SP_MASK & ns) | (UC_MASK & ((c = ctl) - AC_UNIT)));
								//保存前一个队列的索引
              	w.stackPred = (int)c;
              	//更改当前队列的扫描状态
                U.putInt(w, QSCANSTATE, ns);
                if (U.compareAndSwapLong(this, CTL, c, nc))
                    ss = ns;
                else
                    w.scanState = ss;
            }
            checkSum = 0;
        }
    }
}
return null;
```

---

