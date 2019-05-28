### refresh()
    refresh()方法标志着IoC容器的正式启动
    过程大概分为以下三步：
        1.BeanDefintion的Resource定位
        2.BeanDefintion的载入
        3.BeanDefintion的注册

## IoC容器的初始化/Bean定义的载入
    Spring IoC的设计中，分为 1.Bean定义的载入 2.依赖注入 两个过程。注意，这是两个独立的过程，依赖注入一般发生在第一次通过getBean向容器索取Bean的时候(其实是看这个Bean是不是lazyinit的，是不是懒加载的)，如果这个Bean不是懒加载的，那么这个Bean实在IoC容器初始化的时候完成的。

    1.单例模式并且是非延迟加载的对象，会在IOC容器初始化的时候被创建且初始化。
    2.非单例模式或者是延迟加载的对象，是应用第一次向容器索要该Bean对象的时候被创建且初始化。

    下边说的都是IoC容器的初始化：

### BeanDefintion的Resource定位
    由ResourceLoader通过统一的Resource接口完成，这个Resource对各种形式的BeanDefintion的是用都提供了统一接口。
    对于不同的BeanDefintion的存在形式，有不同的Resource提供。比如针对文件系统的FileSystemResource，针对类路径定义Bean信息的ClassPathResource...

### BeanDefintion的载入(得到BeanDefinitionHolder对象)
    Spring 并不是直接把 XML 文件的内容转换成 BeanDefinitionHolder。解析时先解析 XML 得到 Document 对象，Document 对象就是 XML 文件在内存里的存储形式。从 Document 对象提取数据用的是 BeanDefinitionDocumentReader。
    简单来说，这步就是把用户定义好的Bean表示成IoC容器内部的数据结构，而这个IoC容器内部的数据结构就是BeanDefintion。
    BeanDefintion包含了Bean的一系列信息，比如Bean的名字，类型，成员变量都有啥，以及各个成员变量的类型和值...
    
### BeanDefintion的注册
    通过BeanDefinitionHolder对象注册BeanDefintion到AbstractBeanFactory的哈希表
    这步是向IoC容器注册那些BeanDefintion的过程，把BeanDefintion的Bean的信息注册到registryMap中，是调用BeanDefintionRegistry接口的实现来完成的。

### 真正的依赖注入
    getBean方法的调用
    1.通过beanName从beanDefintionMap中获取对应的bean，若没有，进行bean的实例化(依赖注入)操作
    2.从registryMap获取对应的BeanDefintion的信息，根据这些信息
        根据classType实例化属性为空的bean
        根据propertyValues把属性注入到bean中
        把bean put到beanDefintionMap中
    3.返回bean

今天看下spring aop，dubbo服务发现详解，java spi原理，dubbo spi
