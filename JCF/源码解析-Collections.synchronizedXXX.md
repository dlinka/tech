1.进入synchronizedList

```java
return (list instanceof RandomAccess ?
        new SynchronizedRandomAccessList<>(list) :
        new SynchronizedList<>(list));
↓
↓
//add方法
//实际上就是在调用add方法的时候使用synchronized代码块实现同步
synchronized (mutex) {list.add(index, element);}
```

