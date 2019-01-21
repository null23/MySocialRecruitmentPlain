##### 1.ThreadLocal是干嘛用的
    1.ThreadLocal为变量在每个线程(线程隔离)中都创建了一个副本(副本这个词用的好)/线程局部变量。
    2.ThreadLocal提供了一种访问某个变量的特殊方式：访问到的变量属于当前线程，即保证每个线程的变量不一样，而同一个线程在任何地方拿到的变量都是一致的，这就是所谓的线程隔离。
    3.ThreadLocal是用来维护本线程的变量的，并不能解决共享变量的并发问题。ThreadLocal是各线程将值存入该线程的map中，以ThreadLocal自身作为key，需要用时获得的是该线程之前存入的值。如果存入的是共享变量，那取出的也是共享变量，并发问题还是存在的。
    4.使用场景：一个变量需要在一个线程内传递，并且多个线程并不共享。如果一个参数在很多方法(一个方法的调用链)都需要，为了避免这个参数传递多层，我们可以使用ThreadLocal来代替这个参数。
    5.注意：不要把共享变量放到ThreadLocal中，多个线程的ThreadLocal.get获取到的还是这个共享变量本身，还是有并发访问问题。所以保存到ThreadLocal之前，应该通过克隆或者new来创建一个新的对象，再进行保存。

##### 2.一个关于ThreadLocal的Demo
    public class ThreadLocalTest {
        /**
        * 关于这个例子的一些解释：
        * 实例化多个线程对ThreadLocal进行修改，就相当于多个请求过来，容器创建一个新的线程来处理这次的请求
        * 模拟了每个请求，就是每个线程(多个方法的引用链由一个线程处理)，对ThreadLocal进行修改
        * 类只会有一个，ThreadLocal作为类的成员变量也只有一个，但是类中的方法是被多个线程多次调用的
        */
        static class ResourceClass {
            public final static ThreadLocal<String> RESOURCE_1 =
                    new ThreadLocal<String>();
            public final static ThreadLocal<String> RESOURCE_2 =
                    new ThreadLocal<String>();
        }
        static class A {
            public void setOne(String value) {
                ResourceClass.RESOURCE_1.set(value);
            }
            public void setTwo(String value) {
                ResourceClass.RESOURCE_2.set(value);
            }
        }
        static class B {
            public void display() {
                System.out.println(ResourceClass.RESOURCE_1.get()
                        + ":" + ResourceClass.RESOURCE_2.get());
            }
        }
        public static void main(String []args) {
            final A a = new A();
            final B b = new B();
            for(int i = 0 ; i < 15 ; i ++) {
                final String resouce1 = "线程-" + i;
                final String resouce2 = " value = (" + i + ")";
                new Thread() {
                    @Override
                    public void run() {
                        try {
                            a.setOne(resouce1);
                            a.setTwo(resouce2);
                            b.display();
                        }finally {
                            ResourceClass.RESOURCE_1.remove();
                            ResourceClass.RESOURCE_2.remove();
                        }
                    }
                }.start();
            }
        }
    }

##### 3.ThreadLocal的原理
    ThreadLocalMap
    ThreadLocal其实是由一个个ThreadLocalMap维护的，每一个线程对应一个ThreadLocalMap的一个k-v。
    其中，key是ThreadLocal，value是用户存的值。
    ![ThreadLocal内存模型](https://raw.githubusercontent.com/null23/picture/master/Thread/ThreadLocal-JMM.png)

    ThreadLocal的扩容：
　　初始容量16，负载因子2/3，解决冲突的方法是再hash法，也就是：在当前hash的基础上再自增一个常量。

##### 4.ThreadLocal产生的内存泄露问题
    ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用引用他，那么系统gc的时候，这个ThreadLocal势必会被回收，这样一来，ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：
    Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value
    永远无法回收，造成内存泄露。

##### 5.ThreadLocalMap设计时的对上面问题的对策：
    ThreadLocalMap的getEntry函数的流程大概为：
    首先从ThreadLocal的直接索引位置(通过ThreadLocal.threadLocalHashCode & (table.length-1)运算得到)获取Entry e，如果e不为null并且key相同则返回e；
    如果e为null或者key不一致则向下一个位置查询，如果下一个位置的key和当前需要查询的key相等，则返回对应的Entry。否则，如果key值为null，则擦除该位置的Entry，并继续向下一个位置查询。在这个过程中遇到的key为null的Entry都会被擦除，那么Entry内的value也就没有强引用链，自然会被回收。仔细研究代码可以发现，set操作也有类似的思想，将key为null的这些Entry都删除，防止内存泄露。
    　　但是光这样还是不够的，上面的设计思路依赖一个前提条件：要调用ThreadLocalMap的getEntry函数或者set函数。这当然是不可能任何情况都成立的，所以很多情况下需要使用者手动调用ThreadLocal的remove函数，手动删除不再需要的ThreadLocal，防止内存泄露。所以JDK建议将ThreadLocal变量定义成private static的，这样的话ThreadLocal的生命周期就更长，由于一直存在ThreadLocal的强引用，所以ThreadLocal也就不会被回收，也就能保证任何时候都能根据ThreadLocal的弱引用访问到Entry的value值，然后remove它，防止内存泄露。
   