# 1.介绍

*面向方面编程*（AOP）通过提供另一种思考程序结构的方式来补充面向对象编程（OOP）。OOP中模块化的关键单元是类，而在AOP中，模块化单元是*方面*。方面可以实现关注的模块化，例如跨越多种类型和对象的事务管理。

**实现AOP的方式有两种：**

* **预编译**：主要代表有 AspectH
* **动态代理**: springAOP , JbossAOP

**spring 中主实现方式是通过动态代理实现的，动态代理又主要包含两种方式：**

* **jdk的动态代理**
* **第三方包 cglib**

# 2.使用的场景

* logging 日志记录
* Performance optimization 性能优化（性能统计）
* Authentication 权限（安全控制）
* Transactions 事务处理
* Error handling 错误处理（异常处理）
* Caching 缓存
* Context passing 内容传递
* Lazy loading 懒加载
* Debugging 调试
* Persistence 持久化
* source pooling 资源池
* synchronization 同步

# 3.AOP中的基本概念

* 切面（`Aspect`）: 一个关注点的模块化（实现），这个关注点可能会横切多个对象。**Spring 中允许自定义 `Aspect`**

* 连接点（`JoinPoint`）：程序执行过程中的某一个阶段点（比如方法的调用、异常的抛出等），在Spring AOP中，连接点*始终* 表示方法执行。 

* 通知（`Advice`）: 在`Aspect`的某一个特定的连接点上**执行的动作（处理逻辑）**，也就是向连接点注入的代码，类型包括：

  * `around advice`: 围绕`JoinPoint`的`Advice`。这是最有力的`Advice`。`around advice`可以在方法调用之前和之后执行自定义行为。**建议使用下面的特定类型的`Advice`,在必要情况下 在使用 `around advice` 因为做同样的事使用最具体的 `Advice`类型可以提供最简单的编程模型，减少错误出现的可能性**
  * `before advice`: 在`JoinPoint`之前执行但不能阻止执行流程进入`JoinPoint`的`Advice`（除非它(`Advice`)抛出异常）。 
  * `after returning advice`: 在`JionPoint`正常完成后执行的*建议`Advice`
  * `after throwing advice`: 如果方法是抛出异常退出，则执行 `Advice`
  * `after (finally) advice`: 无论`JoinPoint`退出的方式（正常或异常返回），都要执行`Advice` 

  许多AOP框架（包括Spring）将建议建模为*拦截器*，在连接点**周围**维护一系列拦截器。 

* 切入点（`PointCut`）: 匹配连接点的断言，`Advice`与`PointCut`表达式相关联，并在`PointCut`匹配的任何`JoinPoint`处运行，**也就是说 `Pointcut`是`JoinPoint`的集合，它是程序中需要注入`Advice` 的位置的集合，指明`Advice`要在什么样的条件下才能被触发。`org.springframework.aop.Pointcut `接口用来指定到特定的类和方法。**

* 引入/引用（`Introduction`）: `Spring AOP`允许您向任何`Advice Object`引入新接口（和相应的实现）。例如，您可以使用`Introduction`  使bean实现 `IsModified`接口

* 目标对象（`Taget Object`）: 被一个或则多个`Aspect`所`Advice`的对象,也叫做 `Advice Object`，由于Spring AOP是使用运行时代理实现的，因此该对象始终是*代理*对象（`AOP Proxy Object`）。 

* AOP代理（`AOP Proxy`）: 由AOP框架创建的对象，用于实现`Aspect Contract`（`Advice`方法执行等功能）。在Spring Framework中，AOP代理将是JDK动态代理或CGLIB代理。 

* 织入（`Weaving`）: 把`Aspect`连接到其他应用程序的对象或类上，并创建一个被通知（`adviced`）对象，分为：**编译时织入，类加载时织入，执行时织入**

# 4.Spring AOP 的功能和目标

1. Spring AOP是用纯Java实现的。不需要特殊的编译过程。Spring AOP不需要控制类加载器层次结构，因此适合在Servlet容器或应用程序服务器中使用。
2. Spring AOP目前仅支持方法执行连接点（建议在Spring bean上执行方法）。虽然可以在不破坏核心Spring AOP API的情况下添加对字段拦截的支持，但未实现字段拦截**。如果您需要建议字段访问和更新连接点，请考虑使用AspectJ等语言。**
3. Spring AOP的AOP方法与大多数其他AOP框架的方法不同。目的**不是提供最完整的AOP实现**（尽管Spring AOP非常强大）; 它是在AOP实现和Spring IoC之间提供紧密集成，以帮助解决企业应用(不光能在企业应用中使用)程序中的常见问题。
4. Spring AOP永远不会与AspectJ竞争提供全面的AOP解决方案。spring开发团队相信像Spring AOP这样的基于代理的框架和像AspectJ这样的完整框架都很有价值，而且它们是互补的，而不是竞争。**Spring将Spring AOP和IoC与AspectJ无缝集成，以便在一致的基于Spring的应用程序架构中满足AOP的所有使用需求。此集成不会影响Spring AOP API或AOP Alliance API：Spring AOP保持向后兼容。**有关Spring AOP API的讨论，请参阅https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-api。

# 5.Spring AOP的底层实现原理

* Spring AOP 默认使用AOP代理的标准JDK ***动态*代理**。这使得任何接口（或接口集）都可以被代理。 
* Spring AOP也可以使用CGLIB代理。这是代理类而不是接口所必需的。如果业务对象未实现接口，则默认使用CGLIB。 

# 6.spring 具体的使用方式

在spring 中 我们使用aop 主要也分为xml配置的方式和使用注解的方式。

使用xml 方式  我们需要使用到一个非常重要的标签`<aop:config></aop:config>`,一个`<aop:config>`元素可以包含`pointcut, advisor, and aspect elements `

`advisor` :  能够将`Advice`以更为复杂的方式织入到目标对象中，是将`Advice`包装为更复杂切面的装配器。

## 6.1 Aspect

### 6.1.1 xml 方式

 ```xml
<aop:config>
    <aop:aspect id="myAspect" ref="aBean">
        ...
    </aop:aspect>
</aop:config>

<bean id="aBean" class="...">
    ...
</bean>
 ```

### 6.1.2 @Aspect 方式

## 6.2 PointCut

### 6.2.1 xml 方式

```xml
<aop:config>

    <aop:pointcut id="businessService"
        expression="execution(* com.xyz.myapp.service.*.*(..))"/>

</aop:config>
```

也可以在  `Aspect` 中声明`PointCut`

```XML
<aop:config>

    <aop:aspect id="myAspect" ref="aBean">

        <aop:pointcut id="businessService"
            expression="execution(* com.xyz.myapp.service.*.*(..))"/>

        ...

    </aop:aspect>

</aop:config>
```

**expression的指示符有很多类型**：详情参考[支持的切入点指示符](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-pointcuts-designators),[常见切入点表达式 ](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-pointcuts-examples)

**在多个表达式之间可以使用 ||,or 表示 或，使用 &&,and表示 与，使用 ！表示 非** 

#### 方法描述匹配

* `execution`:   **使用频率最高指示符**，执行表达式的格式是 ：

```java
execution( modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)throws-pattern? )
```

从正则表达式中我们可以看出 `ret-type-pattern` ， `name-pattern` ，`param-pattern` 是必须的

`ret-type-pattern`:标识方法的返回值，需要使用全路径的类名如java.lang.String,也可以为*表示任何返回值； 　　　　

`name-pattern`:指定方法名, *  代表所有方法 ,  set* , 代表以set开头的所有方法. 　　　　

`param-pattern`:指定方法参数(声明的类型),(..)代表所有参数,(*)代表一个参数, ( * , String) 代表第一个参数为任何值,第二个为String类型. 

`modifiers-pattern`: 描述访问控制符 比如 `public`.....

`throws-pattern`: 描述抛出什么样的异常

 表达式例子如下： 

- 执行任何公共方法：

```java
execution(public * *(..))
```

- 名称以“set”开头的任何方法的执行：

```java
execution(* set*(..))
```

- 执行`AccountService`接口定义的任何方法：

```java
execution(* com.xyz.service.AccountService.*(..))
```

- 执行service包中定义的任何类的任何方法：

```java
execution(* com.xyz.service.*.*(..))
```

- 执行service包或子包中定义的任何方法：

```java
execution(* com.xyz.service..*.*(..))
```

#### 方法参数描述匹配

* `args` : **更常用于绑定形式** （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了） 

  * 任何连接点，它接受一个参数，并且在运行时传递的参数是`Serializable` 

  ```
  args(java.io.Serializable)
  ```

* `@args` : **也可以用于绑定形式** （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了） 

#### 目标类匹配

* `target` ：**更常用于绑定形式**  （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了）

  * 目标对象实现AccountService接口的任何连接点：
  ```java
    target(com.xyz.service.AccountService)
  ```

* `@target` : **也可以用于绑定形式** （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了）

  - 目标对象具有`@Transactional`注释的任何连接点：

  ```java
  @target(org.springframework.transaction.annotation.Transactional)
  ```

* `within`: （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了） 

  * 服务包中的任何连接点：

  ```java
  within(com.xyz.service.*)
  ```

  - 服务包或子包中的任何连接点：	

  ```java
  within(com.xyz.service..*)
  ```

* `@within` :**也可以用于绑定形式** （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了） 

  - 任何连接点（仅在Spring AOP中执行方法），其中目标对象的声明类型具有`@Transactional`注释：

  ```java
  @within(org.springframework.transaction.annotation.Transactional)
  ```

```
@within(org.springframework.transaction.annotation.Transactional)
```

```
@within(org.springframework.transaction.annotation.Transactional)
```

```
@within(org.springframework.transaction.annotation.Transactional)
```

#### 当前AOP代理对象类型匹配

* `this`:  **更常用于绑定形式** （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了） 

  * 代理实现`AccountService`接口的任何连接点

  ```java
  this(com.xyz.service.AccountService)
  ```

#### 标有...注解的方法匹配 

* `@annotation` : **也可以用于绑定形式** （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了） 

#### 特定的命名Spring bean 

* 匹配特定的bean, 通过id 或 name

```java
bean(id or Name)
```

