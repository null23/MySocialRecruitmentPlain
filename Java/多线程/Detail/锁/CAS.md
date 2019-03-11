#### CAS(Compare And Swap)
   ##### 什么是CAS
    首先，CAS的意义是让我们知道当前的线程的缓存中对应的主存中的值，是否被其他线程修改过，如果修改过，那么应该重新读取主存中的值，如果没有修改过，那我们就可以放心的对缓存中的值进行操作并且更新到主存中。
    这里的重点是：如何判断出一个值是否被其他线程修改过？

    在CAS中有三个参数：内存值V、旧的预期值A、要更新的值B，当且仅当内存值V的值等于旧的预期值A时才会将内存值V的值修改为B，否则什么都不干。

    举个例子：
    当前时刻，i=99，A线程和B线程同时对i进行i++操作，如果A线程++执行完了，缓存的值i=100还没有及时更新到主存里，那么B线程此时从主存获取到的值仍然是99，那么最后的结果是100，而不是101。
    如果我们使用了CAS机制来完成这件事情呢？
        1.A线程i++，读取主存中的值为99
        2.很巧的是，B线程i++，也读取主存中的值为99
        3.按照以往，A线程更新主存为100，由于未保证对B线程的可见性(当然，CAS也不能保证可见性，他只是添加了一个校验罢了，可以理解为一个条件变量)，B线程由于之前读到的99，更新主存为100.
        4.使用CAS：B线程会在更新之前，比较B线程缓存中的值99和主存中的100，发现不相等，更新失败，继续校验，也就是自旋，直到更新成功，所以说CAS是自旋锁。

   ##### Example
    看看AtomicInteger如何实现并发下的累加操作：
```
    public final int getAndAdd(int delta) {    
        return unsafe.getAndAddInt(this, valueOffset, delta);
    }
    //unsafe.getAndAddInt
    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
        return var5;
    }
```

    假设线程A和线程B同时执行getAndAdd操作（分别跑在不同CPU上）：
    AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程A和线程B各自持有一份value的副本，值为3。
    线程A通过getIntVolatile(var1, var2)拿到value值3，这时线程A被挂起。
    线程B也通过getIntVolatile(var1, var2)方法获取到value值3，运气好，线程B没有被挂起，并执行compareAndSwapInt方法比较内存值也为3，成功修改内存值为2。
    这时线程A恢复，执行compareAndSwapInt方法比较，发现自己手里的值(3)和内存的值(2)不一致，说明该值已经被其它线程提前修改过了，那只能重新来一遍了。
    重新获取value值，因为变量value被volatile修饰，所以其它线程对它的修改，线程A总是能够看到，线程A继续执行compareAndSwapInt进行比较替换，直到成功。


#### ABA问题
    根据以上第一个例子，注意一个地方，就是更新之前有可能线程被阻塞，比如B线程被阻塞，C线程执行i--操作并且执行成功，那么B线程阻塞完了进行比较并且更新的时候，发现内存值是99，可以合法更新成100。
    看起来好像没什么问题，但是别忘了，什么是CAS？我们使用CAS的目的是确保:我们知道其他线程修改了内存中的值，知道修改了之后再去比较才有意义，如果我们不知道其他线程修改过内存的值，CAS将会是毫无意义的，这违背了CAS的定义。
    如何解决？加入版本号