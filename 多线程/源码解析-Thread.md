isInterrupted

```java
return isInterrupted(false);
↓
↓
//false不会清空线程的中断状态
private native boolean isInterrupted(boolean ClearInterrupted);
```

interrupted

```java
return currentThread().isInterrupted(true);
↓
↓
//true会清空线程的中断状态
private native boolean isInterrupted(boolean ClearInterrupted);
```

