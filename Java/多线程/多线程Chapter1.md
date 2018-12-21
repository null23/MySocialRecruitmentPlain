#### 什么是线程？

#### 如何创建线程？

#### 线程的状态

#### 工作线程和守护线程
    Java中线程分为两种类型：用户线程和守护线程。通过Thread.setDaemon(false)设置为用户线程；通过Thread.setDaemon(true)设置为守护线程。如果不设置次属性，默认为用户线程。

    用户线程和守护线程的区别：
    1.主线程结束后用户线程还会继续运行,JVM存活；主线程结束后守护线程和JVM的状态又下面第2条确定。
    2.如果没有用户线程，都是守护线程，那么JVM结束（随之而来的是所有的一切烟消云散，包括所有的守护线程）。

    例1：
    public static void main(String[] args) throws InterruptedException {
        Thread mainThread = new Thread(() -> {
            System.out.println("running");
        });
        mainThread.setDaemon(true);
        mainThread.start();
        /**
         * main方法的执行属于用户线程
         * 所有用户线程执行结束之后，守护线程也会结束
         * 因此sleep一秒给了守护线程mainThread，在main方法的用户线程结束前的执行时间
         */
        Thread.sleep(1000L);
    }
    此时会打印出"running"的字样。
    如果我们不设置sleep的话，守护线程在main方法的用户线程结束时，还来不及执行就已经结束，因此不会打印出running。
    如果我们不设置或者设置setDaemon(false)，此时mainThread也是用户线程，main线程结束并不影响mainThread执行，因此如果我们在mainThread中写一个永真循环的话，控制台会一直不断地打印"running"。


