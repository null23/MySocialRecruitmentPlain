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
![非公平锁的tryAcquire方法](https://raw.githubusercontent.com/null23/picture/master/Thread/nonfair-tryAcquire.png)
![公平锁的tryAcquire方法](https://raw.githubusercontent.com/null23/picture/master/Thread/fair-tryAcquire.png)

    可以看到区别，NonFairSync就是直接调用compareAndSetState方法，而FairSync是调用了hasQueuedPredecessors判断了当前线程节点是否是CLH队列的头节点(也就是之前有没有其他线程已经在等待获取同步资源，hasQueuedPredecessors方法返回false，说明之前没有其他线程在等待同步资源了，说明当前节点是头节点)，然后再判断CAS修改状态是否成功。如果成功就把AQS的当前占有线程设置为当前线程，并且返回true。
    这里引申出了另一个概念，也就是AQS的核心概念，CLH队列，AQS的操作大多是围绕这个队列来实现的，等下就会讲CLH队列。

**这里要重新注意下acquire方法是啥时候调用的，是在lock方法中调用的：非公平锁是直接先尝试获取同步状态，不成功就调用acquire方法；公平锁是直接调用acquire方法，然后在acquire方法中获取同步状态。**

    下面从网上copy了两张流程图(来自 https://zhuanlan.zhihu.com/p/54297968 作者：养兔子的大叔)：
![非公平锁的流程图](https://raw.githubusercontent.com/null23/picture/master/Thread/nofair-process.jpg)
![公平锁的流程图](https://raw.githubusercontent.com/null23/picture/master/Thread/fair-process.jpg)
    
    从上边已经看到了，公平锁和非公平锁的区别，目前有两个：
    1.公平锁是先把当前线程加入CLH队列，然后根据CLH队列的节点出队情况分配共享资源；非公平锁是直接先抢占，抢占失败再把当前线程加入CLH队列
    2.对于tryAcquire方法的实现不同：对于公平锁，当判断到锁状态字段state == 0 时，不会立马将当前线程设置为该锁的占用线程，而是去判断是在此线程之前是否有其他线程在等待这个锁（执行hasQueuedPredecessors()方法），如果是的话，则该线程会加入到等待队列中，进行排队（FIFO，先进先出的排队形式）。这也就是为什么FairSync可以让线程之间公平获得该锁。

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

#### acquire
```
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

//不断地获锁操作
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

//队尾添加当前线程节点
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;

        //快速尝试
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}

//加入队尾
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
    从上边可以看出，addWaiter方法是向队列尾部添加一个节点，通过调用compareAndSetTail方法和enq方法来实现的，若compareAndSetTail方法执行失败，则调用enq方法执行队尾添加节点的操作。
    附上一篇关于此操作的链接：https://www.jianshu.com/p/c806dd7f60bc

    当addWaiter方法执行成功，也就是添加节点成功，就会调用acquiredQueued方法进行不断地获锁操作。

```
//不断地获锁操作
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
    可以看到，这里是一个死循环，不断的判断当前节点是不是头节点head，并且不断的尝试获取锁。
    如果是，就把head节点的下个节点设置为新的head节点，并且把头节点head的指向为空(帮助gc垃圾回收)。
    这里有个问题，如果当前节点一直无法获取同步状态，那他将一直保持自旋状态吗？
    我们来看下shouldParkAfterFailedAcquire方法:

```
CANCELLED：值为1，表示线程的获锁请求已经“取消”
SIGNAL：值为-1，表示该线程一切都准备好了,就等待锁空闲出来给我
CONDITION：值为-2，表示线程等待某一个条件（Condition）被满足
PROPAGATE：值为-3，当线程处在“SHARED”模式时，该字段才会被使用上（在后续讲共享锁的时候再细聊）

//检查当前节点是否应该阻塞
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL)
        return true;
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```
    先给个链接，感谢大神：https://blog.csdn.net/chen77716/article/details/6641477
**规则1：当一个新的Node节点被初始化的时候，这个node的waitStatus是默认为0的。**
**规则2：在shouldParkAfterFailedAcquire这个方法中，我们获取而了当前节点的前缀节点pred，然后获取了pred的waitStatus，我们根据当前ws=0，就走到了第二个if的else分支，把前缀节点pred的waitStatus设置为SINGNAL**
**规则3：如果前继的节点状态为SIGNAL，表明当前节点需要unpark，则返回成功，此时acquireQueued方法的第12行（parkAndCheckInterrupt）将导致线程阻塞**
**规则4：如果前继节点状态为CANCELLED(ws>0)，说明前置节点已经被放弃，则回溯到一个非取消的前继节点，返回false，acquireQueued方法的无限循环将递归调用该方法，直至规则1返回true，导致线程阻塞**
**规则5：如果前继节点状态为非SIGNAL、非CANCELLED，则设置前继的状态为SIGNAL，返回false后进入acquireQueued的无限循环，与规则2同**

    下边来一个流程图(来自 https://zhuanlan.zhihu.com/p/54297968 作者：养兔子的大叔)：
![当前线程是否应该阻塞的流程图](https://raw.githubusercontent.com/null23/picture/master/Thread/node-status-process.jpg)
    
