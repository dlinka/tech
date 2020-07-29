Demo

```java
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer(CONSUMER_GROUP);
consumer.setNamesrvAddr("localhost:9876");
consumer.subscribe(TOPIC, "*");
//下面两个参数只会有一个线程处理消息
consumer.setConsumeThreadMax(1);
consumer.setConsumeThreadMin(1);
//最多一次拉取30条
consumer.setPullBatchSize(30);
//回调函数一次只处理一个消息
consumer.setConsumeMessageBatchMaxSize(1);
consumer.registerMessageListener(new MessageListenerConcurrently() {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
});
consumer.start();
```

实验

```java
DefaultMQPushConsumerImpl#pullMessage
     case FOUND:
     //这一行开启断点
     long prevRequestOffset = pullRequest.getNextOffset();
开启4个消费者
生产者发送200个消息
把第1个消费者的断点过掉,让2,3,4卡主
接下来会看到第1个消费者会消费掉所有的消息,2、3、4会打印1消费过的消息
```

![image](https://user-images.githubusercontent.com/4274041/99024293-39865780-25a1-11eb-8e44-dc363fc7f8c4.png)
![image](https://user-images.githubusercontent.com/4274041/99024334-53279f00-25a1-11eb-8ad6-307ab885c9ad.png)
![image](https://user-images.githubusercontent.com/4274041/99024350-6470ab80-25a1-11eb-9af3-d3d689373c11.png)
![image](https://user-images.githubusercontent.com/4274041/99024387-781c1200-25a1-11eb-9af7-939dd1b8489c.png)

结论

    如果消费者长时间没有返回ACK
    其他消费者会重复消费这个消息

---
