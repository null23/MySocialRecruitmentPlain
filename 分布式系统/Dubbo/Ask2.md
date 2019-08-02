## 1、画出公司的服务注册中心原理图！ 
公司采用zookeeper 集群部署，机器都是双核四G50G硬盘，可能配置一般，但是独立运行一个zk没问题吧 

1：Zookeeper leader和follower两种角色，公司在三台机器部署了三个zk，一个leader，2个follower 

2：leader负责注册服务，然后把数据同步给follower 

3：读取服务的时候可以通过zookeeper和foliower读取，follower和leader保证数据强一致性 

4：调用放监听zookeeper服务，一旦服务变更，zk会同步给服务更新注册列表信息到本地缓存 

数据一致性方面： 

1：zookeeper是CP模型，leader节点接收来自服务的所有数据，然后同步给follower节点（ms级别）； 

2：follower节点不接收，数据来自leader节点的同步。 

3：当leader节点故障的时候，会重新选举leader节点，这个时候zookeeper服务不可用，服务不能注册或者发现下线的服务，可能会停顿ms。 

4：当follower选取成leader时候，那么这个时候新的服务又可以注册，然后leader会将服务同步给follower,数据一致性很快得到保障,消费方面很快能够发现新上线的服务。 

高可用性方面： 
zookeeper集群毕竟是CP模型,可用性理论上很难保障,一般zk故障基本会引起可用性，导致线上出现故障

比如磁盘满了，zk故障，一般运维会做清磁盘处理或者扩容处理或者重启操作。 

时效性方面： 
zookeeper 目前没啥问题,正常情况很快可以感知服务的上线，服务的下线。 

容量方面： 
这块可以和高可用结合起来，确实容量出现问题，zk容易出现故障。目前的话大概有几百个实例向zk注册服务。 

再来梳理下另外一个注册中心,Eureka 虽然咱们没有使用 Eureka 集群模式 

1：有多个注册中心 Eureka集群部署Eureka-A、Eureka-B 

2：有个服务需要启动向Eureka-A注册服务，Eureka-A会将注册表信息同步给Eureka-B 

3：Eureka-A、Eureka-B 是peer-To-peer模式，同步的注册信息都是一致的 

4：服务会不断的发送给注册中心心跳，集群里任何一个Euerka实例接收到写请求之后，会自动同步给其他所有的Eureka实例。 

数据一致性方面： 
1：Eureka是peer模式，每个服务实例都是对等的，实例直接会不断同步数据。 

2：当其中Eureka实例故障的时候，可以从其他Eureka拉去服务注册表，但是这个过程中服务注册信息可能不是最新的，不能保证强一致性，但是保证最终一致性。 

时效性方面 eureka，默认配置非常坑，服务注册心跳需要30s，定时扫描注册中心需要60s，如果扫描的服务发现90s内没有心跳，发现实例需要移除， 定时扫描writeCache同步给readOnly需要30s，消费方定时拉去注册信息需要30s。

所以真实环境为了保证时效需要对参数进行调优。 比如这些参数： eureka.server.responseCacheUpdateIntervalMs = 3000 eureka.client.registryFetchIntervalSeconds = 3000 eureka.client.leaseRenewalIntervalInSeconds = 3 eureka.server.evictionIntervalTimerInMs=3000 eureka.instance.leaseExpirationDurationInSeconds = 6 

容量方面：
也很难支撑大规模的服务实例，因为每个eureka实例都要接受所有的请求，实例多了压力太大，扛不住，也很难到几千服务实例。 

提问： 老师，您好，有个问题关于zk可用性的。 zookeeper高可用很难保障，线上目前集群部署leader/follower方式，比如leader节点故障了，可能是由于磁盘满了导致故障， 那么如何保障不重启，清磁盘的方式来重新使用zk服务，这样可用性很难保障啊？

问题解答：
zk可用性本来就是很难保证的，面试训练营里分析了，他核心保证的是一致性而不是可用性，所以很多大公司都经常因为zk故障导致依赖他的各种系统都崩溃