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
        //Node.EXCLUSIVE说明初始化Node时候是独占式的
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