#### IO
 - NIO = New Input Output / Non Blocking Input Output
 - BIO = Bocking Input Output

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
