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
```