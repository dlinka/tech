1.进入DefaultRequestProcessor的processRequest

    //处理Broker注册到NameServer
    case RequestCode.REGISTER_BROKER:
        Version brokerVersion = MQVersion.value2Version(request.getVersion());
        if (brokerVersion.ordinal() >= MQVersion.Version.V3_0_11.ordinal()) {
            return this.registerBrokerWithFilterServer(ctx, request); //2
        }

    //处理Producer从NameServer获取Topic信息
    case RequestCode.GET_ROUTEINFO_BY_TOPIC:
        return this.getRouteInfoByTopic(ctx, request); //3

2.1进入DefaultRequestProcessor的registerBrokerWithFilterServer

    RegisterBrokerResult result = this.namesrvController.getRouteInfoManager().registerBroker(
            requestHeader.getClusterName(),
            requestHeader.getBrokerAddr(),
            requestHeader.getBrokerName(),
            requestHeader.getBrokerId(),
            requestHeader.getHaServerAddr(),
            registerBrokerBody.getTopicConfigSerializeWrapper(),
            registerBrokerBody.getFilterServerList(),
            ctx.channel());
    ↓
    ↓
    this.createAndUpdateQueueData(brokerName, entry.getValue());
    ↓
    ↓
    List<QueueData> queueDataList = this.topicQueueTable.get(topicConfig.getTopicName());
    if (null == queueDataList) {
        queueDataList = new LinkedList<QueueData>();
        queueDataList.add(queueData);
        //Broker传过来的的Topic:TBW102会保存在这里
        this.topicQueueTable.put(topicConfig.getTopicName(), queueDataList);
    }

3.进入DefaultRequestProcessor的getRouteInfoByTopic

    TopicRouteData topicRouteData = this.namesrvController.getRouteInfoManager().pickupTopicRouteData(requestHeader.getTopic());
    ↓
    ↓
    //经过步骤2
    //Producer使用Topic:TBW102可以获取到Broker路由
    List<QueueData> queueDataList = this.topicQueueTable.get(topic);
    if (queueDataList != null) {
        topicRouteData.setQueueDatas(queueDataList);
        foundQueueData = true;
    }

---
