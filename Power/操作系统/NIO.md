### NIO在Java中的应用
    1.JVM只认识二进制的字节流，因此在ClassLoader加载Class的信息的时候，需要把类对应的.class文件转换成二进制byte字节流，作为参数传递给ClassLoader的defineClass方法(ClassLoader提供的方法，将二进制数组转换成Class类的实例)，进行类的实例化。
    这里如果是我们自己手写一个类加载器的话，可能会直接使用普通的IO流(BIO)进行操作。
    而Java本身的实现是采用NIO来实现的，提升了JVM进行类加载时候的性能。