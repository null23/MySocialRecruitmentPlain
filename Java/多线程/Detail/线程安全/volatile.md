### 操作系统内存模型

### 从操作系统抽象到JMM
    原子性
    可见性
    有序性

### MENU
volatile的的总栈加锁
load store内存屏障
写缓冲器
寄存器
mesi缓存一致性协议

### 原子性
    Java中i++不是一个原子性操作

### 可见性
    一个线程修改一个变量的值，另一个线程可以获取到被修改之后的正确的值。

### 有序性
    指令重排序

### volatile
    保证可见性，不保证原子性：多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值.
    例子：多线程场景下，多个线程对volatile修饰的变量i进行i++操作，结果发现i的值仍然不是预期的。
    原因：volatile确实能够保证当i的值被修改之后，立即刷新缓存行，让其他线程读取内存中的新值，保证在进行写入操作之后的可见性。
    但是如果这个写入的线程被阻塞，也就无法立即刷新缓存行，其他线程读到的值仍然是旧的值，因此对于原子性操作时，类似问题仍然存在。

    禁止指令重排序：happens-before原则

### 内存屏障
    通过volatile标记，可以解决编译器层面的可见性与重排序问题。而内存屏障则解决了硬件层面的可见性与重排序问题。
    violate可以触发内存屏障