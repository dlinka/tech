**属性**

```java
//ctl代表线程池的状态和线程数量
AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

//32 - 3 = 29
COUNT_BITS = Integer.SIZE - 3

//线程数量
//0001 1111 1111 1111 1111 1111 1111 1111
CAPACITY = (1 << COUNT_BITS) - 1

//1110 0000 0000 0000 0000 0000 0000 0000
RUNNING = -1 << COUNT_BITS
//0000 0000 0000 0000 0000 0000 0000 0000
SHUTDOWN = 0 << COUNT_BITS
//0010 0000 0000 0000 0000 0000 0000 0000
STOP = 1 << COUNT_BITS
```

**位运算方法**

```java
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

ctlOf

```java
//状态和数量
return rs | wc;
↓
↓
1110 0000 0000 0000 0000 0000 0000 0000
 |(或运算)
0000 0000 0000 0000 0000 0000 0000 0000
↓
↓
1110 0000 0000 0000 0000 0000 0000 0000
```

runStateOf

```java
return c & ~CAPACITY;
↓
↓
~CAPACITY
  1110 0000 0000 0000 0000 0000 0000 0000
&
	两个位都为1时,才为1
↓
↓
计算前3位的值
```

workerCountOf

```java
return c & CAPACITY;
↓
↓
CAPACITY
  0001 1111 1111 1111 1111 1111 1111 1111
&
	两个位都为1时,才为1
↓
↓
计算后29位的值
```



