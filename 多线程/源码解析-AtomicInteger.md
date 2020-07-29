属性

```java
//Unsafe实例
private static final Unsafe unsafe = Unsafe.getUnsafe();

//value属性在内存中的偏移量
private static final long valueOffset;
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}

//volatile保证可见性
private volatile int value;
```

1.AtomicInteger#getAndIncrement

```JAVA
return unsafe.getAndAddInt(this, valueOffset, 1);
```

2.Unsafe#getAndAddInt

```java
int expect;
do {
    expect = this.getIntVolatile(obj, valueOffset);
//1.判断主存中的值和expect值(预期值)是否相等
//2.如果相等更新主存中的值为expect+var
//3.如果不相等就继续循环,由于主存中的值被改变,getIntVolatile会重新从主存读取最新预期值
} while(!this.compareAndSwapInt(obj, valueOffset, expect, expect + var));
return expect;
```



