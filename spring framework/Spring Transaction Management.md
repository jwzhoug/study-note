# 事务需要遵循的原则：ACID

- `Automicity`  原子性 ：事务要么被全部执行，要么全不被执行。如果事务下的子事务全部提交成功，则所有数据库操作被提交，否则应该进行事务回滚。

  **计算机中很多地方（比如编程中）都需要一些保证原子性的实现，一个需要执行完所有的操作才有意义的操作集合，就可以实现成原子性的**

- `Consistency `一致性：事务应该确保数据库状态从一个一致状态转变为另一个一致性状态

  计算机中很多地方（比如编程（多线程）中，cpu工作机制中（多指令并行）需要保证最后的结果是这些所有操作时候的最终结果…..）
  简单的例子：现在有6个人 每人有一个账号，他们之间会相互转账，这样就组成了一个小的数据系统，那么什么事一致性呢？这是个人定义的，比如这几个人账号的总金额不变，那就是一致的，反之不一致。 再比如 现在数据时分布式存储的，有一个数据在几个地方都保存了，那么任何时候这几个地方的数据都必须相同，这也是一致性。

- `Isolation` 隔离性：事务相互间不能被影响(多线程中一些情况也是需要实现隔离性的)

- `Durabillity` 持久性：事务一旦提交之后将他对数据库的操作应该是永久性的，即便出现其他故障，事务处理结果也应不被影响。

# 1. 在spring 中事务管理的顶层接口

* `PlatformTransactionManager` : 事务管理 的最顶层的接口

```java
public interface PlatformTransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

* `TransactionDefinition`: 事务的一些定义信息的顶层接口（事务传播行为，事务隔离级别，事务超时，事务只读），不理解隔离级别的读者可以看后文中的基本概念补充这一个章节

```java
public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    int TIMEOUT_DEFAULT = -1;

    int getPropagationBehavior();

    int getIsolationLevel();

    int getTimeout();

    boolean isReadOnly();

    @Nullable
    String getName();
}
```

* `TransactionStatus`：为事务代码提供了一种简单的方法来控制事务执行以及查询事务的状态。它们对于所有事务`API`都是通用的：

```java
public interface TransactionStatus extends SavepointManager, Flushable {
    boolean isNewTransaction();// 是否是一个新的事务

    boolean hasSavepoint();// 是否有保存点

    void setRollbackOnly();// 

    boolean isRollbackOnly();

    void flush();

    boolean isCompleted();// 事务是否完成
}
```



# 2. 接口实现类的选择

在了解了上面的内容之后，我们知道那只是对应的接口，那么我们在使用中应该怎么去使用对应的实现类呢（当然你也可以基于spring的接口，自定义实现类）

`PlatformTransactionManager`  的实现类的选择我们通常需要了解它的工厂环境，比如：

* 用`JDBC/myBatis `持久化数据时:  我们要选择 `org.springframework.jdbc.datasource.DataSourceTransactionManager`

* 使用`Hibernate`持久化数据时: 我们要选择 `org.springframework.orm.hibernate3.HibernateTransactionManager`

* 使用`JTA`来实现管理事务：我们要选择 `org.springframework.transaction.jta.JtaTransactionManager`

  **当事务跨越多个资源是需要使用这个 `JTA`**

* 使用` JPA` 进行持久化：我们要选择 `org.springframework.orm.jpa.JpaTransactionManager`

* 持久化机制是`Jdo` 的时候：我们要选择 `org.springframework.jdo.JdoTransactionManager`

别的情况下，悬着对应的实现就可以了。

# 基本概念补充

## 隔离级别

- `Serializable` (序列化，串行化) : 串行执行 可避免脏读，不可重复读，幻读的发生。它是最高的事务隔离级别，同事花费的代价也是很大的，性能很低，一般很少使用。

- `Repeatable read` (重复读) :一个事务开始读取这条数据，那么别的事务就不能对其进行修改。 可以解决不可重复读和脏读，不能避免幻读，**但是`mysql`中的 `mvcc `可以避免幻读,这样就不需要升级隔离级别了**

- `Read committed` (读已提交): 一个事物要等另一个事务提交之后才能读取 可避免脏读 

- `Read uncommitted`(读未提交) 

  **mysql 默认是 `REPEATABLE-READ` 重复读**

## 事务传播行为

![1540983454912](https://github.com/Alan-Jun/study-note/blob/master/spring%20framework/assets/1540983454912.png)

# 声明式事务管理

## 1. 配置代理的方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--<context:component-scan base-package="com.stu" />-->

    <context:property-placeholder location="classpath:jdbc.properties"/>

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--service-->
    <bean id="accountService" class="com.stu.service.imp.AccountServiceImp">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!--dao层-->
    <bean id="accountDao" class="com.stu.dao.imp.AccounDaoImp">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 事务管理类 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置service 层代理 -->
    <bean id="accountServiceProxy" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
        <!--配置目标对象-->
        <property name="target" ref="accountService"/>
        <!--注入事务管理器-->
        <property name="transactionManager" ref="transactionManager"/>
        <!-- 注入事务的属性 -->
        <property name="transactionAttributes" >
            <!-- 下面的配置 查看  TransactionProxyFactoryBean 的源码 就可以看到配置-->
            <props>
                <!--格式  key是方法，可以使用通配符 *-->
                <!-- prop 内是value的值，使用 ， 号分割，按顺序是
                    1. PROPAGATION 事务的传播行为
                    2. ISOLATION 事务的隔离级别
                    3. ReadOnly 只读（不能进行任何修改操作）
                    4. +Exception  发生哪一类的异常，我们照常提交
                    5. -Exception  发生哪一类的异常，回滚事务
                    -->
                <prop key="transfer">PROPAGATION_REQUIRED</prop>
                <!--
                    <prop key="transfer">PROPAGATION_REQUIRED,
											+java.lang.ArithmeticException</prop>
                    这样写了 可以发现即使发生上述的异常 ，但是异常发生前的 数据库操作，被提交了
                    -->
            </props>
        </property>
    </bean>

</beans>
```

