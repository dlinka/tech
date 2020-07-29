1.ReputMessageService#run

```java
@Override
public void run() {
    while (!this.isStopped()) {
        try {
            //间隔一毫秒执行doReput方法
            Thread.sleep(1);
            this.doReput();
        } catch (Exception e) {
        }
    }
}
```

2.ReputMessageService#doReput

```java
//reputFromOffset代表CommitLog中已被读取的消息偏移量
SelectMappedBufferResult result = DefaultMessageStore.this.commitLog.getData(reputFromOffset); //3
if (result != null) {
  //循环读取写入的消息
  for (int readSize = 0; readSize < result.getSize() && doNext; ) {
    //解析topic、queueId等参数
    DispatchRequest dispatchRequest = DefaultMessageStore.this.commitLog.checkMessageAndReturnSize(result.getByteBuffer(), false, false);
    //获取消息长度
    int size = dispatchRequest.getMsgSize();
    DefaultMessageStore.this.doDispatch(dispatchRequest); //4
    //更新偏移量
    this.reputFromOffset += size;
    //更新下次循环位置
    readSize += size;
  }
}
```

3.CommitLog#getData

```java
return getData(offset, offset == 0);
↓
↓
//根据偏移量获取要读取的CommitLog
MappedFile mappedFile = mappedFileQueue.findMappedFileByOffset(offset, returnFirstOnNotFound);
if (mappedFile != null) {
    SelectMappedBufferResult result = mappedFile.selectMappedBuffer(pos);
    return result;
}
↓
↓
//获取CommitLog的position
int readPosition = getReadPosition();
if (pos < readPosition && pos >= 0) {
  //这里就是获取需要写入ConsumeQueue的ByteBuffer
	ByteBuffer byteBuffer = this.mappedByteBuffer.slice();
	byteBuffer.position(pos);
	int size = readPosition - pos;
	ByteBuffer byteBufferNew = byteBuffer.slice();
	byteBufferNew.limit(size);
  return new SelectMappedBufferResult(fileFromOffset + pos, byteBufferNew, size, this);
}
```

4.1DefaultMessageStore#doDispatch

```java
for (CommitLogDispatcher dispatcher : this.dispatcherList) {
	dispatcher.dispatch(req);
}
```

4.2CommitLogDispatcherBuildConsumeQueue#dispatch

```java
case MessageSysFlag.TRANSACTION_NOT_TYPE:
case MessageSysFlag.TRANSACTION_COMMIT_TYPE:
	DefaultMessageStore.this.putMessagePositionInfo(request);
```

4.3DefaultMessageStore#putMessagePositionInfo

```java
//根据topic、queueId获取ConsumeQueue文件
ConsumeQueue cq = findConsumeQueue(dispatchRequest.getTopic(), dispatchRequest.getQueueId());
cq.putMessagePositionInfoWrapper(dispatchRequest);
↓
↓
boolean result = putMessagePositionInfo(request.getCommitLogOffset(), request.getMsgSize(), request.getTagsCode(), request.getConsumeQueueOffset());
↓
↓
//CommitLogOffset、MessageSize、TagCode存入ByteBuffer
byteBufferIndex.putLong(offset);
byteBufferIndex.putInt(size);
byteBufferIndex.putLong(tagsCode);
...
return mappedFile.appendMessage(this.byteBufferIndex.array());
↓
↓
//写入ConsumeQueue
this.fileChannel.position(currentPos);
this.fileChannel.write(ByteBuffer.wrap(data));
```

