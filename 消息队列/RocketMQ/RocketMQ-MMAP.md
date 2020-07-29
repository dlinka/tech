示例

```java
File file = new File("test.txt");
FileChannel channel = new RandomAccessFile(file, "rw").getChannel();
MappedByteBuffer mappedByteBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 1000);
for (int i = 0; i < 1000; i++){
    mappedByteBuffer.put((byte)i);
}
mappedByteBuffer.position(0);
for (int i = 0; i < 1000; i++){
    System.out.println(mappedByteBuffer.get());
}
```

1.进入FileChannelImpl#map

    try {
        //native函数map0完成内存映射
        addr = map0(imode, mapPosition, mapSize);
    } catch (OutOfMemoryError x) {
        //执行一次GC
        System.gc();
        try {
            //让GC线程执行完成
            Thread.sleep(100);
        } catch (InterruptedException y) {
            Thread.currentThread().interrupt();
        }
        try {
            //在执行一次
            addr = map0(imode, mapPosition, mapSize);
        } catch (OutOfMemoryError y) {
            // After a second OOME, fail
            throw new IOException("Map failed", y);
        }
        ...
        //DirectByteBuffer实例
        return Util.newMappedByteBuffer(isize, addr + pagePosition, mfd, um);
        ↓
        ↓
        //进入Util类的newMappedByteBuffer方法
        if (directByteBufferRConstructor == null)
            initDBBRConstructor();
        //利用java.nio.DirectByteBuffer的构造函数初始化
        dbb = (MappedByteBuffer)directByteBufferRConstructor.newInstance(new Object[] { new Integer(size), new Long(addr), fd, unmapper });

---
