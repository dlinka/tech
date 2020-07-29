1.BrokerStartup#main

```java
BrokerController brokerController = createBrokerController(args);
↓
↓
BrokerController controller = new BrokerController(brokerConfig, nettyServerConfig, nettyClientConfig, messageStoreConfig);
boolean initResult = controller.initialize();
↓
↓
messageStore = new DefaultMessageStore(messageStoreConfig, brokerStatsManager, messageArrivingListener, brokerConfig);
↓
↓
//AllocateMappedFileService继承ServiceThread
allocateMappedFileService = new AllocateMappedFileService(this);
allocateMappedFileService.start();
```

2.AllocateMappedFileService#run

```java
while (!this.isStopped() && this.mmapOperation()) {}
↓
↓
try {
  //这里是从优先级队列中获取文件创建请求
  //优先级顺序根据文件名的大小
  req = requestQueue.take();
  AllocateRequest expectedRequest = requestTable.get(req.getFilePath());
	//并发的情况下队列之中可能会有多个
  if (expectedRequest != req) {
    return true;
  }
	if (req.getMappedFile() == null) {
    ...
    else {
      //初始化MappedFile
      mappedFile = new MappedFile(req.getFilePath(), req.getFileSize());
    }
  	req.setMappedFile(mappedFile);
    isSuccess = true;
  }
}finally {
	if (req != null && isSuccess)
    //下面的源码分析中会有阻塞等待这里创建MappedFile的逻辑
    req.getCountDownLatch().countDown();
}
```

3.MappedFileQueue#getLastMappedFile

**源码解析-消息存储(4.CommitLog#asyncPutMessage)**的调用栈会进入这里

```java
//这里会一次创建两个
///Users/dlinka/store/commitlog/00000000000000000000
String nextFilePath = storePath + File.separator + UtilAll.offset2FileName(createOffset);
String nextNextFilePath = storePath + File.separator + UtilAll.offset2FileName(createOffset + mappedFileSize);
if (allocateMappedFileService != null) {
	mappedFile = allocateMappedFileService.putRequestAndReturnMappedFile(nextFilePath, nextNextFilePath, mappedFileSize);
}
↓
↓
int canSubmitRequests = 2;
//创建第一个请求
AllocateRequest nextReq = new AllocateRequest(nextFilePath, fileSize);
boolean nextPutOK = requestTable.putIfAbsent(nextFilePath, nextReq) == null;
if (nextPutOK) {
  //添加到优先级队列
	boolean offerOK = requestQueue.offer(nextReq);
}
//创建第二个请求
AllocateRequest nextNextReq = new AllocateRequest(nextNextFilePath, fileSize);
boolean nextNextPutOK = requestTable.putIfAbsent(nextNextFilePath, nextNextReq) == null;
if (nextNextPutOK) {
	//添加到优先级队列
  boolean offerOK = requestQueue.offer(nextNextReq);
}
...
//只拿出第一个请求
AllocateRequest result = requestTable.get(nextFilePath);
if (result != null) {
  //阻塞等待上面创建MappedFile
	result.getCountDownLatch().await(waitTimeOut, TimeUnit.MILLISECONDS);
  requestTable.remove(nextFilePath);
	return result.getMappedFile();
}
```

