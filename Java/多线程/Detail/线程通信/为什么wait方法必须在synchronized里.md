#### 为什么WAIT必须在同步块中(转载)
我们知道java的Object有wait和notify方法，如果要使用wait和notify的话，那么必须在synchronized块中，否则会抛出IllegalMonitorStateException。但是为什么必须在同步块中调用呢？直接wait，然后在notify不行吗？我一直存在这样的疑问，只到后来查到了Stack Overflow的一个回答，豁然开朗。大概翻译了下：
假设我们要自定义一个blocking queue，如果没有使用synchronized的话，我们可能会这样写：
class BlockingQueue {
    Queue<String> buffer = new LinkedList<String>();

    public void give(String data) {
        buffer.add(data);
        notify();                   // Since someone may be waiting in take!
    }

    public String take() throws InterruptedException {
        while (buffer.isEmpty())    // 不能用if，因为为了防止虚假唤醒
            wait();
        return buffer.remove();
    }
}

这段代码可能会导致如下问题：
    1.一个消费者调用take，发现buffer.isEmpty
    2.此时，在消费者调用wait之前，由于cpu的调度，消费者线程被挂起，生产者调用give，然后notify
    3.然后消费者调用wait (注意，由于错误的条件判断，导致wait调用在notify之后，这是关键)
    4.如果很不幸的话，生产者产生了一条消息后就不再生产消息了，那么消费者就会一直挂起，无法消费，造成死锁。

->解决这个问题的方法就是：总是让give/notify和take/wait为原子操作。
也就是说wait/notify是线程之间的通信，他们存在竞态，我们必须保证在满足条件的情况下才进行wait。换句话说，如果不加锁的话，那么wait被调用的时候可能wait的条件已经不满足了(如上述)。由于错误的条件下进行了wait，那么就有可能永远不会被notify到，所以我们需要强制wait/notify在synchronized中

Stack Overflow的原文链接：
https://stackoverflow.com/questions/2779484/why-must-wait-always-be-in-synchronized-block

