**current**

```java
if (UNSAFE.getInt(Thread.currentThread(), PROBE) == 0)
	localInit();
//单例
return instance;
```

**localInit**

```java
//probe用来多线程随机生成数组索引
int p = probeGenerator.addAndGet(PROBE_INCREMENT);
int probe = (p == 0) ? 1 : p;
//seed用来生成随机数的种子
long seed = mix64(seeder.getAndAdd(SEEDER_INCREMENT));
Thread t = Thread.currentThread();
UNSAFE.putLong(t, SEED, seed);
UNSAFE.putInt(t, PROBE, probe);
```

**nextInt**

```java
return mix32(nextSeed());
↓
↓
//nextSeed方法
Thread t; long r;
UNSAFE.putLong(t = Thread.currentThread(), SEED, r = UNSAFE.getLong(t, SEED) + GAMMA);
return r;
```

---

