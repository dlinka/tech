线程状态

NEW:A thread that has not yet started is in this state.

    //就是这个状态
    Thread thread = new Thread();

RUNNABLE:A thread executing in the Java virtual machine is in this state.
    
    //处于可运行状态的线程正在Java虚拟机中执行,但它可能正在等待来自操作系统(例如处理器)的其他资源
    A thread in the runnable state is executing in the Java virtual machine but it may be waiting for other resources from the operating system such as processor.
    通过这个定义知道Java里面的RUNNABLE状态应该是操作系统里面的RUNNING状态
    而且操作系统中的可运行状态(RUNNABLE)在Java里面也算是RUNNABLE状态
    
    其实在JVM里,线程执行I/O,他的状态也是RUNNABLE,因为这个线程一直在JVM中运行
    通过这点可以知道Java的线程状态和OS的线程状态是完全的两码事

BLOCKED

    线程进入锁池就进入这个状态

WAITING

    线程进入等待池就进入这个状态

TERMINATED

    线程执行结束

---
