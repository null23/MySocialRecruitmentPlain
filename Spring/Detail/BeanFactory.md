### BeanFactory和ApplicationContext
    BeanFactory的类图
        几个重要的BeanFactory(接口)
            BeanFactory：顶层接口
                    //这里是对FactoryBean的转义定义，因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象
                    String FACTORY_BEAN_PREFIX = "&";
                    //这里根据bean的名字，在IOC容器中得到bean实例，这个IOC容器就是一个大的抽象工厂。 
                    Object getBean(String name) throws BeansException;
                    //这里根据bean的名字和Class类型来得到bean实例，和上面的方法不同在于它会抛出异常：如果根据名字取得的bean实例的Class类型和需要的不同的话。
                    <T> T getBean(String name, Class<T> requiredType);
                    <T> T getBean(Class<T> requiredType) throws BeansException;
                    Object getBean(String name, Object... args) throws BeansException;
                    //这里提供对bean的检索，看看是否在IOC容器有这个名字的bean
                    boolean containsBean(String name);
                    //这里根据bean名字得到bean实例，并同时判断这个bean是不是单件 
                    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
                    //这里根据bean名字得到bean实例，并同时判断这个bean是不是原型 
                    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
                    //这里对得到bean实例的Class类型  
                    Class<?> getType(String name) throws NoSuchBeanDefinitionException;
                    //这里得到bean的别名，如果根据别名检索，那么其原名也会被检索出来  
                    String[] getAliases(String name);

            ListableBeanFactory(BeanFactory子接口)：获取beans的枚举
                    //是否含有给定的名称的bean
                    boolean containsBeanDefinition(String beanName);
                    int getBeanDefinitionCount();
                    //得到工厂所有的bean的名称数组
                    String[] getBeanDefinitionNames();
                    String[] getBeanNamesForType(Class<?> type);
                    //根据给定的类型得到和相应的策略得到所有的bean名称数组
                    String[] getBeanNamesForType(Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);
                    //根据给定过的类型得到所有该类型的bean，返回的结果是一个Map<bean名称，bean对象>的形式
                    <T> Map<String, T> getBeansOfType(Class<T> type) throws BeansException;
                    //得到给定名称的bean上的给定注解类型的注解对象
                    <A extends Annotation> A findAnnotationOnBean(String beanName, Class<A> annotationType);


            HierarchicalBeanFactory(BeanFactory子接口)：获取双亲ioc容器
                    //得到父工厂
                    BeanFactory getParentBeanFactory();
                    //在本地工厂中有没有给定名称的bean，不包括继承的工厂
                    boolean containsLocalBean(String name);

            AutowireCapableBeanFactory(BeanFactory子接口)：提供自动装配bean能力的功能支持
                    //用个给定的class类型制造一个完整的bean
                    <T> T createBean(Class<T> beanClass) throws BeansException;
                    //bean初始化完成之后执行回调函数和后处理器,
                    void autowireBean(Object existingBean) throws BeansException;
                    // 自动注入和设置bean的属性、执行factory回调函数比如setBeanName和setBeanFactory和执行bean的所有的后处理器
                    Object configureBean(Object existingBean, String beanName) throws BeansException;
                    //调用bean的init方法，这个方法是客户配置的，在bean实例化之后调用
                    Object initializeBean(Object existingBean, String beanName) throws BeansException;
                    //初始化完成之后应用后处理器
                    Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
                    //应用前处理器
                    Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName);

### BeanFactory和ApplicationContext
    如果说BeanFactory是Sping的心脏，那么ApplicationContext就是完整的身躯了。
    ApplicationContext实现了

### BeanDefinition
    以 BeanDefinition 类为核心发散出的几个类，都是用于解决 Bean 的具体定义问题，包括 Bean 的名字是什么、它的类型是什么，它的属性赋予了哪些值或者引用，也就是 如何在 IoC 容器中定义一个 Bean，使得 IoC 容器可以根据这个定义来生成实例 的问题。

### DefaultListableBeanFactory