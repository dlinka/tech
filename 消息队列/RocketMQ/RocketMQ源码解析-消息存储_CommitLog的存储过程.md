### 相关类初始化

1.BrokerStartup#main

```java
BrokerController brokerController = createBrokerController(args);
↓
↓
final BrokerController controller = new BrokerController(brokerConfig, nettyServerConfig, nettyClientConfig, messageStoreConfig);
...
boolean initResult = controller.initialize();
↓
↓
//DefaultMessageStore
messageStore = new DefaultMessageStore(messageStoreConfig, brokerStatsManager, messageArrivingListener, brokerConfig); //2
...
//Netty
remotingServer = new NettyRemotingServer(nettyServerConfig, clientHousekeepingService);
...
registerProcessor(); //3
```

2.DefaultMessageStore的构造方法

```java
//CommitLog
this.commitLog = new CommitLog(this);
↓
↓
//MappedFileQueue
this.mappedFileQueue = new MappedFileQueue(defaultMessageStore.getMessageStoreConfig().getStorePathCommitLog(), defaultMessageStore.getMessageStoreConfig().getMappedFileSizeCommitLog(), defaultMessageStore.getAllocateMappedFileService());
↓
↓
//MappedFile对应的就是commitlog文件
private final CopyOnWriteArrayList<MappedFile> mappedFiles = new CopyOnWriteArrayList<MappedFile>();
```

3.BrokerController#registerProcessor

```java
//消息接收处理类
SendMessageProcessor sendProcessor = new SendMessageProcessor(this);
...
//注册到Netty
remotingServer.registerProcessor(RequestCode.SEND_MESSAGE_V2, sendProcessor, sendMessageExecutor);
//把消息接收处理类也注册到fastRemotingServer中,但是fastRemotingServer不会处理PULL_MESSAGE类型的请求
//NOTE:如果remotingServer出现性能瓶颈,可以使用fastRemotingServer来分摊一部分流量
fastRemotingServer.registerProcessor(RequestCode.SEND_MESSAGE_V2, sendProcessor, sendMessageExecutor);
↓
↓
Pair<NettyRequestProcessor, ExecutorService> pair = new Pair<NettyRequestProcessor, ExecutorService>(processor, executorThis);
this.processorTable.put(requestCode, pair);
```

---

### 消息存储流程

1.NettyRemotingAbstract#processMessageReceived  

NettyRemotingAbstract是上面初始化**remotingServer**的实现类NettyRemotingServer的父类

```java
case REQUEST_COMMAND:
    processRequestCommand(ctx, cmd);
↓
↓
//Pair
final Pair<NettyRequestProcessor, ExecutorService> matched = this.processorTable.get(cmd.getCode());
...
Runnable run = new Runnable() {
	@Override
  public void run() {
    ...
		if (pair.getObject1() instanceof AsyncNettyRequestProcessor) {
    	AsyncNettyRequestProcessor processor = (AsyncNettyRequestProcessor)pair.getObject1();
    	processor.asyncProcessRequest(ctx, cmd, callback);
		}
  }
};
...
final RequestTask requestTask = new RequestTask(run, ctx.channel(), cmd);
//提交到线程池去执行
pair.getObject2().submit(requestTask);
```

2.SendMessageProcessor#asyncProcessRequest

```java
asyncProcessRequest(ctx, request).thenAcceptAsync(responseCallback::callback, brokerController.getSendMessageExecutor());
↓
↓
return asyncSendMessage(ctx, request, mqtraceContext, requestHeader);
↓
↓
//MessageExtBrokerInner
MessageExtBrokerInner msgInner = new MessageExtBrokerInner();
msgInner.setTopic(requestHeader.getTopic());
msgInner.setQueueId(queueIdInt);
...
else {
		putMessageResult = this.brokerController.getMessageStore().asyncPutMessage(msgInner);
}
```

3.DefaultMessageStore#asyncPutMessage

```java
CompletableFuture<PutMessageResult> putResultFuture = this.commitLog.asyncPutMessage(msg);
```

4.CommitLog#asyncPutMessage

```java
//默认情况使用自旋锁
putMessageLock.lock();
try{
  if (null == mappedFile || mappedFile.isFull()) {
    //MappedFile
    mappedFile = this.mappedFileQueue.getLastMappedFile(0);
	}
	result = mappedFile.appendMessage(msg, this.appendMessageCallback);
} finally {
  putMessageLock.unlock();
}
```

5.MappedFile#appendMessage

```java
return appendMessagesInner(msg, cb);
↓
↓
ByteBuffer byteBuffer = writeBuffer != null ? writeBuffer.slice() : mappedByteBuffer.slice();
byteBuffer.position(currentPos);
if (messageExt instanceof MessageExtBrokerInner) {
	result = cb.doAppend(getFileFromOffset(), byteBuffer, fileSize - currentPos, (MessageExtBrokerInner) messageExt);
}
```

6.DefaultAppendMessageCallback#doAppend

```java
this.msgStoreItemMemory.putInt(msgLen);
this.msgStoreItemMemory.putInt(CommitLog.MESSAGE_MAGIC_CODE);
...
byteBuffer.put(this.msgStoreItemMemory.array(), 0, msgLen);
...
AppendMessageResult result = new AppendMessageResult(AppendMessageStatus.PUT_OK, wroteOffset, msgLen, msgId, msgInner.getStoreTimestamp(), queueOffset, CommitLog.this.defaultMessageStore.now() - beginTimeMills);
...
switch (tranType) {
	case MessageSysFlag.TRANSACTION_NOT_TYPE:
  //默认情况tranType等于0,所以这里也会被执行
  //这里主要用来设置QUEUEOFFSET
	case MessageSysFlag.TRANSACTION_COMMIT_TYPE:
		CommitLog.this.topicQueueTable.put(key, ++queueOffset);
}
```

---

