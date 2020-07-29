##### API

getHoldCount()

```java
Queries the number of holds on this lock by the current thread.
查询当前线程对该锁的保持次数
```

void lock()

```java
Acquires the lock.
Acquires the lock if it is not held by another thread and returns immediately, setting the lock hold count to one.
If the current thread already holds the lock then the hold count is incremented by one and the method returns immediately.
If the lock is held by another thread then the current thread becomes disabled for thread scheduling purposes and lies dormant until the lock has been acquired, at which time the lock hold count is set to one.
获取锁
如果没有其他线程持有锁,则获取锁后将立即返回,将锁保持计数设置为1
如果当前线程已经持有锁,则将锁保持计数加1,该方法立即返回
如果锁是由另一个线程持有的,则当前线程将出于线程调度目的而被禁用,并将处于休眠状态,直到获取了锁为止,而在获取锁时会将锁保持计数设置为1
```

boolean tryLock()

```java
Acquires the lock only if it is not held by another thread at the time of invocation.
If the current thread already holds this lock then the hold count is incremented by one and the method returns true.
If the lock is held by another thread then this method will return immediately with the value false.
仅当调用时其他线程未持有锁时才能获取锁
如果当前线程已经持有锁,则将锁保持计数加1,并且该方法返回true
如果锁被其他线程持有,则此方法将立即返回false值
```

void lockInterruptibly()

```java
Acquires the lock unless the current thread is interrupted.
Acquires the lock if it is not held by another thread and returns immediately, setting the lock hold count to one.
If the current thread already holds this lock then the hold count is incremented by one and the method returns immediately.
If the lock is held by another thread then the current thread becomes disabled for thread scheduling purposes and lies dormant until one of two things happens:
The lock is acquired by the current thread or Some other thread interrupts the current thread.
If the lock is acquired by the current thread then the lock hold count is set to one.
If the current thread has its interrupted status set on entry to this method or is interrupted while acquiring the lock, then InterruptedException is thrown and the current thread's interrupted status is cleared.
除非当前线程被中断,否则获取锁
如果没有其他线程持有锁,则获取锁并立即返回,将锁保持计数设置为1
如果当前线程已经持有此锁,则锁保持计数加1,该方法将立即返回
如果锁是被其他线程锁持有,则出于线程调度目的,当前线程将被禁用,并处于休眠状态,直到发生以下两种情况之一：
锁由当前线程获得或者其他某个线程中断当前线程
如果当前线程获取了锁,则将锁保持计数设置为1
如果当前线程在进入此方法时已设置其中断状态或者在获取锁时被中断,则抛出InterruptedException并清除当前线程的中断状态.
```

