Tag过滤有两步  
一步是服务端过滤  
一步是客户端过滤

---

服务端过滤

1.进入DefaultMessageStore的getMessage方法

    //获取ConsumeQueue中的tag code
    long tagsCode = bufferConsumeQueue.getByteBuffer().getLong();
    //进入2.1
    if (messageFilter != null && !messageFilter.isMatchedByConsumeQueue(isTagsCodeLegal ? tagsCode : null, extRet ? cqExtUnit : null)) {
        ...
    }
    //进入2.2
    if (messageFilter != null && !messageFilter.isMatchedByCommitLog(selectResult.getByteBuffer().slice(), null)) {
        ...
    }

2.1进入DefaultMessageFilter的isMatchedByConsumeQueue方法

    //SubscriptionData就是消费者初始化的时候创建的关于tag的类
    //判断消费者创建的tags是否包含tag code
    return subscriptionData.getSubString().equals(SubscriptionData.SUB_ALL)
            || subscriptionData.getCodeSet().contains(tagsCode.intValue());
            
2.2进入DefaultMessageFilter的isMatchedByCommitLog方法

    //为什么这里直接返回true?我觉得是如果hash碰撞的情况,交给下面的客户端处理,可以总体上提高服务端的性能
    return true;

---

客户端过滤

1.进入DefaultMQPushConsumerImpl的pullMessage方法

    pullResult = DefaultMQPushConsumerImpl.this.pullAPIWrapper.processPullResult(pullRequest.getMessageQueue(), pullResult, subscriptionData);

2.进入PullAPIWrapper的processPullResult方法

    for (MessageExt msg : msgList) {
        if (msg.getTags() != null) {
            //判断当前消费者中的tags是否含有消息tag
            //为什么还需要客户端判断?因为服务端通过hash出来的,可能会有hash碰撞的问题
            if (subscriptionData.getTagsSet().contains(msg.getTags())) {
                msgListFilterAgain.add(msg);
            }
        }
    }
    
---
