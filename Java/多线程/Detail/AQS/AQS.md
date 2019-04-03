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
![ReentrantLock内部类Sync-继承AQS抽象类](https://raw.githubusercontent.com/null23/picture/master/Thread/sync.png)
    这个内部类继承了一个抽象类AbstractQueuedSynchronizer，这个抽象类就是我们所说的AQS。
    