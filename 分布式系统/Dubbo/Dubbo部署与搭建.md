### 安装ZK并启动
    执行zkServer.sh

### Demo搭建
    1.用本地环境搭建的，当然是单机的，用官方的dubbo-demo搭的，从git上clone下来，用的2.5.x版本的，毕竟教程大部分都是依照2.5.x的，不想花费多余的时间。
    2.首先是跑dubbo-demo，修改dubbo-demo-provider.xml和dubbo-demo-consumer.xml：
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>   
    把注册中心的地址配上，然后我们就可以debug跑provider的main函数了。
    3.此时我们打开zk的客户端，运行zkCli.cmd，可以使用ls /命令查看当前zk都有哪些节点，我们可以看见dubbo节点
    4.debug运行provider的main函数，发现控制台打印日志，打开zk客户端也可以查看到当前的应用的consumer信息

    这里注意zk的节点：
                                        dubbo
                        application1                  application2
                provider        consumer        provider        consumer
            p1 p2 p3 p4 p5    c1 c2 c3 c4 c5    p1 p2 p3        c1 c2 c3 c4  这里包含group和version信息