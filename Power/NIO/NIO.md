#### IO
 - IO有2个步骤，内核数据准备就绪，用户进程取数据2个步骤
 - NIO 的非阻塞说的是用户进程不必等待数据就绪，数据就绪后呢，用户进程取数据过程则是同步的
 - BIO 同步阻塞，是因为这两个步骤都会阻塞用户进程
 - NIO = New Input Output / Non Blocking Input Output
 - BIO = Bocking Input Output

IO多路复用和阻塞IO看起来貌似没有什么区别（两个阶段都是阻塞的），事实上还更差一些，因为这里需要使用两个系统调用(select和recvfrom)，
而阻塞IO只调用了一个系统调用(recvfrom)。但是，用select的优势在于它可以同时处理多个连接。
（所以，如果处理的连接数不是很高的话，使用select/poll/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，
select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。
#### NIO 和传统 BIO 的区别
 - NIO 面向缓冲区，BIO 面向流（字节流）
 - NIO 非阻塞，BIO 阻塞
 - NIO 有选择器，BIO 没有
 
#### BIO 流的过程
 - 先建立面向流的管道
 - 从管道中以流的形式读取数据（输入流/输出流）
 - 流是单向的
 
#### NIO 流的过程
 - 建立一条双向的 **通道**
 - 通道就相当于轨道，而缓冲区相当于火车

#### NIO - 同步非阻塞IO

#### 缓冲区

#### 通道

#### 选择器
