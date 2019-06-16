### 到底什么是FactoryBean
    我不建议把BeanFactory和FactoryBean放在一起说，因为太容易引起人的误导，网上的大部分文章，都写得太烂，更加误导了读者对于FactoryBean的理解。在理解FactoryBean的时候，我们只需要把BeanFactory理解为一个IoC容器就好了，当然事实也确实如此。

    FactoryBean本质就是一个Bean，FactoryBean也是被IoC容器管理。这个Bean实现了FactoryBean接口，提供了最主要的getObject()的方法，FactoryBean的意义就在于此，可以通过getObject()方法获得一个Object。
    当我们通过getBean("factoryBeanName")获取这个FactoryBean的时候，IoC容器会自动调用FactoryBean的getObject()方法。

    下面先看下FactoryBean接口提供的三个方法：
```
    //获取bean实例，会有两个地方可能会调用这个方法
    //一个是显式的初始化一个FactoryBean调用，另一个是getBean(factoryBeanName)
    public Object getObject() throws Exception {
        return new A();
    }

    public Class<?> getObjectType() {
        return null;
    }

    //这个返回的Object A，在IoC容器是否是单例的
    //注意这里说的是IoC容器，并不是显式初始化FactoryBean调用getObject方法返回的，显式调用永远得到的都是新的对象
    public boolean isSingleton() {
        return true;
    }
```

    上边代码的注释提到了两个可以调用getObject方法的地方，一个是FactoryBean显式调用，一个是IoC容器在getBean的时候自动调用。

### 那么这个Object和IoC容器的关系是什么呢？这个Object是由Spring容器管理的么？
    这个Object姑且我们可以认为他是一个Bean，这个Bean的初始化是由FactoryBean做的，比如我们在FactoryBean的getObject方法中显式的new Object()并返回。
    那么这个getObject方法是何时调用的？是由我们手动调用？还是IoC容器什么时候会自动调用？
    没错。我们可以自己显式的调用FactoryBean.getObject()，但是Spring容器会在我们调用getBean方法的时候，也会自动的调用FactoryBean.getObject()方法。
    那么这两种有啥区别呢？

### 触发getObject的时机
    1.实例化FactoryBean，显式调用getObject方法
    FactoryBean<?> factoryBean = (FactoryBean<?>) context.getBean(FACTORY_BEAN_PREFIX + "slaughterImpl");
    factoryBean.getObject();

    2.调用ApplicationContext的getBean()方法
    A a = (A) context.getBean("slaughterImpl");
    此时IoC容器的getBean方法的doGetBean方法会判断slaughterImpl是不是一个FactoryBean，很明显如果slaughterImpl实现了FactoryBean接口的话，那自然是，此时会调用slaughterImpl.getObject()方法(会判断是不是单例)返回实例化的Bean。

### FactoryBean和其他Bean的区别？本身是如何加载的？
    第一次调用getObject方法的时机：
        FactoryBean作为一个Bean，也会在IoC容器初始化的时候被加载并且实例化，都是在getBean(factoryBeanName)调用preInstantiateSingletons()方法之后实例化，并且调用getObject方法。但是获取到的不是FactoryBean，而是一个Object。

    后续调用getObject方法的时机：
        在后续的调用getBean方法的时候，由于FactoryBean之前在IoC容器加载的时候已经被初始化了，所以会直接进入getObjectForBeanInstance()方法。
        如果此时的beanInstance是普通的bean，就会直接返回，如果是个FactoryBean，会调用FactoryBean.getObject()。

### FactoryBean的单例属性
    会判断这个FactoryBean是不是单例的。
    如果是，就先从FactoryBean的缓存，也就是FactoryBeanRegistrySupport类的factoryBeanObjectCache获取这个Object实例，如果缓存中没有，就调用FactoryBean的getObject方法创建一个新的Object实例。
    如果不是，每次都调用FactoryBean的getObject方法创建实例。


### 如何获取FactoryBean？



