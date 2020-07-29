**线程执行的不确定性**

```java
CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> {
    //ForkJoinPool.commonPool-worker-1
    Thread.currentThread().getName();
		TimeUnit.SECONDS.sleep(0);
    return 1;
}).thenApply(i -> {
    //ForkJoinPool.commonPool-worker-1或者main
    Thread.currentThread().getName();
    return i;
});
```

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 1).thenApply(i -> {
    TimeUnit.SECONDS.sleep(1000);    
    return i;
});
//main线程被阻塞
```

---

**依赖前一个任务结果再去进行计算可以使用thenApply,为什么还会有thenCompose方法呢?**

```java
//compose2依赖compose1
CompletableFuture<Integer> compose1() {
    return CompletableFuture.supplyAsync(() -> 0);
}
CompletableFuture<String> compose2(Integer i) {
    return CompletableFuture.supplyAsync(() -> "a " + i);
}

//使用thenApply方法返回值没有使用thenCompose方法返回值友好
CompletableFuture<CompletableFuture<String>> future = compose1().thenApply(i -> compose2(i));
CompletableFuture<String> future = compose1().thenCompose(i -> compose2(i));
```

---