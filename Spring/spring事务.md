####MySQL中的事务：
#####1.事务的四个特性(ACID)：
	1.原子性(A)
		一个事务要么成功，要么失败回滚
	2.一致性(C)
		事务未成功，事务中修改的状态不会成功
	3.隔离性(I)
		这个是针对多个事务的并发来说的，一个事务在提交以前，对其他事物是否可见的
	4.持久性(D)
		一旦事务提交，修改将会永久保存到数据库中
#####2.隔离级别
	1.由事务并发所产生的问题：
		1.脏读
			事务A读取到事务B修改但未提交的数据
		2.不可重复读
			事务A多次读取数据，事务B也对该数据进行操作，事务B对该数据的修改(update)，可能会造成事务A多次读取的结果不同。
		3.幻读
			事务A读取某范围内的数据(例如limit 1, 10)，事务B对于该范围内的数据进行修改(insert)，会造成事务A多次读取的结果不同(幻行)
	2.解决上述问题：
		为了解决上述问题，MySQL采用了四种隔离机制：
		1.Read Uncommitted 未提交读
			事务B的修改，即使并未提交，也会对事务A可见。仍然存在脏读，不可重复读，幻读的问题。
		2.Read Committed 提交读
			只能看见提交的事务，只有事务B真正提交了，才会对事务A可见。解决了脏读，仍然存在不可重复读，幻读的问题。
		3.Repeatable Read 可重复读
			MySQL默认的隔离级别，保证了同一个事务中，多次读取同一事物的结果是一致的。解决了脏读，不可重复读，但是仍然存在幻读的问题。
		4.Serializable 可串行化
			最高的隔离级别，强制多个事务串行执行，以此解决幻读的问题。简单来说，会在读取的每一行数据上都加锁，但是会出现超时和锁争用的问题。

Spring事务管理的为不同的事务API提供一致的编程模型，具体的事务管理机制由对应各个平台去实现。

####支持两种事务管理方式： 
   #####1.编程式事务管理：
	使用TransactionTemplate或PlatformTransactionManager实现
	编程式事务管理优势：可以控制事务的粒度，最细粒度到代码块级别；

   #####2.声明式事务管理：
	例如我们经常使用的@Transactional注解，这是建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务（此处取决于事务的传播行为），在执行完目标方法之后根据执行情况提交或者回滚事务（执行成功则提交，失败则进行实物的回滚）
	声明式实物管理优势：在方法外进行声明，事务控制的代码不会与业务逻辑代码混在一起，最细粒度到方法级别（解决方法：可以将需要进行事务管理的代码块独立为方法，通过方法间调用实现）；符合spring倡导的非侵入式的开发方式，即业务处理逻辑代码与事务管理代码不放在一起


了解编程式事务和声明式事务之前，我们要对一下几个概念了解：
	1.隔离级别：
		Isolation.DEFAULT：采用MySQL默认的隔离级别
		Isolation.READ_UNCOMMITTED
		Isolation.READ_COMMITTED
		Isolation.REPEATABLE_READ
		Isolation.SERIALIZABLE

	2.传播行为：
		当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。
		支持当前事务的情况：
		TransactionDefinition.PROPAGATION_REQUIRED： 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
		TransactionDefinition.PROPAGATION_SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
		TransactionDefinition.PROPAGATION_MANDATORY： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
		
		不支持当前事务的情况：
		TransactionDefinition.PROPAGATION_REQUIRES_NEW： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
		TransactionDefinition.PROPAGATION_NOT_SUPPORTED： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
		TransactionDefinition.PROPAGATION_NEVER： 以非事务方式运行，如果当前存在事务，则抛出异常。

		其他情况：
		TransactionDefinition.PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

	3.超时：
		执行多长时间之后仍然没执行完，回滚

	4.只读：
		事务是否只读或者读/写资源

	5.回滚规则：
		规定了哪些异常会导致事务回滚

#####编程式事务：
	上边已经提到，Spring的编程式事务可以通过TransactionTemplate和更底层的PlatformTransactionManager来实现。

	Spring编程式事务的核心接口是PlatformTransactionManager接口，这个接口为各个平台提供了事务管理的实现，其中分别有三个方法：
	1.getTranscation：
		通过TransactionDefinition获取TranscationStatus(一个接口)，TranscationStatus表现了当前事务的状态，例如是否完成，是否回滚。
	2.commit
		提交当前事物
	3.rollback
		回滚当前事物

#####实现声明式事务的四种方式：
	1.基于 TransactionInterceptor 的声明式事务: Spring 声明式事务的基础，通常也不建议使用这种方式，但是与前面一样，了解这种方式对理解 Spring 声明式事务有很大作用。
	2.基于 TransactionProxyFactoryBean 的声明式事务: 第一种方式的改进版本，简化的配置文件的书写，这是Spring 早期推荐的声明式事务管理方式，
	但是在 Spring 2.0 中已经不推荐了。
	3.基于<tx>和<aop>命名空间的声明式事务管理：目前推荐的方式，其最大特点是与Spring AOP结合紧密，可以充分利用切点表达式的
	强大支持，使得管理事务更加灵活。
	4.基于@Transactional的全注解方式： 将声明式事务管理简化到了极致。开发人员只需在配置文件中加上一行启用相关后处理 Bean 的配置然后在需要实施
	事务管理的方法或者类上使用 @Transactional 指定事务规则即可实现事务管理，而且功能也不必其他方式逊色。
	通过设置配置中的TransactionManager的proxy-target-class=true等属性，可以切换动态代理的实现方式，是基于JDK还是Cglib