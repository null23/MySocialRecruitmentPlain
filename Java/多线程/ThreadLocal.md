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