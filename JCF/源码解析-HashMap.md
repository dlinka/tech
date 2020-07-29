1.put方法

```java
inflateTable(threshold); //进入2
int hash = hash(key); //进入3
int i = indexFor(hash, table.length); //进入4
addEntry(hash, key, value, i); //进入5
```

2.inflateTable方法

```java
//初始化数组大小
//找到大于等于toSize的2的幂次方
//为什么初始化数组的容量必须为2的幂次方?参考下面4
int capacity = roundUpToPowerOf2(toSize);
table = new Entry[capacity];
```

2.1roundUpToPowerOf2方法

```java
//number减1是考虑临界值的情况
//使用左移操作使参数变大,这样才能得到大于等于toSize的2的幂次方
Integer.highestOneBit((number - 1) << 1)

//找到小于等于i的2的幂次方
public static int highestOneBit(int i) {
    i |= (i >>  1);
    i |= (i >>  2);
    i |= (i >>  4);
    i |= (i >>  8);
    i |= (i >> 16);
    return i - (i >>> 1);
}
↓
↓
//例如i=19,计算得到16
0001 0011
0000 1001 //i >> 1
0001 1011 //i = i | (i >> 1)
0000 0110 //i >> 2
0001 1111 //i = i | (i >> 2)
...
0000 1111 //i >>> 1
0001 0000 //return i - (i >>> 1)
```

3.hash方法

```java
int h = hashSeed;
h ^= k.hashCode();
h ^= (h >>> 20) ^ (h >>> 12);
return h ^ (h >>> 7) ^ (h >>> 4);
```

4.indexFor方法

```java
//length如果不是2的幂次方就会造成数组一些位置空置,空间浪费
return h & (length-1);

例如h等272727,length等于8
1010  1010  0111 //27的二进制
0000 0000 0111 //7的二进制
0000 0000 0111 //&运算结果等于7
```

5.addEntry方法

```java
//按照默认的规则HashMap中元素个数大于等于12个,并且插入的数组位置不为空就进行扩容
if ((size >= threshold) && (null != table[bucketIndex])) {
    resize(2 * table.length); //进入5.1
}
createEntry(hash, key, value, bucketIndex); //进入5.2
```

5.1resize方法

```java
//数组两倍扩容,默认newCapacity为32
Entry[] newTable = new Entry[newCapacity];
transfer(newTable, initHashSeedAsNeeded(newCapacity));

void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
假设rehash等于false
假设元素在原始数组7这个位置上(默认0-15)
则按照上面的移动方法,新元素移动的位置为7或者23(7+16)
从原始数组最先移动的元素放在新数组的链表最下面
```

5.2createEntry方法

```java
//插入的元素放入链表的头部
Entry<K,V> e = table[bucketIndex];
//然后移动链表
table[bucketIndex] = new Entry<>(hash, key, value, e);
```

---