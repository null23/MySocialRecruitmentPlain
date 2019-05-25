### Node
    说到CLH队列，就必须说一下Node，因为CLH队列是基于AQS抽象类的内部的一个数据结构-Node构成的。
    Node包含以下属性：
        线程的2种等待模式：
            SHARED：表示线程以共享的模式等待锁（如ReadLock）
            EXCLUSIVE：表示线程以互斥的模式等待锁（如ReetrantLock），互斥就是一把锁只能由一个线程持有，不能同时存在多个线程使用同一个锁

        线程在队列中的状态枚举：
            CANCELLED：值为1，表示线程的获锁请求已经“取消”
            SIGNAL：值为-1，表示该线程一切都准备好了,就等待锁空闲出来给我
            CONDITION：值为-2，表示线程等待某一个条件（Condition）被满足
            PROPAGATE：值为-3，当线程处在“SHARED”模式时，该字段才会被使用上（在后续讲共享锁的时候再细聊）
            初始化Node对象时，默认为0

        成员变量：
            waitStatus：该int变量表示线程在队列中的状态，其值就是上述提到的CANCELLED、SIGNAL、CONDITION、PROPAGATE
            prev：该变量类型为Node对象，表示该节点的前一个Node节点（前驱）
            next：该变量类型为Node对象，表示该节点的后一个Node节点（后继）
            thread：该变量类型为Thread对象，表示该节点的代表的线程
            nextWaiter：该变量类型为Node对象，表示等待condition条件的Node节点（暂时不用管它，不影响我们理解主要知识点）

### CLH队列的构建
    CLH队列的由一个个Node节点构成，那么一个Node节点是如何初始化的呢？
    Node初始化的触发条件：
        在调用对应的模板模式实现的方法tryAcquire失败之后，会把当前线程作为一个Node加入CLH队列
        此时加入队列的时候，会调用addWaiter方法，把一个新的Node加入CLH队列。

        
        
```
//在compareAndSetState失败后，会调用acquire方法
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        //Node.EXCLUSIVE说明初始化Node时候是独占式的(acquireQueued方法下面会详细说明)
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

//addWaiter中Node的构造函数
Node(Thread thread, Node mode) {     // Used by addWaiter
    this.nextWaiter = mode;
    this.thread = thread;
}

//addWaiter方法，就是往CLH队列中添加一个新的Node节点到队列的尾部，并且把当前的节点tail更新成新的Node作为tail
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    //临时保存老的tail节点，由于加入新节点之后，老的tail肯定不是最后一个节点了，所以这里把老tail声明为pred
    Node pred = tail;
    //有可能tail还没有初始化，要初始化tail需要调用enq方法
    if (pred != null) {
        //新节点的前缀节点指向老的tail节点，也就是pred节点
        node.prev = pred;
        //如果CAS修改当前节点为tail失败(多个线程同时修改tail)，则使用enq方法自旋修改tail
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}

//自旋修改新的node节点为新的tail节点
private Node enq(final Node node) {
    //经典的lockfree算法：循环+CAS
    for (;;) {
        Node t = tail;
        //有可能tail还没有初始化，此时tail=head
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            //设置新的Node节点为此时CLH队列的tail
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

### CLH队列的元素自旋获锁
```
//采用自旋的方式，不断获取锁，自选可以被interrupted变量打断，然后进入阻塞状态(思考：阻塞的线程什么时候被唤醒呢)
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        //是否被打断自旋
        boolean interrupted = false;
        //永真循环，也就是自旋，但是会被interrtpt变量打断
        for (;;) {
            //当前节点的前缀节点
            final Node p = node.predecessor();
            //如果前缀节点是头结点，并且当前线程获锁成功，替换前缀节点为当前节点，并且删除head节点
            if (p == head && tryAcquire(arg)) {
                //设置当前线程对应的节点为头结点，并且清除当前节点的thread和prev属性
                setHead(node);
                //删除原来的头结点
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //是否打断自旋，使当前线程进入阻塞状态？
            //shouldParkAfterFailedAcquire方法负责根据Node状态判断是否需要打断，是就返回true(&&会导致短路)
            //parkAndCheckInterrupt方法负责打断当前线程执行(阻塞)
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }

    //清除新的头节点的thread和prev属性，也就是CLH队列的头节点除了初始化时一个新的节点，其他时候都是后续节点代替的
    //只是thread和prev属性被清除罢了，但是signal状态没变
    private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }

    /**
     * CANCELLED：值为1，表示线程的获锁请求已经“取消”
     * SIGNAL：值为-1，表示该线程一切都准备好了,就等待锁空闲出来给我
     * CONDITION：值为-2，表示线程等待某一个条件（Condition）被满足
     * PROPAGATE：值为-3，当线程处在“SHARED”模式时，该字段才会被使用上（在后续讲共享锁的时候再细聊）
     * 初始化Node对象时，默认为0
     */
    //根据pred状态判断当前线程是否需要阻塞
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        //如果前缀节点的状态=singal，就阻塞当前线程，
        if (ws == Node.SIGNAL)
            //返回true，会导致&&短路失败，执行parkAndCheckInterrupt方法
            return true;
        if (ws > 0) {
            //如果前缀节点状态是cancelled，就说明这个节点不需要获取锁了，直接扔了就行
            //这里会把所有的前缀节点以前的节点(包括前缀节点)，只要是状态是cancelled，全给清除
            do {
                //优先级：从右往左赋值
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            //新的前缀节点的next指针指向当前节点
            pred.next = node;
        } else {
            //初始化的Node的waitStatus=0，如果前缀节点的waitStatus=0，把其waitStatus更新成-1(signal)
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
}
```

### lockfree名词解释
    循环+CAS