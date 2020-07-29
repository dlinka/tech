**ForkJoinPool#helpStealer(WorkQueue w, ForkJoinTask<?> task)**

```java
WorkQueue[] ws = workQueues;
int oldSum = 0, checkSum, m;
if (ws != null && (m = ws.length - 1) >= 0 && w != null && task != null) {
    do {
        checkSum = 0;
        ForkJoinTask<?> subtask;
      	//j表示调用者
      	//v表示窃取者
        WorkQueue j = w, v;
        descent: for (subtask = task; subtask.status >= 0; ) {
            //寻找哪个WorkQueue窃取了当前任务
          	for (int h = j.hint | 1, k = 0, i; ; k += 2) {
              	if (k > m)
                    break descent;
                if ((v = ws[i = (h + k) & m]) != null) {
                  	//找到了窃取者
                    if (v.currentSteal == subtask) {
                        //记录这个窃取者的索引位
                      	//下次循环的时候需要用到
                      	j.hint = i;
                        break;
                    }
                    checkSum += v.base;
                }
            }
            for (;;) {
                ForkJoinTask<?>[] a; int b;
                checkSum += (b = v.base);
              	//窃取者当前可能正在join任务
                ForkJoinTask<?> next = v.currentJoin;
                if (subtask.status < 0 || j.currentJoin != subtask || v.currentSteal != subtask)
                    break descent;
                //窃取者的任务队列当前没有任务
              	//第一种情况:窃取者正在执行当前任务,还没有fork任务
              	//第二种情况:所有fork任务都被窃取了,当前正在join某一个fork的任务
              	if (b - v.top >= 0 || (a = v.array) == null) {
                  	//对应第一种情况:这个任务才开始执行,都还没有fork任务,所以重新循环
                  	//循环判断这里会使用oldSum和checkSum,如果两次循环都一样,就没有必要帮助了,浪费资源
                    if ((subtask = next) == null)
                        break descent;
                  	//对应第二种情况:查找join的任务被谁窃取了,说白了就是帮助窃取者的窃取者(递归)
                    j = v;
                    break;
                }
              	//偷取窃取者base位置的任务去执行
                int i = (((a.length - 1) & b) << ASHIFT) + ABASE;
                ForkJoinTask<?> t = ((ForkJoinTask<?>) U.getObjectVolatile(a, i));
                //假设窃取者任务一直被窃取,那么这里会一直循环,直到获取到base位置的任务
              	if (v.base == b) {
                  	if (t == null)
    										break descent;
                    if (U.compareAndSwapObject(a, i, t, null)) {
                      	v.base = b + 1;
                        //记录当前队列之前窃取的任务
                        ForkJoinTask<?> ps = w.currentSteal;
                        int top = w.top;
                        do {
                            U.putOrderedObject(w, QCURRENTSTEAL, t);
                            t.doExec();
                        //当前任务还没有结束 &&
                        //当前任务队列里面新增加了任务 &&
                        //当前队列任务队列按照LIFO执行任务
                        } while (task.status >= 0 && w.top != top && (t = w.pop()) != null);
                        U.putOrderedObject(w, QCURRENTSTEAL, ps);
                        //当前任务队列中新增加了任务,则去执行自己的任务去,不要再帮助了
                        if (w.base != w.top)
                            return;
                    }
                }
            }
        }
    //下面只考虑找到窃取者后的逻辑,只考虑第二个"checkSum += (b = v.base)"
    //假设窃取者一直在sleep,并且没有fork出来任何任务,那么循环两次就会退出,不在浪费时间帮助了
    } while (task.status >= 0 && oldSum != (oldSum = checkSum));
}
```

