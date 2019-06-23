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

### BeanDefintion的Resource定位(配置文件资源->Resource)
    由ResourceLoader通过统一的Resource接口完成，这个Resource对各种形式的BeanDefintion的是用都提供了统一接口。
    对于不同的BeanDefintion的存在形式，有不同的Resource提供。比如针对文件系统的FileSystemResource，针对类路径定义Bean信息的ClassPathResource...
    ---
    这个 Resource 定位指的是 BeanDefinition 的资源定位，它由 ResourceLoader 通过统一的 Resource 接口来完成，这个 Resource 对各种形式的 BeanDefinition 的使用都提供了统一的接口。Spring 提供了适用于各种场景的默认实现，如类路径下的资源可以用 ClassPathResource、网络上的资源可以用 UrlResource。
    解读：
    Resource 是 Spring 对资源的一个抽象，通过对资源进行统一抽象，Spring 可以支持各种形式的资源文件以及自定义的资源，同时又可以确保底层对 Resource 的处理流程是统一的。Resource 是一个接口，Spring 采用的是面向接口的开发方式。

### BeanDefintion的载入(Resource->BeanDefinitionHolder对象)
    Spring 并不是直接把 XML 文件的内容转换成 BeanDefinitionHolder。解析时先解析 XML 得到 Document 对象，Document 对象就是 XML 文件在内存里的存储形式。从 Document 对象提取数据用的是 BeanDefinitionDocumentReader。
    简单来说，这步就是把用户定义好的Bean表示成IoC容器内部的数据结构，而这个IoC容器内部的数据结构就是BeanDefintion。
    BeanDefintion包含了Bean的一系列信息，比如Bean的名字，类型，成员变量都有啥，以及各个成员变量的类型和值...
    ---
    BeanDefinition 的载入过程，就是解析 Resource 对象得到 BeanDefinitionHolder 对象的过程。BeanDefinitionHolder 的作用是根据名称或者别名持有 beanDefinition，承载了 name 和 BeanDefinition 的映射信息。所以 BeanDefinitionHolder 拥有如下三个字段：
    beanDefinition：实际持有的 beanDefinition 对象；
    beanName：Bean 的名字；
    aliases：Bean 的别名；
    Spring 并不是直接把 XML 文件的内容转换成 BeanDefinitionHolder。解析时先解析 XML 得到 Document 对象，Document 对象就是 XML 文件在内存里的存储形式。从 Document 对象提取数据用的是 BeanDefinitionDocumentReader。

    对于 Document 对象中的一个节点 Element，是使用 BeanDefinitionParser 进行解析。开发者也可以自定义 BeanDefinitionParser 从而实现对 xml 配置的自定义解析，可以实现诸如自定义 XML 标签的功能。

    解读：
    XML 的载入大致可以分为 Document、Element 两个载入阶段。作为通用的技术框架，Spring 还是为我们提供了很多扩展点，可以按需改变 Spring 的执行流程。对于 XML 的载入，最常见的就是自定义标签，比如 dubbo。
    
### BeanDefintion的注册(BeanDefinitionHolder->BeanDefinition->BeanDefinitionMap)
    通过BeanDefinitionHolder对象注册BeanDefintion到AbstractBeanFactory的哈希表
    这步是向IoC容器注册那些BeanDefintion的过程，把BeanDefintion的Bean的信息注册到registryMap中，是调用BeanDefintionRegistry接口的实现来完成的。
    ---
    这个操作是通过调用 BeanDefinitionRegistry 接口来实现的。这个注册过程把载入过程中解析得到的 BeanDeifinition 向 Ioc 容器进行注册。
    调用 registerBeanDefinition 方法解析 BeanDefinitionHolder 对象，按照 Bean 的名称、别名将 BeanDefinition 注册到 IoC 容器中，存储在 beanDefinitionMap 中。至此，容器的初始化基本完成。

    解读：
    registerBeanDefinition 是一个可以复用的方法，毕竟 Spring 内部注册的流程是确定的，因此 registerBeanDefinition 处于底层实现中。各种形式不同的上层实现最终都调用了同一个注册方法，殊途同归。在阿里很多软件架构中，也都采用了类似的设计，把注册配置统一收口便于管理。

### 真正的依赖注入
    getBean方法的调用
    1.通过beanName从beanDefintionMap中获取对应的bean，若没有，进行bean的实例化(依赖注入)操作
    2.从registryMap获取对应的BeanDefintion的信息，根据这些信息
        根据classType实例化属性为空的bean
        根据propertyValues把属性注入到bean中
        把bean put到beanDefintionMap中
    3.返回bean

今天看下spring aop，dubbo服务发现详解，java spi原理，dubbo spi

### BeanFactory和FactoryBean
    FactoryBean本质就是一个Bean，通过这个Bean可以获取这个FactoryBean能创建的实例对象。
    想要获取一个FactoryBean，就必须让一个Bean实现FactoryBean接口，然后这个Bean的实例就是FactoryBean了，我们可以在其getObject方法中实例化我们想获得的其他bean。
    这个FactoryBean的作用仅仅就是获取其他的Bean，比如有个连接池类，我们选择在FactoryBean的getObject方法中初始化这个类。

    FactoryBean只有三个方法：
    public Object getObject() throws Exception {
        return null;
    }
    public Class<?> getObjectType() {
        return null;
    }
    public boolean isSingleton() {
        return false;
    }
