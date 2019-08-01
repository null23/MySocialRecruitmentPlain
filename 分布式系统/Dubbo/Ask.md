作业1 自己画出RPC框架设计图！   
作业2 不看资料，自己手画Spring Cloud底层原理图

优秀作业，学员：顾飞点评：思考的非常深入，技术选型有自己的见解
作业1： 
1、动态代理层：RPC框架的一切调用细节都是动态代理方法里实现的。可选用Java原生Proxy类方法或Spring的Cglib实现。生成接口类的代理类，重写方法，实现真实调用。 

2、注册中心：提供者启动时进行注册，消费者对某些服务进行订阅，服务发现。
 ① zookeeper， dubbo service-A service-B provider cosumer provider consumer IP+port IP+port IP+port IP+port IP+port IP+port IP+port IP+port 如上图设计好的树状结构来存储
Providers启动时在Zookeeper中某个服务节点下创建临时子节点，节点名称使用IP、端口、权重等信息拼接字符串，这样实现服务注册。
Consumer监听某个服务节点服务的子节点，并且拉取子节点列表。实现服务订阅和发现。
当Provider上线或下线时，临时节点发生新增或删除。所以Consumer那边监听会收到通知，拉取最新的注册表数据，更新本地注册表缓存，这样实现服务发现。 
② 也可以采用Eureka的实现方式，采用纯内存ConcurrentHashMap来存储注册表，Provider启动时将IP、端口、权重等信息注册。实现服务注册。
Consumers定时去拉取注册表缓存在本地。实现服务发现。然后各服务采用定时心跳的方式来告诉注册中心自己是否存活。 实现故障感知。 
比如用Map<String,Map<String,Lease<InstanceInfo>>>这样的结构来保存，服务名为key，Value Map的key为实例ID，Lease中维护最近的心跳时间，InstanceInfo中保存实例的IP、端口等信息。这是内存注册表。 
并且可以加入多级缓存来优化并发读写冲突。 配置好合适的注册表拉取时间和心跳时间，比如30秒，则可以单机承载大规模系统日千万级的访问量。 
综上：Zookeeper方式显然更加实时感知集群变动。而Eureka的方式因为是定时拉取，所以消费端对集群变更感知会有一定延时性。

3、Cluster层：实现路由和负载均衡、集群事件处理，比如故障切换等。必备轮询、加权轮询、随机、加权随机、一致性Hash等路由或负载均衡方式。 
加权可以根据权重增加虚拟节点实现。一致性Hash，采用Hash环方式，并加入一定数量的虚拟节点来使节点更加均衡的分布在Hash环上，顺时针旋转时，继而请求分布更加均匀。可以采用ConcurrentSkipListMap来实现。 

4、数据发送部分：Exchange，Portocol、序列化反序列化机制、Transport网络IO。 Exchange封装为Request/Response，结合BlockingQueue将Netty异步通信转化同步。 
协议可以是HTTP、dubbo等 序列化加入多种选择，比如Json、Protobuf、Protostuff、Hessian、Kryo等、Java序列化等等。 

5、网络IO部分，可以使用BIO、NIO。BIO性能极差！推荐使用NIO框架Netty来实现，并处理好数据粘包问题。 
使用多路复用IO机制，服务端有一组Acceptor线程，ServerSocketChannel监听某个端口，而Acceptor线程通过多路复用器Selector来轮询监听ServerSocketChannel的Accept事件。
当客户端的netty请求连接时，此时会创建对应的SocketChannel。被Processor线程组通过Selector轮询监听，来处理实际的IO。
当客户端发来数据时，服务端SocketChannel触发事件，由Processor线程去解析请求，经历反序列化和Exchange、Protocol层层解析后，经由代理类找到对应的服务实现方法处理。再经过SocketChannel返回给客户端。 
客户端同样是processor线程组监听到SocketChannel的事件，读取响应给程序。 

综上所述： 划分层次的话，就是10层 Service、Config、Proxy、Registry、Cluster、Protocol、Exchage、Transport、Serialize、Monitor