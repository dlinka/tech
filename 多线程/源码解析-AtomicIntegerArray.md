**base**

```java
//返回指定数组的第一个元素地址相对于数组起始地址的偏移值
private static final int base = unsafe.arrayBaseOffset(int[].class);
```

**scale和shift**

```java
static {
  //返回指定类型数组的元素所占用的字节数
  //4
	int scale = unsafe.arrayIndexScale(int[].class);
  //2 = 31 - 29
	shift = 31 - Integer.numberOfLeadingZeros(scale);
}
↓
↓
//用在下面的byteOffset方法
2的shift次方等于scale
```

**byteOffset**

```java
//一个数学公式,通过下面的计算=i * scale + base,性能更好
//i << shift = i * 2的shift次方
//5 << 1 = 5 * 2的1次方
return ((long) i << shift) + base;
```

