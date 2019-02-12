#####多线程编程中常用的API
(记API是一件很蠢的事情)
    1.getId()
    获取该线程的标识符(Long型)，此id是唯一的

    2.currentThread()
    返回对当前正在执行的线程对象的引用

    3.getName()
    获取当前线程的名称，默认是自动生成的。若调用setName方法，可以修改当前线程的名称。
    要避免在线程已经开始执行后，再调用setName方法修改线程名称。

    4.getPriority()
    获取当前线程的优先级。
    说一下线程优先级的概念：
        Java线程优先级的范围是0-10，低于0或者高于10会报IllegalArgumentException的异常。
        每个线程都有一个优先级，在同一时刻，优先级高的线程优先启动。
        优先级高的线程能得到更多的计算资源。
        优先级高的线程并不一定最先运行结束，这取决于这个线程所处理的内容。
        优先级低的线程并不是非要等到优先级高的线程运行完再运行，因为多线程本来指的就是线程之间的来回切换与调度。

    5.isAlive()
    测试线程是否处于活动状态。如果线程已经启动且尚未终止，则为活动状态。

    6.interrupt()
    interrupt()的作用是中断本线程。
    本线程中断自己是被允许的；其它线程调用本线程的interrupt()方法时，会通过checkAccess()检查权限。这有可能抛出SecurityException异常。
    如果本线程是处于阻塞状态：调用线程的wait(), wait(long)或wait(long, int)会让它进入等待(阻塞)状态，或者调用线程的join(), join(long), join(long, int), sleep(long), sleep(long, int)也会让它进入阻塞状态。若线程在阻塞状态时，调用了它的interrupt()方法，那么它的“中断状态”会被清除并且会收到一个InterruptedException异常。例如，线程通过wait()进入阻塞状态，此时通过interrupt()中断该线程；调用interrupt()会立即将线程的中断标记设为“true”，但是由于线程处于阻塞状态，所以该“中断标记”会立即被清除为“false”，同时，会产生一个InterruptedException的异常。

        1、终止处于“阻塞状态”的线程
        通常，我们通过“中断”方式终止处于“阻塞状态”的线程。
        当线程由于被调用了sleep(), wait(), join()等方法而进入阻塞状态；若此时调用线程的interrupt()将线程的中断标记设为true。由于处于阻塞状态，中断标记会被清除，同时产生一个InterruptedException异常。将InterruptedException放在适当的位置就能终止线程，形式如下：
        @Override
        public void run() {
            try {
                while (true) {
                    // 执行任务...
                }
            } catch (InterruptedException ie) {  
                // 由于产生InterruptedException异常，退出while(true)循环，线程终止！
            }
        }
        说明：在while(true)中不断的执行任务，当线程处于阻塞状态时，调用线程的interrupt()产生InterruptedException中断。中断的捕获在while(true)之外，这样就退出了while(true)循环！
        注意：对InterruptedException的捕获务一般放在while(true)循环体的外面，这样，在产生异常时就退出了while(true)循环。否则，InterruptedException在while(true)循环体之内，就需要额外的添加退出处理。形式如下：

        @Override
        public void run() {
            while (true) {
                try {
                    // 执行任务...
                } catch (InterruptedException ie) {  
                    // InterruptedException在while(true)循环体内。
                    // 当线程产生了InterruptedException异常时，while(true)仍能继续运行！需要手动退出
                    break;
                }
            }
        }
        说明：上面的InterruptedException异常的捕获在whle(true)之内。当产生InterruptedException异常时，被catch处理之外，仍然在while(true)循环体内；要退出while(true)循环体，需要额外的执行退出while(true)的操作。

        2、终止处于“运行状态”的线程
        通常，我们通过“标记”方式终止处于“运行状态”的线程。其中，包括“中断标记”和“额外添加标记”。

        （1）通过“中断标记”终止线程
        形式如下：
        @Override
        public void run() {
            while (!isInterrupted()) {
                // 执行任务...
            }
        }
        说明：isInterrupted()是判断线程的中断标记是不是为true。当线程处于运行状态，并且我们需要终止它时，可以调用线程的interrupt()方法，使用线程的中断标记为true，即isInterrupted()会返回true。此时，就会退出while循环。
        注意：interrupt()并不会终止处于“运行状态”的线程！它会将线程的中断标记设为true。

        （2）通过“额外添加标记”。
        形式如下：
        private volatile boolean flag= true;
        protected void stopTask() {
            flag = false;
        }

        @Override
        public void run() {
            while (flag) {
                // 执行任务...
            }
        }
        说明：线程中有一个flag标记，它的默认值是true；并且我们提供stopTask()来设置flag标记。当我们需要终止该线程时，调用该线程的stopTask()方法就可以让线程退出while循环。
        注意：将flag定义为volatile类型，是为了保证flag的可见性。即其它线程通过stopTask()修改了flag之后，本线程能看到修改后的flag的值。

        综合线程处于“阻塞状态”和“运行状态”的终止方式，比较通用的终止线程的形式如下：
        @Override
        public void run() {
            try {
                // 1. isInterrupted()保证，只要中断标记为true就终止线程。
                while (!isInterrupted()) {
                    // 执行任务...
                }
            } catch (InterruptedException ie) {  
                // 2. InterruptedException异常保证，当InterruptedException异常产生时，线程被终止。
            }
        }
        三、终止线程的示例
        interrupt()常常被用来终止“阻塞状态”线程。参考下面示例：
        package com.demo.interrupt;
        public class MyThread extends Thread{
            public MyThread(String name) {
                super(name);
            }
            @Override
            public void run() {
                try {  
                    int i=0;
                    while (!isInterrupted()) {
                        Thread.sleep(100); // 休眠100ms
                        i++;
                        System.out.println(Thread.currentThread().getName()+" ("+this.getState()+") loop " + i);  
                    }
                } catch (InterruptedException e) {  
                    System.out.println(Thread.currentThread().getName() +" ("+this.getState()+") catch InterruptedException.");  
                }
            }
        }