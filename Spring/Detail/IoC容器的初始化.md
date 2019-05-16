### refresh()
    refresh()方法标志着IoC容器的正式启动
    过程大概分为以下三步：
        1.BeanDefintion的Resource定位
        2.BeanDefintion的载入
        3.BeanDefintion的注册

## IoC容器的初始化/Bean定义的载入
    Spring IoC的设计中，分为 1.Bean定义的载入 2.依赖注入 两个过程。注意，这是两个独立的过程，依赖注入一般发生在第一次通过getBean向容器索取Bean的时候(其实是看这个Bean是不是lazyinit的，是不是懒加载的)，如果这个Bean不是懒加载的，那么这个Bean实在IoC容器初始化的时候完成的。

    下边说的都是IoC容器的初始化：

### BeanDefintion的Resource定位
    由ResourceLoader通过统一的Resource接口完成，这个Resource对各种形式的BeanDefintion的是用都提供了统一接口。
    对于不同的BeanDefintion的存在形式，有不同的Resource提供。比如针对文件系统的FileSystemResource，针对类路径定义Bean信息的ClassPathResource...

### BeanDefintion的载入
    简单来说，这步就是把用户定义好的Bean表示成IoC容器内部的数据结构，而这个IoC容器内部的数据结构就是BeanDefintion。
    BeanDefintion包含了Bean的一系列信息，比如Bean的名字，类型，成员变量都有啥，以及各个成员变量的类型和值...
    
### BeanDefintion的注册
    这步是向IoC容器注册那些BeanDefintion的过程。是调用BeanDefintionRegistry接口的实现来完成的。