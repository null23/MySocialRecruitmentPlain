## kafka，es里面我记得都有mmap，这个零拷贝是写时复制？
    不是，mmap就是零拷贝的一种实现方式
    linux的sendfile也是零拷贝的实现方式
    不需要进行内核缓冲区和用户缓冲区之间的数据拷贝，基于内存映射，减少了大量io操作，所以叫零拷贝。大概是这样子
    物理上通过DMA实现

    https://blog.csdn.net/hzrandd/article/details/51025341  
    https://blog.csdn.net/zl1zl2zl3/article/details/89378682