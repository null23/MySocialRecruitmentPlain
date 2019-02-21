#### 1.JVM规范
    class加载过程
    class file大概结构
    invoke指令

#### 2.ServletW
    API
    Servlet生命周期
    异步Servlet

#### 3.JVM
    内存区域
        Java堆
        方法区
        Java方法栈
        本地方法栈
    GC
    代
        新生代
        Eden
        Survivor
        旧生代
        持久代
    不同类型GC
        Serial
        ParNew
        Parallel Scavenge
        Serial Old
        Parallel Old
        CMS
        G1
     Reference
        Strong
        Weak
        Soft
   JVM options
   OOM
   常用java性能诊断工具
        jps
        jstat
        jmap
        jstack
        jinfo
        jConsole
        jVisualVM
        BTrace

#### 4.Java Concurrent
    两个keyword
        synchronized
        volatile
    锁
    原子性/可见性
    并发相关的一些数据结构
        ConcurrentHashMap
        LinkedBlockQueue
        AtomicXxx
    Executor框架
    Future
    Java内存模型

#### 5.Java NIO
    Buffer
    Channel
    Selector

#### 6.集合类
    JDK内置的集合类的实现方式以及使用场景
    List
        ArrayList
        LinkedList
    Set
        HashSet
        LinkedSet
        TreeSet
    Map
        HashMap
        TreeMap
 等 

 #### 7.JDK
    （1）List、Map、Set实现类的源代码
    （2）ReentrantLock、AQS的源代码
    （3）AtomicInteger的实现原理，主要能说清楚CAS机制并且AtomicInteger是如何利用CAS机制实现的
    （4）线程池的实现原理
    （5）Object类中的方法以及每个方法的作用