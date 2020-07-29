示例

    DefaultMQProducer producer = new DefaultMQProducer("new_topic_group");
    producer.setNamesrvAddr("localhost:9876");
    producer.start();
    SendResult sendResult = producer.send(new Message("new_topic", "",  message.getBytes()));
    producer.shutdown();

1.进入DefaultMQProducer的send

    return this.defaultMQProducerImpl.send(msg);

2.进入DefaultMQProducerImpl的send

    return send(msg, this.defaultMQProducer.getSendMsgTimeout());
    ↓
    ↓
    return this.sendDefaultImpl(msg, CommunicationMode.SYNC, null, timeout);
    ↓
    ↓
    TopicPublishInfo topicPublishInfo = this.tryToFindTopicPublishInfo(msg.getTopic());
    if (topicPublishInfo != null && topicPublishInfo.ok()) {
    }
    //没有相应的Topic抛异常
    throw new MQClientException("No route info of this topic: " + msg.getTopic() + FAQUrl.suggestTodo(FAQUrl.NO_TOPIC_ROUTE_INFO),
            null).setResponseCode(ClientErrorCode.NOT_FOUND_TOPIC_EXCEPTION);
            
3.进入DefaultMQProducerImpl的tryToFindTopicPublishInfo
    
    //topicPublishInfoTable作用是保存所有Topic的相关信息
    //初始化topicPublishInfoTable的时候会添加Topic:TBW102
    TopicPublishInfo topicPublishInfo = this.topicPublishInfoTable.get(topic);
    if (null == topicPublishInfo || !topicPublishInfo.ok()) {
        //根据自定义的Topic从NameServer获取信息
        this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic);
        topicPublishInfo = this.topicPublishInfoTable.get(topic);
    }

    if (topicPublishInfo.isHaveTopicRouterInfo() || topicPublishInfo.ok()) {
        return topicPublishInfo;
    } else {
        //没获取到再使用Topic:TBW102获取
        this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic, true, this.defaultMQProducer); //4
        topicPublishInfo = this.topicPublishInfoTable.get(topic);
        return topicPublishInfo;
    }
    
4.进入MQClientInstance的updateTopicRouteInfoFromNameServer

    TopicRouteData topicRouteData;
    if (isDefault && defaultMQProducer != null) {
        topicRouteData = this.mQClientAPIImpl.getDefaultTopicRouteInfoFromNameServer(defaultMQProducer.getCreateTopicKey(), 1000 * 3); //5
    } else {
        topicRouteData = this.mQClientAPIImpl.getTopicRouteInfoFromNameServer(topic, 1000 * 3);
    }
    ...
    TopicPublishInfo publishInfo = topicRouteData2TopicPublishInfo(topic, topicRouteData);
    publishInfo.setHaveTopicRouterInfo(true);
    ...
    if (impl != null) {
        //更新topicPublishInfoTable
        impl.updateTopicPublishInfo(topic, publishInfo); //6
    }

5.进入MQClientAPIImpl的getDefaultTopicRouteInfoFromNameServer

    return getTopicRouteInfoFromNameServer(topic, timeoutMillis, false);
    ↓
    ↓
    //发送到NameServer
    RemotingCommand request = RemotingCommand.createRequestCommand(RequestCode.GET_ROUTEINFO_BY_TOPIC, requestHeader);
    RemotingCommand response = this.remotingClient.invokeSync(null, request, timeoutMillis);
    
6.进入DefaultMQProducerImpl的updateTopicPublishInfo

    //将自己创建的topic放入topicPublishInfoTable中
    TopicPublishInfo prev = this.topicPublishInfoTable.put(topic, info);
    
---
    






