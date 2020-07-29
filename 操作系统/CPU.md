如何计算CPU个数

    CPU核总数 = CPU个数 * CPU核数  
    逻辑CPU总数 = CPU核总数 * 超线程数

Mac

    sysctl machdep.cpu
    ↓
    machdep.cpu.core_count: 2 //CPU核总数
    machdep.cpu.thread_count: 4 //逻辑CPU总数
    
Linux

    //查看CPU个数
    //physical id表示物理CPU的id
    cat /proc/cpuinfo|grep "physical id"|sort|uniq
    physical id     : 0
    physical id     : 1
    
    //查看CPU核数
    //cpu cores表示物理CPU上的CPU核数
    cat /proc/cpuinfo|grep "cpu cores"|sort|uniq
    cpu cores       : 4
    
    //查看CPU核总数
    //core id表示CPU核id
    cat /proc/cpuinfo|grep "core id"|sort|uniq
    core id         : 0
    core id         : 1
    core id         : 2
    core id         : 3
    core id         : 4
    core id         : 5
    core id         : 6
    core id         : 7
    
    //查看逻辑CPU数
    //siblings表示物理CPU上的逻辑CPU数
    cat /proc/cpuinfo|grep "siblings"|sort|uniq
    siblings        : 8
    
    //查看逻辑CPU总数
    //processor表示逻辑CPU的id
    cat /proc/cpuinfo|grep "processor"|sort|uniq
    processor       : 0
    processor       : 1
    processor       : 10
    processor       : 11
    processor       : 12
    processor       : 13
    processor       : 14
    processor       : 15
    processor       : 2
    processor       : 3
    processor       : 4
    processor       : 5
    processor       : 6
    processor       : 7
    processor       : 8
    processor       : 9
    
---
