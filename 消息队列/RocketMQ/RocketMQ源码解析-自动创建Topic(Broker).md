启动Broker

1.Broker启动执行BrokerStartup的main方法

    BrokerController brokerController = createBrokerController(args); //2
    start(brokerController); //3

2.进入BrokerController的createBrokerController方法

    final BrokerController controller = new BrokerController(brokerConfig, nettyServerConfig, nettyClientConfig, messageStoreConfig);
    ↓
    ↓
    this.topicConfigManager = new TopicConfigManager(this);
    ↓
    ↓
    //默认情况开启自动创建Topic功能
    if (this.brokerController.getBrokerConfig().isAutoCreateTopicEnable()) {
        //TBW102
        String topic = TopicValidator.AUTO_CREATE_TOPIC_KEY_TOPIC;
        TopicConfig topicConfig = new TopicConfig(topic);
        this.topicConfigTable.put(topicConfig.getTopicName(), topicConfig);
    }

3.进入BrokerStartup的start

```java
controller.start();
↓
↓
this.registerBrokerAll(true, false, true);
↓
↓
//topicConfigWrapper包含步骤2中创建的topicConfigTable
TopicConfigSerializeWrapper topicConfigWrapper = this.getTopicConfigManager().buildTopicConfigSerializeWrapper();
...
doRegisterBrokerAll(checkOrderConfig, oneway, topicConfigWrapper);
↓
↓
//进入doRegisterBrokerAll
List<RegisterBrokerResult> registerBrokerResultList = this.brokerOuterAPI.registerBrokerAll(
        this.brokerConfig.getBrokerClusterName(),
        this.getBrokerAddr(),
        this.brokerConfig.getBrokerName(),
        this.brokerConfig.getBrokerId(),
        this.getHAServerAddr(),
        topicConfigWrapper,
        this.filterServerManager.buildNewFilterServerList(),
        oneway,
        this.brokerConfig.getRegisterBrokerTimeoutMills(),
        this.brokerConfig.isCompressedRegister());
↓
↓
//Topic:TBW102这时候同步到NameServer
RegisterBrokerBody requestBody = new RegisterBrokerBody();
requestBody.setTopicConfigSerializeWrapper(topicConfigWrapper);
```

---

自定义的Topic消息到达Broker  

1.进入AbstractSendMessageProcessor的msgCheck

```java
TopicConfig topicConfig = this.brokerController.getTopicConfigManager().selectTopicConfig(requestHeader.getTopic());
if (null == topicConfig) {
    topicConfig = this.brokerController.getTopicConfigManager().createTopicInSendMessageMethod(requestHeader.getTopic(), requestHeader.getDefaultTopic(), RemotingHelper.parseChannelRemoteAddr(ctx.channel()), requestHeader.getDefaultTopicQueueNums(), topicSysFlag);
}
```

2.进入TopicConfigManager的createTopicInSendMessageMethod

```java
//创建自定义TopicConfig对象
topicConfig = new TopicConfig(topic);
...
if (topicConfig != null) {
    this.topicConfigTable.put(topic, topicConfig);
    //把Topic信息持久化到${user.home}/store/config/topics.json
    this.persist();
}
...
//同步到NameServer
if (createNew) {
    this.brokerController.registerBrokerAll(false, true, true);
}
```

---
