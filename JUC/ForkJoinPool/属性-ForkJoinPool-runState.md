### runState

**int**

|                       | **二进制**                              | 备注     |
| --------------------- | --------------------------------------- | -------- |
| **RSLOCK(1)**         | **00000000 00000000 00000000 00000001** |          |
| **RSIGNAL(1<<1)**     | **00000000 00000000 00000000 00000010** |          |
| **STARTED(1<<2)**     | **00000000 00000000 00000000 00000100** |          |
| **STOP(1<<29)**       | **00100000 00000000 00000000 00000000** |          |
| **TERMINATED(1<<30)** | **01000000 00000000 00000000 00000000** |          |
| **SHUTDOWN(1 << 31)** | **10000000 00000000 00000000 00000000** | **负数** |

**lockRunState**

```java
private int lockRunState() {
    int rs;
    return ((((rs = runState) & RSLOCK) != 0 ||
             !U.compareAndSwapInt(this, RUNSTATE, rs, rs |= RSLOCK)) ?
            awaitRunStateLock() : rs);
}
↓
↓
int rs = runState;
//查看当前是否已经被锁
if((rs & RSLOCK) != 0){
  return awaitRunStateLock();
}
//没有就尝试上锁
if(!U.compareAndSwapInt(this, RUNSTATE, rs, rs |= RSLOCK)){
  return awaitRunStateLock();
}
//上锁成功
return rs;
```

---

