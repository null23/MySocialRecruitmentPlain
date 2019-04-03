### ReetrantLock
    在谈到同步问题的时候，我们往往会想到synchorized关键字，然后就是Lock。我们可以选择直接在代码块/方法体上加上synchorized关键字，也可以使用Lock的tryLock/unLock进行显示的加锁/解锁。
    Lock是一个接口，定义了几个核心的API:

![https://github.com/null23/picture/blob/master/Thread/lock.png?raw=true](lock核心api)