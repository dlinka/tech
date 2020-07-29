| 下标  | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| key   |      |      | 12   | null | 4    | null | 15   | 25   |      | null |
| value |      |      | v12  | v13  | v4   | v5   | v15  | v25  |      | v19  |

```java
为了有助于理解下面的源码,参考上面数组
假设hash是对10取余,3、5、9位置此刻被垃圾回收
set(15, XXX);
```

**1.set(ThreadLocal<?> key, Object value)**

```java
Entry[] tab = table;
int len = tab.length;
//i = 5
int i = key.threadLocalHashCode & (len-1);

for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
  ThreadLocal<?> k = e.get();
  //如果相等,直接赋值返回就行了
  if (k == key) {
  	e.value = value;
    return;
  }
  
  //如果这个位置被垃圾回收了
  if (k == null) {
  	replaceStaleEntry(key, value, i); //1.1
    return;
  }
}
```

**1.1replaceStaleEntry(ThreadLocal<?> key, Object value, int staleSlot)**

```java
Entry[] tab = table;
int len = tab.length;
Entry e;

int slotToExpunge = staleSlot;
//从5的位置往前直到数组没有被占用的位置为止
//1的位置结束循环
for(int i = prevIndex(staleSlot, len); (e = tab[i]) != null; i = prevIndex(i, len)) {
  if (e.get() == null)
    //slotToExpunge = 3
    slotToExpunge = i;
}

//从5的位置往后直到数组位置没有被占用为止
for (int i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
	ThreadLocal<?> k = e.get();
  if (k == key) {
    //6位置的值变成新值
  	e.value = value;
    //5位置和6位置互换
    tab[i] = tab[staleSlot];
    tab[staleSlot] = e;
    
    //从3位置开始清除陈旧条目
		int j = expungeStaleEntry(slotToExpunge); //1.1.1
    cleanSomeSlots(j, len); //1.1.2
    return;
  }
}
```

**1.1.1expungeStaleEntry(int staleSlot)**

```java
Entry[] tab = table;
int len = tab.length;

//3位置清除
tab[staleSlot].value = null;
tab[staleSlot] = null;
size--;

Entry e;
int i;
//从4的位置往前直到数组没有被占用的位置为止
for (i = nextIndex(staleSlot, len); (e = tab[i]) != null; i = nextIndex(i, len)) {
	ThreadLocal<?> k = e.get();
  //6位置被清除
  if (k == null) {
    e.value = null;
    tab[i] = null;
    size--;
  } else {
    int h = k.threadLocalHashCode & (len - 1);
    if (h != i) {
      //i = 7才会到这里
      //h = 5
    	tab[i] = null;
      //6位置此刻为null
      while (tab[h] != null)
      	h = nextIndex(h, len);
      //7位置元素赋值到6位置
      tab[h] = e;
    }
	}
}
//返回7
return i;
```

**1.1.2cleanSomeSlots(int i, int n)**

```java
boolean removed = false;
Entry[] tab = table;
int len = tab.length;
do {
    i = nextIndex(i, len);
    Entry e = tab[i];
    //这步会清空9位置
    if (e != null && e.get() == null) {
        //这里还会重新开始循环
        n = len;
        removed = true;
        //参考上面1.1.1
        i = expungeStaleEntry(i);
    }
} while ( (n >>>= 1) != 0); //默认2的4次方,这里会循环5次
return removed;
```

