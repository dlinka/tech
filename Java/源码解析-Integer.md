```java
Integer i1 = Integer.valueOf(27);
Integer i2 = Integer.valueOf(27);
 //true
System.out.println(i1 == i2);

Integer i1 = Integer.valueOf(227);
Integer i2 = Integer.valueOf(227);
//false
System.out.println(i1 == i2);
```

1.Integer#valueOf

```java
//如果传入参数大于等于-128并且小于等于127,返回缓存
if (i >= IntegerCache.low && i <= IntegerCache.high)
    return IntegerCache.cache[i + (-IntegerCache.low)];
return new Integer(i);
```

