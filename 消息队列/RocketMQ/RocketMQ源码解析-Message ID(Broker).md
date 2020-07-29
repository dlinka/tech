SendResult [sendStatus=SEND_OK, msgId=0A00C334F59A18B4AAC25F363E9D0000, offsetMsgId=0A00C33400002A9F0000000000084476, messageQueue=MessageQueue [topic=TopicTest, brokerName=cyg.local, queueId=2], queueOffset=653]

---

offsetMsgId的生成规则

1.进入MessageDecoder的decode方法

    int msgIDLength = storehostIPLength + 4 + 8;
    ByteBuffer byteBufferMsgId = ByteBuffer.allocate(msgIDLength);
    String msgId = createMessageId(byteBufferMsgId, msgExt.getStoreHostBytes(), msgExt.getCommitLogOffset());
    ↓
    ↓
    //msgId生成因子包含地址和偏移量
    input.put(addr);
    input.putLong(offset);
    return UtilAll.bytes2string(input.array());

---
