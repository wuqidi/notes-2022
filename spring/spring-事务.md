@Transtional

属性propagation：事务传播属性

* Propagation.REQUIRED：默认。如果存在事务则在当前事务中运行；若没有则新建事务。
* Propagation.SUPPORTS：如果在事务内，则支持当前事务执行；若不在事务内，非事务执行。
* Propagation.MANDATORY：必须在已有事务内执行，否则异常。
* Propagation.NOT_SUPPORT：不支持事务，存在事务则挂起；否则非事务方式执行。
* Propagation.NEVER：不支持事务，存在事务抛出异常；否则非事务方式执行。
* Propagation.NESTED：不存在事务，则新建事务；若存在事务，嵌套一个新事务执行。嵌套事务：外部回滚，内部也跟着回滚；内部回滚，外部不会跟着回滚。
* Propagation.REQUIRES_NEW：无论是否存在事务，都会新建一个事务。外部事务与内部事务均是独立。
  * 挂起：先暂停老的事务，执行新的事务，新的执行完，执行老的。

属性isolation：事务隔离级别

	* Isolation.DEFAULT：默认。数据库默认隔离级别。
	* Isolation.READ_UNCOMMITTED：读未提交（会脏读、不可重复度）
	* Isolation.READ_COMMITTED：读已提交（会不可重复读、幻读）
	* Isolation.REPEATABLE_READ：可重复度（会幻读）
	* Isolation.SERIALIZABLE：串行化

属性readOnly：默认false=读写，true-只读

​	只读在事务开始~结束，在这期间其他事务提交的数据，该事务将看不见。

属性timeout：**事务**超时时间，单位：秒。减少占用系统资源。

属性rollbackFor：导致事务回滚的异常类。默认RuntimeException。

属性rollbackForClassName：

属性noRollbackFor：不会导致事务回滚的异常类

属性noRollbackForClassName：

属性value：指定事务管理器

##### 失效原因：

1、方法不是public

2、所在类不在spring容器

3、声明在接口方法上

4、数据源没有配置事务管理器

5、传播属性设置Propagation.NOT_SUPPORT

6、错误catch了

7、确保同一线程内

8、无事务方法调用有事务方法：无事务方法和有事务方法均被aop代理此时事务无效，在同一个aop下。
		*事务方法调用无事务方法是可以生效的。*

