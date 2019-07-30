## kafka，es里面我记得都有mmap，这个零拷贝是写时复制？
    不是，mmap就是零拷贝的一种实现方式
    linux的sendfile也是零拷贝的实现方式
    不需要进行内核缓冲区和用户缓冲区之间的数据拷贝，基于内存映射，减少了大量io操作，所以叫零拷贝。大概是这样子
    物理上通过DMA实现

## 写时复制
    写时复制就是说当你写一个数据更新的是复制好的一个副本数据，大量线程读数据读的是旧的数据，读写之间没有任何并发冲突。写的是副本数据，写完之后走volatile写回原数据。写时只有一个写请求可以写，内部使用reentrantlock加锁，但写的时候同事可以允许大量线程来并发读。

## Copy-on-Write
使用 Copy-on-Write 更多地体现的是一种延时策略，只有在真正需要复制的时候才复制，而不是提前复制好，同时 Copy-on-Write 还支持按需复制，所以 Copy-on-Write 在操作系统领域是能够提升性能的。相比较而言，Java 提供的 Copy-on-Write 容器，由于在修改的同时会复制整个容器，所以在提升读操作性能的同时，是以内存复制为代价的。

    https://blog.csdn.net/hzrandd/article/details/51025341  
    https://blog.csdn.net/zl1zl2zl3/article/details/89378682