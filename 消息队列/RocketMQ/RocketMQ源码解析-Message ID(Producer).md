SendResult [sendStatus=SEND_OK, msgId=0A00C334E7FF18B4AAC251C47A56005A, offsetMsgId=0A00C33400002A9F0000000000083D14, messageQueue=MessageQueue [topic=TopicTest, brokerName=cyg.local, queueId=0], queueOffset=650]

---

msgId的生成规则  

1.进入DefaultMQProducer的send方法

    return this.defaultMQProducerImpl.send(msg);
    ↓
    ↓
    return send(msg, this.defaultMQProducer.getSendMsgTimeout());
    ↓
    ↓
    return this.sendDefaultImpl(msg, CommunicationMode.SYNC, null, timeout);
    ↓
    ↓
    sendResult = this.sendKernelImpl(msg, mq, communicationMode, sendCallback, topicPublishInfo, timeout - costTime);
    ↓
    ↓
    MessageClientIDSetter.setUniqID(msg);
    ↓
    ↓
    //PROPERTY_UNIQ_CLIENT_MESSAGE_ID_KEYIDX = "UNIQ_KEY";
    msg.putProperty(MessageConst.PROPERTY_UNIQ_CLIENT_MESSAGE_ID_KEYIDX, createUniqID());

2.进入MessageClientIDSetter的createUniqID方法

    sb.append(FIX_STRING); //3
    sb.append(UtilAll.bytes2string(createUniqIDBuffer())); //4
    return sb.toString();
    
3.进入MessageClientIDSetter的static初始化块

    FIX_STRING = UtilAll.bytes2string(tempBuffer.array());
    ↓
    ↓
    ByteBuffer tempBuffer = ByteBuffer.allocate(ip.length + 2 + 4);
    tempBuffer.put(ip);
    tempBuffer.putShort((short) UtilAll.getPid());
    tempBuffer.putInt(MessageClientIDSetter.class.getClassLoader().hashCode());

4.进入MessageClientIDSetter的createUniqIDBuffer方法

    ByteBuffer buffer = ByteBuffer.allocate(4 + 2);
    buffer.putInt((int) (System.currentTimeMillis() - startTime));
    buffer.putShort((short) COUNTER.getAndIncrement());

---

赋值msgId和offsetMsgId  

1.进入MQClientAPIImpl的processSendResponse方法

    SendMessageResponseHeader responseHeader = (SendMessageResponseHeader) response.decodeCommandCustomHeader(SendMessageResponseHeader.class);
    String uniqMsgId = MessageClientIDSetter.getUniqID(msg);
    SendResult sendResult = new SendResult(sendStatus, uniqMsgId, responseHeader.getMsgId(), messageQueue, responseHeader.getQueueOffset());

---
