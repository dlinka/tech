
    如果消息消费失败需要重试,RocketMQ将消息重新发送到Broker服务器,此时msgId不会发送变化,但该消息的offsetMsgId会发送变化,因为其存储在服务器中的位置发生了变化

---

1.进入MessageClientExt

    @Override
    public String getMsgId() {
        //如果存在就使用msgId返回
        //如果不存在就使用offsetMsgId返回
        String uniqID = MessageClientIDSetter.getUniqID(this);
        if (uniqID == null) {
            return this.getOffsetMsgId();
        } else {
            return uniqID;
        }
    }

---
