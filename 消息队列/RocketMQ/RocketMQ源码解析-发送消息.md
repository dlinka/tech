1.DefaultMQProducer#send

```java
this.defaultMQProducerImpl.send(msg);
↓
↓
return send(msg, this.defaultMQProducer.getSendMsgTimeout());
↓
↓
return this.sendDefaultImpl(msg, CommunicationMode.SYNC, null, timeout);
↓
↓
TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic()); //2
...
MessageQueue mqSelected = this.selectOneMessageQueue(topicPublishInfo, lastBrokerName); //3
...
sendResult = this.sendKernelImpl(msg, mq, communicationMode, sendCallback, topicPublishInfo, timeout - costTime); //4
```

2.DefaultMQProducerImpl#tryToFindTopicPublishInfo

```java
this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic);
↓
↓
return updateTopicRouteInfoFromNameServer(topic, false, null);
↓
↓
TopicPublishInfo publishInfo = topicRouteData2TopicPublishInfo(topic, topicRouteData);
↓
↓
TopicPublishInfo info = new TopicPublishInfo();
...
MessageQueue mq = new MessageQueue(topic, qd.getBrokerName(), i);
info.getMessageQueueList().add(mq);
```

3.DefaultMQProducerImpl#selectOneMessageQueue

```java
return this.mqFaultStrategy.selectOneMessageQueue(tpInfo, lastBrokerName);
↓
↓
return tpInfo.selectOneMessageQueue(lastBrokerName);
↓
↓
if (lastBrokerName == null) {
    return selectOneMessageQueue();
}
↓
↓
int index = this.sendWhichQueue.getAndIncrement();
int pos = Math.abs(index) % this.messageQueueList.size();
//负载均衡选择第二步中创建的MessageQueue
return this.messageQueueList.get(pos);
```

4.DefaultMQProducerImpl#sendKernelImpl

```java
//地址
String brokerAddr = this.mQClientFactory.findBrokerAddressInPublish(mq.getBrokerName());
...
//参数
SendMessageRequestHeader requestHeader = new SendMessageRequestHeader();
requestHeader.setProducerGroup(this.defaultMQProducer.getProducerGroup());
requestHeader.setTopic(msg.getTopic());
...
sendResult = this.mQClientFactory.getMQClientAPIImpl().sendMessage(
                        brokerAddr,
                        mq.getBrokerName(),
                        msg,
                        requestHeader,
                        timeout - costTimeSync,
                        communicationMode,
                        context,
                        this);
↓
↓
return sendMessage(addr, brokerName, msg, requestHeader, timeoutMillis, communicationMode, null, null, null, 0, context, producer);
↓
↓
SendMessageRequestHeaderV2 requestHeaderV2 = SendMessageRequestHeaderV2.createSendMessageRequestHeaderV2(requestHeader);
request = RemotingCommand.createRequestCommand(msg instanceof MessageBatch ? RequestCode.SEND_BATCH_MESSAGE : RequestCode.SEND_MESSAGE_V2, requestHeaderV2);
...
return this.sendMessageSync(addr, brokerName, msg, timeoutMillis - costTimeSync, request);
↓
↓
//发送请求
RemotingCommand response = this.remotingClient.invokeSync(addr, request, timeoutMillis);
return this.processSendResponse(brokerName, msg, response);
```

---
