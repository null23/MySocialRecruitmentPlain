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
            //如果head节点的下一个节点它是null或者已经被cancelled了（status>0）
            //那么就从队列的尾巴往前找，找到一个最前面的并且状态不是cancelled的线程
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

### 关于头节点和锁释放的关系
    1.头节点是正在持有锁的节点，只不过是其thread属性在tryAcquire成功的时候设置为空了，并不代表头节点没有任何线程占有，这是个误区，需要注意

    2.头节点的后继节点，也就是CLH队列的第二个节点，当自旋的时候发现头节点获取锁了，waitStatus状态=-1的时候，会阻塞
***这里有个问题，为什么头节点的状态一定是-1？***
    其实不一定，加入CLH队列只有一个节点。
    但是别忘了，在多个节点的情况下，后续节点会把前缀节点的waitStatus>0的，改为waitStatus=-1

    举个例子，一个新的CLH队列，有两个节点，head节点属于Thread0(但是head的thread属性已经被清除)，Thead0已经获取到锁，Thread1刚进入队列。
    第二个节点在进行第一次自旋的时候，会把head节点的waitStatus从初始化的0修改为-1。然后第二个节点在第二次自旋的时候发现head节点的waitStatus=-1，进入阻塞状态。
    现在第三个节点来了，第三个节点第一次自选的时候，发现第二个节点的waitStatus=0，然后CAS修改成-1.然后第三个节点在第二次自旋的时候，会进入阻塞状态
    ....

    3.当头节点释放锁，刚才说到第二个节点是阻塞的，所以head节点就会唤醒后继节点(其实就是第二个节点，如果第二个节点已经取消获锁，waitStatus=canceled=1，那么就从队尾开始找，找到一个waitStatus不大于0的节点。为啥是不大于0呢？因为如果只有两个节点的话，第二个节点的waitStatus并没有第三个节点来修改成-1，而是保持着初始化时候的0，所以waitStatus=0的节点也是可以被唤醒的)，然后第二个节点继续执行自旋操作，也就是acquire方法的自旋操作，这个自旋操作先判断当前节点的前缀节点是不是head，然后对应的tryAcquire(公平非公平的模板设计模式实现的方法)使用CAS获锁
    如果获锁成功，就把当前节点设为新的head节点，并清除thread和prev属性
***这里有个问题，为啥要用tryAcquire来让第二个节点获锁？按说在head释放锁之后，不就应该第二个节点直接就能获取锁了？***
    nonono，加入在第二个节点获锁的时候，又来了个新的获锁请求，如果是非公平锁，是不会先直接就加入到CLH队列中的，而是先尝试获锁，因此第二个节点仍然后可能获锁失败。
    而公平锁，会根据hasQueuedPredecessors判断之前是否有等待的线程，如果有，就会加入CLH队列。比如第二个节点在等待，那新的获锁请求只能往后稍稍加入CLH队列了。
    

### lockfree名词解释
    循环+CAS