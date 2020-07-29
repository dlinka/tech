1.DefaultMQPushConsumer#start

```java
this.defaultMQPushConsumerImpl.start();
↓
↓
case CREATE_JUST:
    //消费消息
    else if (this.getMessageListenerInner() instanceof MessageListenerConcurrently) {
        this.consumeMessageService = new ConsumeMessageConcurrentlyService(this, (MessageListenerConcurrently) this.getMessageListenerInner());
    }
    ...
    mQClientFactory.start();
↓
↓
//开启拉取消息线程
this.pullMessageService.start();
↓
↓
//PullMessageService#run
//pullRequestQueue是一个阻塞队列
PullRequest pullRequest = this.pullRequestQueue.take();
this.pullMessage(pullRequest);
↓
↓
DefaultMQPushConsumerImpl impl = (DefaultMQPushConsumerImpl) consumer;
impl.pullMessage(pullRequest);
↓
↓
//拉取后的回调处理
PullCallback pullCallback = new PullCallback() {
    @Override
    public void onSuccess(PullResult pullResult) {
    }
};
...
this.pullAPIWrapper.pullKernelImpl(
            pullRequest.getMessageQueue(),
            subExpression,
            subscriptionData.getExpressionType(),
            subscriptionData.getSubVersion(),
            pullRequest.getNextOffset(),
            this.defaultMQPushConsumer.getPullBatchSize(),
            sysFlag,
            commitOffsetValue,
            BROKER_SUSPEND_MAX_TIME_MILLIS,
            CONSUMER_TIMEOUT_MILLIS_WHEN_SUSPEND,
            CommunicationMode.ASYNC,
            pullCallback
        );
↓
↓
PullResult pullResult = this.mQClientFactory.getMQClientAPIImpl().pullMessage(
            brokerAddr,
            requestHeader,
            timeoutMillis,
            communicationMode,
            pullCallback);
↓
↓
case ASYNC:
    this.pullMessageAsync(addr, request, timeoutMillis, pullCallback);
↓
↓
this.remotingClient.invokeAsync(addr, request, timeoutMillis, new InvokeCallback() {
    @Override
    public void operationComplete(ResponseFuture responseFuture) {
        PullResult pullResult = MQClientAPIImpl.this.processPullResponse(response);
        //调用上面的回调处理
        pullCallback.onSuccess(pullResult);
    }
});
↓
↓
case FOUND:
    else {
        DefaultMQPushConsumerImpl.this.consumeMessageService.submitConsumeRequest(pullResult.getMsgFoundList(), processQueue, pullRequest.getMessageQueue(), dispatchToConsume);
        //间隔多场时间再次拉取消息
        if (DefaultMQPushConsumerImpl.this.defaultMQPushConsumer.getPullInterval() > 0) {
            DefaultMQPushConsumerImpl.this.executePullRequestLater(pullRequest, DefaultMQPushConsumerImpl.this.defaultMQPushConsumer.getPullInterval());
        } else {
            DefaultMQPushConsumerImpl.this.executePullRequestImmediately(pullRequest);
        }
    }
↓
↓
//一次处理多少消息
final int consumeBatchSize = this.defaultMQPushConsumer.getConsumeMessageBatchMaxSize();
if (msgs.size() <= consumeBatchSize) {
    //直接提交线程池
    this.consumeExecutor.submit(consumeRequest);
} else {
    //分批提交线程池
    for (int total = 0; total < msgs.size(); ) {
        List<MessageExt> msgThis = new ArrayList<MessageExt>(consumeBatchSize);
        for (int i = 0; i < consumeBatchSize; i++, total++) {
            if (total < msgs.size()) {
                msgThis.add(msgs.get(total));
            } else {
                break;
            }
        }
        ConsumeRequest consumeRequest = new ConsumeRequest(msgThis, processQueue, messageQueue);
        this.consumeExecutor.submit(consumeRequest);
    }
}    
```

---
