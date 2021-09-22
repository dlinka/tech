**tryRemoveAndExec(ForkJoinTask<?> task)**

```java
ForkJoinTask<?>[] a; int m, s, b, n;
if ((a = array) != null && (m = a.length - 1) >= 0 && task != null) {
		//任务队列中如果存在任务
  	while ((n = (s = top) - (b = base)) > 0) {
        for (ForkJoinTask<?> t;;) {
            long j = ((--s & m) << ASHIFT) + ABASE;
          	//再次判断任务队列中是否存在任务
          	if ((t = (ForkJoinTask<?>)U.getObject(a, j)) == null)
              	//Doug Lea的注释是"shorter than expected",翻译成中文是"比预期短"
              	//如果比预期短就返回true,表示我还没有查找任务,任务就被窃取了,我可以去偷别人
              	//如果比预期长就返回false,表示自己查找任务还不如别人窃取的快,我也就没有必要去窃取别人的任务来执行了
                return s + 1 == top;
            //如果任务队列中存在这个任务
          	else if (t == task) {
                boolean removed = false;
								//如果任务在队列头部
              	if (s + 1 == top) {
                    if (U.compareAndSwapObject(a, j, task, null)) {
                        //top - 1
                      	U.putOrderedInt(this, QTOP, s);
                        removed = true;
                    }
                }
              	//如果任务不在队列头部,使用一个空任务填补在此位置
                else if (base == b)
                    removed = U.compareAndSwapObject(a, j, task, new EmptyTask());
                if (removed)
                  	//执行任务
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

---
