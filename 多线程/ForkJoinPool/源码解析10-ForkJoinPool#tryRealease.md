**tryRelease(long c, WorkQueue v, long inc)**

```java
//ctl低32位
int sp = (int)c;
//计算栈顶WorkQueue激活后的scanState
//& ~INACTIVE = & 01111111 11111111 11111111 11111111
int vs = (sp + SS_SEQ) & ~INACTIVE;
Thread p;
if (v != null && v.scanState == sp) {
  	//ctl低32位为当前唤醒WorkQueue中持有的下一个待唤醒WorkQueue索引
    long nc = (UC_MASK & (c + inc)) | (SP_MASK & v.stackPred);
    if (U.compareAndSwapLong(this, CTL, c, nc)) {
        v.scanState = vs;
        if ((p = v.parker) != null)
            U.unpark(p);
        return true;
    }
}
return false;
```

