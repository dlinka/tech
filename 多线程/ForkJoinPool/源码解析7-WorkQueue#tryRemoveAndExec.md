**1.WorkQueue#tryRemoveAndExec(ForkJoinTask<?> task)**

```java
ForkJoinTask<?>[] a; int m, s, b, n;
if ((a = array) != null && (m = a.length - 1) >= 0 && task != null) {
    //任务队列中如果存在任务
  	while ((n = (s = top) - (b = base)) > 0) {
        for (ForkJoinTask<?> t;;) {
            long j = ((--s & m) << ASHIFT) + ABASE;
            //如果任务队列不存在任务
          	if ((t = (ForkJoinTask<?>)U.getObject(a, j)) == null)
              	//狗爷给的注释是"shorter than expected",翻译成中文"比预期短",下面是我的理解
              	//如果比预期短就返回true,代表我还没有查找任务,任务就被偷走了,我也可以去偷别人
              	//如果比预期长就返回false,代表自己查找任务还不如别人偷的快,我也就没有必要去偷别人的任务来执行了
                return s + 1 == top;
            //如果在任务队列中找到了这个任务
          	else if (t == task) {
                boolean removed = false;
								//如果任务在队列头部
              	if (s + 1 == top) {
                    if (U.compareAndSwapObject(a, j, task, null)) {
                        U.putOrderedInt(this, QTOP, s);
                        removed = true;
                    }
                }
                else if (base == b)
                  	//使用一个空任务填补在此位置
                    removed = U.compareAndSwapObject(a, j, task, new EmptyTask());
                if (removed)
                    task.doExec();
                break;
            }
            if (--n == 0)
                return false;
        }
      	//任务结束
        if (task.status < 0)
            return false;
    }
}
return true;
```

