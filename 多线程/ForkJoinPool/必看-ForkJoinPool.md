**核心概念**

```java
ForkJoinPool内部提供了一个线程池common,本系列源码解析也都是在common上解析
common中parallelism值为CPU核心数量 - 1,所以如果CPU核心数量是8,那么parallelism就等于7
parallelism等于7,意味着WorkQueue数组长度是16
WorkQueue数组偶数位存放execute、submit方法的任务(Submission Task),一共有8个
WorkQueue数组奇数位存放fork方法的任务(Worker Task),一共有7个,也就是会有7个ForkJoinWorkerThread与之绑定
  
//表示XXX的掩码
XXX_MASK
```

---

