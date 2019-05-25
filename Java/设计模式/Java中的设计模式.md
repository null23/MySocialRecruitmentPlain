### 静态工厂
    ServiceLoader
    load(Class<S> service, ClassLoader loader)就是典型的静态工厂方法，直接调用ServiceLoader的私有构造器进行实例化，除了需要指定加载类的目标类型，还需要传入类加载器的实例。