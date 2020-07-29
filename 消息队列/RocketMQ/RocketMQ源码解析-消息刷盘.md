##### 刷盘服务相关类的初始化

1.CommitLog的构造方法

```java
if (FlushDiskType.SYNC_FLUSH == defaultMessageStore.getMessageStoreConfig().getFlushDiskType()) {
	//同步刷盘服务
  this.flushCommitLogService = new GroupCommitService();
} else {
  //异步刷盘服务
  this.flushCommitLogService = new FlushRealTimeService();
}
```



##### 刷盘服务相关类的启动

1.进入BrokerStartup#main

```java
start(brokerController);
↓
↓
controller.start();
↓
↓
if (this.messageStore != null) {
	this.messageStore.start();
}
↓
↓
this.commitLog.start();
↓
↓
this.flushCommitLogService.start();
```



##### 刷盘服务的调用

1.CommitLog#asyncPutMessage

```java
//消息存储
result = mappedFile.appendMessage(msg, this.appendMessageCallback);
...
//消息存储之后会在这里进行消息刷盘
CompletableFuture<PutMessageStatus> flushResultFuture = submitFlushRequest(result, putMessageResult, msg);
↓
↓
//同步刷盘
if (FlushDiskType.SYNC_FLUSH == this.defaultMessageStore.getMessageStoreConfig().getFlushDiskType()) {
  final GroupCommitService service = (GroupCommitService) this.flushCommitLogService;
  //broker选择了同步刷盘,producer自己也可以选择不用等待
  if (messageExt.isWaitStoreMsgOK()) {
  	GroupCommitRequest request = new GroupCommitRequest(result.getWroteOffset() + result.getWroteBytes(),	this.defaultMessageStore.getMessageStoreConfig().getSyncFlushTimeout());
    service.putRequest(request); //2
    return request.future();
	} else {
  	service.wakeup();
    return CompletableFuture.completedFuture(PutMessageStatus.PUT_OK);
	}
}
//异步刷盘
else{
  if (!this.defaultMessageStore.getMessageStoreConfig().isTransientStorePoolEnable()) {
    flushCommitLogService.wakeup();
  } else {
  	commitLogService.wakeup();
  }
  return CompletableFuture.completedFuture(PutMessageStatus.PUT_OK);
}
```

2.GroupCommitService#putRequest

```java
//加锁写到requestsWrite列表中
//可以看下面的同步队列源码分析,这里用了两个列表,一个用来写,一个用来读,减少阻塞
synchronized (this.requestsWrite) {
    this.requestsWrite.add(request);
}
```



##### 同步刷盘服务GroupCommitService

1.GroupCommitService#run

```java
while (!this.isStopped()) {
    try {
        this.waitForRunning(10); //2
        this.doCommit(); //3
    } catch (Exception e) {
    }finally {
        this.onWaitEnd(); //4
    }
}
```

2.ServiceThread#waitForRunning

```java
//阻塞等待
waitPoint.await(interval, TimeUnit.MILLISECONDS);
```

3.GroupCommitService#doCommit

```java
synchronized (this.requestsRead) {
  if (!this.requestsRead.isEmpty()) {
    //遍历队列
    for (GroupCommitRequest req : this.requestsRead) {
    	...
      CommitLog.this.mappedFileQueue.flush(0);
      ...
    }
    //清空队列
    this.requestsRead.clear();      
  }
}
```

4.进入GroupCommitService#onWaitEnd

```java
this.swapRequests();
↓
↓
//这里的设计挺巧妙
//刷盘的时候使用requestsRead
//写入刷盘请求使用requestsWrite
List<GroupCommitRequest> tmp = this.requestsWrite;
this.requestsWrite = this.requestsRead;
this.requestsRead = tmp;
```



##### 异步刷盘服务FlushRealTimeService

1.FlushRealTimeService#run

```java
while (!this.isStopped()) {
	//刷盘周期500毫秒
  int interval = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getFlushIntervalCommitLog();
  //4页
  int flushPhysicQueueLeastPages = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getFlushCommitLogLeastPages();
  ...
	try {
    else{
      //默认阻塞
      this.waitForRunning(interval);
    }
    CommitLog.this.mappedFileQueue.flush(flushPhysicQueueLeastPages);  
  }
}
```



---

无论同步刷盘还是异步刷盘都会调用这个flush方法

1.MappedFileQueue#flush

```java
//获取要刷盘的MappedFile
MappedFile mappedFile = this.findMappedFileByOffset(this.flushedWhere, this.flushedWhere == 0);
if (mappedFile != null) {
	int offset = mappedFile.flush(flushLeastPages);
}
```

2.MappedFile#flush

```java
else {
	this.mappedByteBuffer.force();
}
```



