### 负载均衡的几种负载均衡策略
#### random loadbalance
默认情况下，dubbo是random load balance随机调用实现负载均衡，可以对provider不同实例设置不同的权重，会按照权重来负载均衡，权重越大分配流量越高，一般就用这个默认的就可以了。

#### roundrobin loadbalance
还有roundrobin loadbalance，这个的话默认就是均匀地将流量打到各个机器上去，但是如果各个机器的性能不一样，容易导致性能差的机器负载过高。所以此时需要调整权重，让性能差的机器承载权重小一些，流量少一些。

跟运维同学申请机器，有的时候，我们运气，正好公司资源比较充足，刚刚有一批热气腾腾，刚刚做好的一批虚拟机新鲜出炉，配置都比较高。8核+16g，机器，2台。过了一段时间，我感觉2台机器有点不太够，我去找运维同学，哥儿们，你能不能再给我1台机器，4核+8G的机器。我还是得要。

#### leastactive loadbalance
这个就是自动感知一下，如果某个机器性能越差，那么接收的请求越少，越不活跃，此时就会给不活跃的性能差的机器更少的请求

#### consistanthash loadbalance
一致性Hash算法，相同参数的请求一定分发到一个provider上去，provider挂掉的时候，会基于虚拟节点均匀分配剩余的流量，抖动不会太大。如果你需要的不是随机负载均衡，是要一类请求都到一个节点，那就走这个一致性hash策略。


### 集群容错的几种机制
#### failover cluster模式
失败自动切换，自动重试其他机器，默认就是这个，常见于读操作

#### failfast cluster模式
一次调用失败就立即失败，常见于写操作

#### failsafe cluster模式
出现异常时忽略掉，常用于不重要的接口调用，比如记录日志

#### failbackc cluster模式
失败了后台自动记录请求，然后定时重发，比较适合于写消息队列这种

#### forking cluster
并行调用多个provider，只要一个成功就立即返回

#### broadcacst cluster
逐个调用所有的provider

### 我司生产环境常用的
    负载均衡：
        radom balance(默认)
        leastactive balance(性能差就少分点)

    集群容错策略：
        failover(默认) 失败自动切换 
        failfast    快速失败，失败报错

**provider和consumer都是可以进行配置的，配置的优先级:provider > consumer> 默认**