1.前言：
	1.1 为什么需要Spring？Spring是如何简化Java开发的？
		1.基于POJO/Bean的轻量级和最小侵入性编程(侵入性：什么是侵入性？如何对比评估侵入性？)
		2.通过依赖注入和面向接口实现松耦合(DI和面向接口)
		3.基于切面和惯例进行声明式编程(声明式事务和编程式事物)
		4.通过切面和模板减少样板式代码(减少冗余代码)
	1.2 依赖注入-DI
		1.DI的核心就是解耦合，将对象之间的依赖关系，由第三方组件(Spring)在创建对象的时候来协调设定。对象不用进行实例化创建，以及依赖管理。
		2.依赖注入的实现方式：
			1.构造函数注入
			2.setter注入
			3.接口注入
	1.3 AOP
	1.4 容器(Bean的创建)
		1.Bean工厂
		2.应用上下文 ApplicationContext接口(简单点，可以理解为，应用上下文就是容器的对象，是对于Spring容器抽象的实现)
			如何创建应用上下文？配置XML就可以了
	1.5 Bean的生命周期
		1.非Spring：直接new进行实例化，由Java自动回收
		2.Spring：
			1.对bean进行实例化
			2.将值和bean的引用，注入到bean对应的属性中
			3.若bean实现了BeanNameAware接口，Spring将ID传递给setBeanName()方法
			4.若bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
			5.若bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext()方法，将bean所在的应用上下文的引用(容器的实例，XML的创建)传进来
			6.若bean实现了BeanPostProcessor接口，Spring将调用他们的postProcessBeforeInitialization()方法
			7.若bean实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法，类似的，如果bean使用init method声明了初始化方法，该方法也会被调用
			8.若bean实现了BeanPostProcessor接口，Spring将调用他们的postProcessAfterInitialization()方法
			9.此时，bean已经准备就绪，可以被应用程序使用了，他们将一直驻留在应用上下文中，直到该应用上下文被销毁
			10.如果bean实现了DisposableBean接口，Spring将调用他的destory()接口方法。同样，如果bean使用destory method声明了销毁方法，该方法也会被调用。
	1.6	容器的生命周期
------------------------------------------------------------------------------------------------------------------------------------------------------
2.装配Bean
	2.1 何为装配？
		创建对象之间协作关系的行为被称为装配，这也是依赖注入(DI)的本质
	2.2	Spring配置的几个方案(为了装配Bean)
		1.XML中显式配置
		2.Java中显式配置
		3.自动装配(注解，隐式的bean发现机制)
	2.3	自动化装配Bean
		1.Spring从两个角度来实现自动化装配：
			1.组件扫描
			2.自动装配
		2.为Bean设置组件扫描的基础包：
			1.XML配置<context:componet-scan/>属性
			2.基于Java配置，自动扫描配置在JavaConfig上的
		3.用注解实现自动装配
			1.@Autowired
	2.4	通过XML装配Bean
		1.声明一个简单的<bean>
			<bean id="slaughter" class="wqj.slaughter"/>
		2.构造函数注入bean引用
			<bean id="slaughter" class="wqj.slaughter">
					<constructor-arg ref="pure"/>
			</bean>
------------------------------------------------------------------------------------------------------------------------------------------------------
3.高级装配