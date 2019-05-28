### ReentrantLock
    在谈到同步问题的时候，我们往往会想到synchorized关键字，然后就是Lock。我们可以选择直接在代码块/方法体上加上synchorized关键字，也可以使用Lock的tryLock/unLock进行显示的加锁/解锁。
    Lock是一个接口，定义了几个核心的API:
![AQS核心API](https://raw.githubusercontent.com/null23/picture/master/Thread/lock.png)

    ReentrantLock就是Lock接口的实现类，核心方法lock/unLock如下：
```
    public void lock() {
        sync.lock();
    }
    public void unlock() {
        sync.release(1);
    }
```
    可以看到，lock/unLock方法都调用了一个叫做sync的成员变量，这个成员变量是ReentrantLock的内部类：
    Sync是一个抽象内部类，继承了一个抽象类AbstractQueuedSynchronizer，这个抽象类就是我们所说的AQS。
![ReentrantLock抽象内部类Sync-继承AQS抽象类](https://raw.githubusercontent.com/null23/picture/master/Thread/sync.png)
    
    从下图我们可以看出，ReentrantLock的抽象内部类Sync提供了两种实现：FairSync公平锁和NonfairSync非公平锁。
    决定当前Sync是FairSync/NonfairSync的是在实例化ReentrantLock的时候，构造函数中是true(公平锁)/false(非公平锁)，默认都是非公平锁。
![Sync的两个实现类](https://raw.githubusercontent.com/null23/picture/master/Thread/sync-lock.png)

**一句话总结上边的东西：**
**ReetrantLock实现了Lock接口,操作其成员变量sync这个AQS的子类,来完成锁的相关功能。而sync这个成员变量有2种形态：NonfairSync和FairSync.**

    接下来就是看FairSync/NonfairSync分别对lock方法的实现有哪些不同了
    可以发现，NonFairSync是先使用compareAndSetState获取同步状态的，若获取失败，则调用acquire方法；而FairSync方法是直接调用acquire方法。
![非公平锁的lock方法](https://raw.githubusercontent.com/null23/picture/master/Thread/nonfair-lock.png)
![公平锁的lock方法](https://raw.githubusercontent.com/null23/picture/master/Thread/fair-lock.png)

    acquire方法已经被抽象类AQS实现了，这个方法是干嘛的？
![AQS抽象类的acquire方法](https://raw.githubusercontent.com/null23/picture/master/Thread/acquire-aqs.png)

    可以看到，都调用了tryAcquire方法，NonFairSync和FairSync分别对其进行了不同的实现：
### 非公平锁的tryAcquire方法
```
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
        {
            throw new Error("Maximum lock count exceeded");
        }
        setState(nextc);
        return true;
    }
    return false;
}
```
### 公平锁的tryAcquire方法
```
//会在compareAndSetState失败之后，执行acquire方法的时候执行一次
//然后会在CLH队列自旋的时候执行
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        //判断之前还有节点等待不
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    //重入
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

    可以看到区别，NonFairSync就是直接调用compareAndSetState方法，而FairSync是调用了hasQueuedPredecessors判断了当前线程节点是否是CLH队列的头节点(也就是之前有没有其他线程已经在等待获取同步资源，hasQueuedPredecessors方法返回false，说明之前没有其他线程在等待同步资源了，说明当前节点是头节点)，然后再判断CAS修改状态是否成功。如果成功就把AQS的当前占有线程设置为当前线程，并且返回true。
    这里引申出了另一个概念，也就是AQS的核心概念，CLH队列，AQS的操作大多是围绕这个队列来实现的，等下就会讲CLH队列。

**这里要重新注意下acquire方法是啥时候调用的，是在lock方法中调用的：非公平锁是直接先尝试获取同步状态，不成功就调用acquire方法；公平锁是直接调用acquire方法，然后在acquire方法中获取同步状态。**

    下面从网上copy了两张流程图(来自 https://zhuanlan.zhihu.com/p/54297968 作者：养兔子的大叔)：
![非公平锁的流程图](https://raw.githubusercontent.com/null23/picture/master/Thread/nofair-process.jpg)
![公平锁的流程图](https://raw.githubusercontent.com/null23/picture/master/Thread/fair-process.jpg)
    
    从上边已经看到了，公平锁和非公平锁的区别，目前有两个：
    1.公平锁是先把当前线程加入CLH队列，然后根据CLH队列的节点出队情况分配共享资源；非公平锁是直接先抢占，抢占失败再把当前线程加入CLH队列
***2.对于tryAcquire方法的实现不同：对于公平锁，当判断到锁状态字段state == 0 时，不会立马将当前线程设置为该锁的占用线程，而是去判断是在此线程之前是否有其他线程在等待这个锁（执行hasQueuedPredecessors()方法），如果是的话，则该线程会加入到等待队列中，进行排队（FIFO，先进先出的排队形式）。这也就是为什么FairSync可以让线程之间公平获得该锁。***

```
//是否有其他线程先于当前线程等待获取锁，即判断队列中是否有前驱节点
//Head节点初始化时是一个空的Node节点，不存储任何线程相关的信息
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
    //1)如果h==t成立，队列为空，无前驱节点，返回false。
    //2）如果h!=t成立，从head节点的next是否为null，如果为null，返回true。什么情况下h!=t的同时h.next==null？？，
　　　　//有其他线程第一次正在入队时，可能会出现。见AQS的enq方法，compareAndSetHead(node)完成，还没执行tail = head语句时，此时tail=null,head=newNode,head.next-null。
    //3）如果h!=t成立，从head节点的next是否不为null，则判断是否是当前线程，如果是返回false，否则有前驱节点，返回true
}
```

### acquire
    请看CLH队列的文档

    下边来一个流程图(来自 https://zhuanlan.zhihu.com/p/54297968 作者：养兔子的大叔)：
![当前线程是否应该阻塞的流程图](https://raw.githubusercontent.com/null23/picture/master/Thread/node-status-process.jpg)
    
### unLock释放锁
```
 public final boolean release(int arg) {
     //解锁一次，释放一次重入锁的记录，直到释放完可以唤醒之后的线程
    if (tryRelease(arg)) {
        Node h = head;
        //理论上说waitStatus是不会=0的
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

//可重入的实现，每次释放锁都-1次获锁次数的记录，直到没有任何重入的记录
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}

//唤醒之后的线程
private void unparkSuccessor(Node node) {
    /*
        * If status is negative (i.e., possibly needing signal) try
        * to clear in anticipation of signalling.  It is OK if this
        * fails or if status is changed by waiting thread.
        */
    int ws = node.waitStatus;
    //头节点的状态肯定是-1，因为是之前获锁成功的节点代替的，肯定被修改过waitStatus=-1
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    ////如果s被取消，跳过被取消节点，直到找到一个没被取消的节点，唤醒这个节点，他会继续自旋
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```