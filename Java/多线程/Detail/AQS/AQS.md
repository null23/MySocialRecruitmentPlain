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

    可以看到区别，NonFairSync就是直接调用compareAndSetState方法，而FairSync是调用了hasQueuedPredecessors判断了当前线程节点是否是CLH队列的头节点，然后再判断CAS修改状态是否成功。如果成功就把AQS的当前占有线程设置为当前线程，并且返回true。
    这里引申出了另一个概念，也就是AQS的核心概念，CLH队列，AQS的操作大多是围绕这个队列来实现的，等下就会讲CLH队列。

**这里要重新注意下acquire方法是啥时候调用的，是在lock方法中调用的：非公平锁是直接先尝试获取同步状态，不成功就调用acquire方法；公平锁是直接调用acquire方法，然后在acquire方法中获取同步状态。**

    下面从网上copy了两张图(来自 https://zhuanlan.zhihu.com/p/54297968 作者：养兔子的大叔)：
![非公平锁的流程图](https://raw.githubusercontent.com/null23/picture/master/Thread/nofair-process.jpg)
![公平锁的流程图](https://raw.githubusercontent.com/null23/picture/master/Thread/fair-process.jpg)
    

    
