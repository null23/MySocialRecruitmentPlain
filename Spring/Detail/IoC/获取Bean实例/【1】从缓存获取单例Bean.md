### AbstractBeanFactory的getBean()方法
### 从缓存获取单例Bean
```
final String beanName = transformedBeanName(name);
Object bean;
//从缓存中获取单例Bean-sharedInstance
Object sharedInstance = getSingleton(beanName);
if (sharedInstance != null && args == null) {
    if (logger.isDebugEnabled()) {
        if (isSingletonCurrentlyInCreation(beanName)) {
            logger.debug("Returning eagerly cached instance of singleton bean '" + beanName +
                    "' that is not fully initialized yet - a consequence of a circular reference");
        } else {
            logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
        }
    }
    //在返回bean之前，对bean进行处理，因为这个bean很可能不是我们想要的那个bean
***//比如sharedInstrance其实仅仅是个FactoryBean，但是，我们想要的是getObject返回的那个实例***
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
}

---------------------------------------------------------------------------------------------------
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

---------------------------------------------------------------------------------------------------
//从缓存中获取单例Bean的实现方法，这个方法仅仅是从缓存中获取Bean
//还有另一个getSingleton(String beanName, ObjectFactory<?> singletonFactory)方法，这个方法主要是创建Bean的方法
//区别在于：一个仅仅是从缓存里拿，另一个是会创建
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
```
    在初始化单例Bean的时候，如果这个Bean之前已经被实例化过了，就会从缓存中获取Bean的实例。
    从缓存中获取单例Bean的步骤：
    1.从singletonObjects(beanName->instance)中获取
    2.从earlySingletonObjects(beanName->ObjectFactory)中获取
    3.从singletonFactories(beanName->ObjectFactory)中获取
    第2步和第3步的区别，我现在也不清楚啊，哎
    
### getObjectForBeanInstance()
    在返回bean之前，对bean进行处理，因为这个bean很可能不是我们想要的那个bean。其实就是检测当前 bean 是否是 FactoryBean 类型的 bean 。
***比如sharedInstrance其实仅仅是个FactoryBean，但是，我们想要的是getObject返回的那个实例，所以就要调用这个方法来返回我们真正想要的那个bean***


### isSingletonCurrentlyInCreation() 判断当前bean是否正在创建
```
//判断当前bean是否正在创建，是由一个singletonsCurrentlyInCreation的Set集合来维护的
public boolean isSingletonCurrentlyInCreation(String beanName) {
    return this.singletonsCurrentlyInCreation.contains(beanName);
}

---------------------------------------------------------------------------------------------------
//创建bean之前，添加"正在创建当前bean的标记"
protected void beforeSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}

---------------------------------------------------------------------------------------------------
//创建完bean之后，移除"正在创建当前bean的标记"
protected void afterSingletonCreation(String beanName) {
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {
        throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");
    }
}
```
    isSingletonCurrentlyInCreation()
        判断当前Bean是否被创建，通过bean创建的标记来判断
    beforeSingletonCreation()
        在Bean创建之前添加标记，其实就是在一个Set集合通过记录beanName来维护Bean是否正在创建
    afterSingletonCreation()
        在Bean创建之前移除标记

### 后续请见【1.1】FactoryBean相关的
    
