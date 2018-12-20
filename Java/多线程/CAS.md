#### CAS(Compare And Swap)
   ##### 什么是CAS
    在CAS中有三个参数：内存值V、旧的预期值A、要更新的值B，当且仅当内存值V的值等于旧的预期值A时才会将内存值V的值修改为B，否则什么都不干。其伪代码如下：
    if(this.value == A)
    {
        this.value = B
        return true;
    } 
    else
    {
        return false;
    }

   ##### Example
    看看AtomicInteger如何实现并发下的累加操作：
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

    假设线程A和线程B同时执行getAndAdd操作（分别跑在不同CPU上）：
    AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程A和线程B各自持有一份value的副本，值为3。
    线程A通过getIntVolatile(var1, var2)拿到value值3，这时线程A被挂起。
    线程B也通过getIntVolatile(var1, var2)方法获取到value值3，运气好，线程B没有被挂起，并执行compareAndSwapInt方法比较内存值也为3，成功修改内存值为2。
    这时线程A恢复，执行compareAndSwapInt方法比较，发现自己手里的值(3)和内存的值(2)不一致，说明该值已经被其它线程提前修改过了，那只能重新来一遍了。
    重新获取value值，因为变量value被volatile修饰，所以其它线程对它的修改，线程A总是能够看到，线程A继续执行compareAndSwapInt进行比较替换，直到成功。
