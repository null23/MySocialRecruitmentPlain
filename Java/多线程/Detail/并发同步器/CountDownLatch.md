###场景
    多个线程同时计算，最后合并多个线程的执行结果的时候可以用CountDownLatch

### 坑
    抛异常导致countDown方法没执行，计数器没+1，永远阻塞
    ps：如何设置超时时间？
    countDownLatch.await(20,TimeUnit.SECONDS);