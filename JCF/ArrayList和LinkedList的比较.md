调用add()方法的比较

| | 循环调用1千万次 |
| ---- | ---- |
| ArrayList | 39355 |
| LinkedList | 41928 |

    没有做实验之前,我一直认为ArrayList的扩容复制开销会大一点
    但是结果显示两者其实差不多,甚至ArrayList比LinkedList执行时间更少
    为什么会出现这种现象呢?LinkedList的add()方法会大量的new对象,其实这个开销也不低
    
    使用ArrayList的时候如果对容量有预估,最好指定一个capacity,减少扩容复制
    比如在我Mac上同样执行千万次add()方法,指定capacity比不指定capacity性能提高25%

---

遍历方法的比较

| \ | fori | forEach | iterator |
| ---- | ---- | ---- | ---- |
| ArrayList | 80 | 443 | 84 |
| LinkedList | 超时 | 477 | 202 |

    通过实验结果可知
    使用iterator遍历LinkedList最好
        for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
            it.next();
        }
    使用fori遍历ArrayList最好
    
    为什么上面结果会有超时的现象呢?
    进入LinkedList的源码,会发现node()方法中有导致执行大量的for循环
        return node(index).item;
        ↓
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }

---

remove()方法的比较

| \ | 1百万数据循环删除50万次 |
| --- | --- |
| ArrayList | 62228 |
| LinkedList | 超时 |

    ArrayList每一次删除都会执行System.arraycopy()方法
    我想对这个方法可以再次测试,维度是基于移动长度,得出的结果就可以判断删除元素的位置对remove()方法的影响
    
    为什么LinkedList会超时呢?
    查看源码发现remove()方法跟add()方法同样的逻辑,会执行大量的for循环

---
