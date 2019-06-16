### AbstractBeanFactory的getBean()方法
```
final String beanName = transformedBeanName(name);
Object bean;
//从缓存中获取单例Bean
Object sharedInstance = getSingleton(beanName);
if (sharedInstance != null && args == null) {
    if (logger.isDebugEnabled()) {
        if (isSingletonCurrentlyInCreation(beanName)) {
            logger.debug("Returning eagerly cached instance of singleton bean '" + beanName +
                    "' that is not fully initialized yet - a consequence of a circular reference");
        }
        else {
            logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
        }
    }
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
}
---------------------------------------------------------------------------------------------------------
/**
 * 存放的是单例 bean 的映射。
 */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
/**
 * 存放的是 ObjectFactory，可以理解为创建单例 bean 的 factory 。
 * 对应关系是 bean name --> ObjectFactory
 **/
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
/**
 * 存放的是早期的 bean，对应关系也是 bean name --> bean instance。
 * 它与 {@link #singletonFactories} 区别在于 earlySingletonObjects 中存放的 bean 不一定是完整。
 * 从 {@link #getSingleton(String)} 方法中，我们可以了解，bean 在创建过程中就已经加入到 earlySingletonObjects 中了。
 * 所以当在 bean 的创建过程中，就可以通过 getBean() 方法获取。
 * 这个 Map 也是【循环依赖】的关键所在。
 */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

//从缓存中获取单例Bean的实现方法
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    //从beanName-instance的k-v缓存中取bean实例(一级缓存)
    Object singletonObject = this.singletonObjects.get(beanName);
    //如果一级缓存中取不到，并且当前的bean正在被创建(这个很重要，在创建bean前后都会打标记)
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            //从创建单例bean的ObjectFactory中获取，这里取到的是完整的bean(二级缓存)
            singletonObject = this.earlySingletonObjects.get(beanName);
            //如果二级缓存取不到，并且当前bean允许提前实例化
            if (singletonObject == null && allowEarlyReference) {
                //从singletonFactories中获取相应的ObjectFactory对象。若不为空，则调用其ObjectFactory#getObject(String name) 方法，创建 Bean 对象，然后将其加入到 earlySingletonObjects ，然后从 singletonFactories 删除。
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    //TODO 把未初始化完成的bean放到二级缓存earlySingletonObjects中(没懂)
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    //TODO 删除三级缓存(没懂)
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
---------------------------------------------------------------------------------------------------------
//创建bean之前，添加"正在创建当前bean的标记"
protected void beforeSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}
---------------------------------------------------------------------------------------------------------
//创建完bean之后，移除"正在创建当前bean的标记"
protected void afterSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
        throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
    }
}
```
    在初始化单例Bean的时候，会先从缓存中获取单例Bean的信息，获取不到的时候，才会去创建这个单例Bean。

### isSingletonCurrentlyInCreation()
    判断当前的bean是否正在被加载
```
//判断当前bean是否正在创建，是由一个singletonsCurrentlyInCreation的Set集合来维护的
public boolean isSingletonCurrentlyInCreation(String beanName) {
    return this.singletonsCurrentlyInCreation.contains(beanName);
}

```