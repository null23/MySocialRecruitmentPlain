### 安装ZK并启动
    执行zkServer.sh

### Demo搭建
    1.用本地环境搭建的，当然是单机的，用官方的dubbo-demo搭的，从git上clone下来，用的2.5.x版本的，毕竟教程大部分都是依照2.5.x的，不想花费多余的时间。
    2.首先是跑dubbo-demo，修改dubbo-demo-provider.xml和dubbo-demo-consumer.xml：
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>   
    把注册中心的地址配上，然后我们就可以debug跑provider的main函数了。
    3.此时我们打开zk的客户端，运行zkCli.cmd，可以使用ls /命令查看当前zk都有哪些节点，我们可以看见dubbo节点
    4.debug运行provider的main函数，发现控制台打印日志，打开zk客户端也可以查看到当前的应用的consumer信息

    这里注意zk的节点，是一个树形结构，大概长这样：
                                        dubbo
                        application1                  application2
                provider        consumer        provider        consumer
            p1 p2 p3 p4 p5    c1 c2 c3 c4 c5    p1 p2 p3        c1 c2 c3 c4  这里包含group和version信息

### 另一种配置方式
    官方demo是用Spring配置的，这里我们不使用Spring配置方式了，我们直接用dubbo提供的api来做。
    代码如下：

#### provider的main函数
```
    public static void main(String[] args) throws Exception{
        //服务实现
        DemoService demoService = new DemoServiceImpl();

        //当前应用配置
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("demo-provider");

        //连接注册中心配置
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setProtocol("zookeeper");
        registryConfig.setAddress("127.0.0.1:2181");

        //服务提供者协议配置
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(20880);

        //注意，ServiceConfig为重对象，内部封装了与注册中心的连接，以及开启服务端口
        //服务提供者暴露服务配置
        ServiceConfig<DemoService> serviceConfig = new ServiceConfig<DemoService>();
        serviceConfig.setApplication(applicationConfig);
        serviceConfig.setRegistry(registryConfig);
        serviceConfig.setProtocol(protocolConfig);
        serviceConfig.setInterface(DemoService.class);
        serviceConfig.setRef(demoService);
        serviceConfig.setVersion("1.0.0");
        serviceConfig.setGroup("group");

        serviceConfig.export();

        Thread.currentThread().join();
    }
```
#### consumer的main函数：
```
    public static void main(String[] args) throws Exception{
        //不用spring，直接用dubbo API的配置来做
        //当前应用配置
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("demo-consumer");

        //连接注册中心配置
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setProtocol("zookeeper");
        registryConfig.setAddress("127.0.0.1:2181");

        //服务提供者协议配置
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(20880);

        //注意，ServiceConfig为重对象，内部封装了与注册中心的连接，以及开启服务端口
        //服务提供者暴露服务配置
        ReferenceConfig<DemoService> referenceConfig = new ReferenceConfig<DemoService>();
        referenceConfig.setApplication(applicationConfig);
        referenceConfig.setRegistry(registryConfig);
        referenceConfig.setInterface(DemoService.class);
        referenceConfig.setVersion("1.0.0");
        referenceConfig.setGroup("group");

        DemoService demoService = referenceConfig.get();
        System.out.println("***********"+demoService.sayHello("nmsl"));

        try {
            Thread.currentThread().join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

    注意这里代码设置的group和version。
    当然我们可以让provider使用spring配置，consumer直接使用Dubbo的api，但是千万要注意group和version是否一致。

### dubbo-admin
    此时把dubbo-admin打包部署到tomcat上，可以监控到应用信息，以及相应的provider和consumer
### dubbo-monitor
    和上边一样，这是个监控，可以监控服务线上情况，比如调用次数，耗时等等
