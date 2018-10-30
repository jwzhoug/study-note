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

  **许多AOP框架（包括Spring）将建议建模为*拦截器*，在连接点周围维护一系列拦截器。** 

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

**下面的内容按照章节顺序讲解配置其实是这样一个过程：先配置一个切面`Aspect`，切面中包含了切入点`PointCut`，切入点中采用不同的表达式筛选出了对应的连接点`JoinPoint`，接下来就是定义这些`JoinPoint`,需要执行的 处理逻辑`Advice`,**

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

修饰在类上，注意这个类同时需要用到`@Component` 注解，因为这个注解不能通过类路径自动检测来发现（当然你也可以在xml配置这个bean,来达到使用 `@Component`注解的效果）

关于注解方面的内容可以查看我的另一篇文章[spring 注解驱动](https://github.com/Alan-Jun/study-note/blob/master/spring/Spring%20Annotation%20Driver.md)

`@Aspect`和 xml 配置的方式是相同的功能，不过是通过注解在类上来实现了。

**他可能包含方法和字段。还可能包含`PointCut`，`Advice`和`Introdution`声明。** 

**要使用Java启用@AspectJ支持**

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

##### 使用XML配置启用@AspectJ支持

要使用基于XML的配置启用@AspectJ支持

```xml
<aop:aspectj-autoproxy/>
```

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

`expression `属性中的表达式请参考 6.2.3 

### 6.2.2 @Pointcut 方式

定义切入点：（注意方法返回类型必须是void）

```java
@Pointcut("execution(* transfer(..))")
private void anyOldTransfer() {...}
```

注解中使用的 表达式 参考 6.2.3 

**由于ASpectJ 是编译期的AOP，所以他在检查代码并匹配连接点与切入点的代价是较为昂贵的，为了降低这样的代价，我们需要尽量进行明确的指定,详情请看 官网 [写出好的切入点](https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-common-pointcuts)**

### 6.2.3 expression的指示符类型

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

 **表达式例子如下**： 

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

  * 它接受一个参数，并且传递的参数的运行时类型具有`@Classified`注释：

  ```
  @args(com.xyz.security.Classified)
  ```

#### 目标类匹配

* `target` ：**更常用于绑定形式**  （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了）

  * 目标对象实现AccountService接口的任何连接点：
  ```java
    target(com.xyz.service.AccountService)
  ```

* `@target` : **也可以用于绑定形式** （仅在 Spring AOP 这样使用 ，如果在spring中使用 AspectJ 这里就不能这样用了）

  - **目标对象**具有`@Transactional`注释的任何连接点：

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

  - **目标对象**具有`@Transactional`注释的任何连接点：：

  ```java
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

  - 其中**执行方法**具有 `@Transactional`注释的任何连接点：

  ```
  @annotation(org.springframework.transaction.annotation.Transactional)
  ```

#### 特定的命名Spring bean 

* 匹配特定的bean, 通过id 或 name

  ```java 
  bean(id or Name)
  ```

* 具有与通配符表达式匹配的名称的Spring bean

  ```
  bean(*name or *id)
  ```

## 6.3 Advice

### 6.3.1 xml

和第三节介绍的一样，`Advice`的5种类型下面都会有

- `before advice`: 在`JoinPoint`之前执行但不能阻止执行流程进入`JoinPoint`的`Advice`（除非它(`Advice`)抛出异常）。 

  ```xml
  <aop:aspect id="beforeExample" ref="aBean">
  
      <aop:before
          pointcut-ref="dataAccessOperation"
          method="doAccessCheck"/>
  
      ...
  
  </aop:aspect>
  ```

  `pointcut-ref ` 指向的是 定义的`pointcut` 的 `id`。还可以使用 内联的方式定义

  ```xml
  <aop:aspect id="beforeExample" ref="aBean">
  
      <aop:before
          pointcut="execution(* com.xyz.myapp.dao.*.*(..))"
          method="doAccessCheck"/>
      ...
  
  </aop:aspect>
  ```

  `method` 指定的方法是 `<aop:aspect id="beforeExample" ref="aBean">` 中`ref`指定的bean中定义的方法

- `after returning advice`: 在`JionPoint`正常完成后执行的*建议`Advice`

  ```xml
  <aop:aspect id="afterReturningExample" ref="aBean">
  
      <aop:after-returning
          pointcut-ref="dataAccessOperation"
          method="doAccessCheck"/>
  
      ...
  
  </aop:aspect>
  ```

  method 指定的方法带有 returnning指定的参数的方式，可以参考这个博客的使用方式https://www.cnblogs.com/ssslinppp/p/4633496.html

  ```xml
  <aop:aspect id="afterReturningExample" ref="aBean">
  
      <aop:after-returning
          pointcut-ref="dataAccessOperation"
          returning="retVal"
          method="doAccessCheck"/>
  
      ...
  
  </aop:aspect>
  ```

- `after throwing advice`: 如果方法是抛出异常退出，则执行 `Advice`

  ```xml
  <aop:aspect id="afterThrowingExample" ref="aBean">
  
      <aop:after-throwing
          pointcut-ref="dataAccessOperation"
          method="doRecoveryActions"/>
  
      ...
  
  </aop:aspect>
  ```

  method 指定的方法带有 returnning指定的参数的方式，可以参考这个博客的使用方式https://blog.csdn.net/owen_william/article/details/50812780

  ```xml
  
  <aop:aspect id="afterThrowingExample" ref="aBean">
  
      <aop:after-throwing
          pointcut-ref="dataAccessOperation"
          throwing="ex"
          method="doRecoveryActions"/>
  
      ...
  
  </aop:aspect>
  ```

- `after (finally) advice`: 无论`JoinPoint`退出的方式（正常或异常返回），都要执行`Advice` 

- `around advice`: 围绕`JoinPoint`的`Advice`。这是最有力的`Advice`。`around advice`可以在方法调用之前和之后执行自定义行为。

  ```xml
  <aop:aspect id="aroundExample" ref="aBean">
  
      <aop:around
          pointcut-ref="businessService"
          method="doBasicProfiling"/>
  
      ...
  
  </aop:aspect>
  ```

  `doBasicProfiling`  方法的实现：

  ```java
  public Object doBasicProfiling(ProceedingJoinPoint pjp) throws Throwable {
     // 在连接点前 做点什么
      Object retVal = pjp.proceed();// 这里相当于调用 我们实际的方法（连接点）
     // 在连接点后 做点什么
      return retVal;
  }
  ```

  **上面方法 抛出的 throws Throwable 异常 也可以在方法内部处理掉**

  这个已经很类似 我们使用动态代理时候的代码

  我们看一下 动态代理的代码：

  *  cglib

    ```java
    public class CglibProxy implements MethodInterceptor {
    
       private Enhancer enhancer = new Enhancer();
       
       public Object getProxy(Class clazz){
          //设置创建子类的类
          enhancer.setSuperclass(clazz);
          enhancer.setCallback(this);
          
          return enhancer.create();
       }
       
       /**
        * 拦截所有目标类方法的调用
        * obj  目标类的实例
        * m   目标方法的反射对象
        * args  方法的参数
        * proxy代理类的实例
        */
       @Override
       public Object intercept(Object obj, Method m, Object[] args,
             MethodProxy proxy) throws Throwable {
          System.out.println("日志开始...");
          //代理类调用父类的方法
          proxy.invokeSuper(obj, args);
          System.out.println("日志结束...");
          return obj;
       }
    
    }
    ```

    

  * jdk 动态代理

    ```java
    public class TimeHandler implements InvocationHandler {
    
    	public TimeHandler(Object target) {
    		super();
    		this.target = target;
    	}
    
    	private Object target;
    	
    	@Override
    	public Object invoke(Object proxy, Method method, Object[] args)
    			throws Throwable {
    		long starttime = System.currentTimeMillis();
    		System.out.println("汽车开始行驶....");
    		method.invoke(target);
    		long endtime = System.currentTimeMillis();
    		System.out.println("汽车结束行驶....  汽车行驶时间："
    				+ (endtime - starttime) + "毫秒！");
    		return null;
    	}
    }
    
    ```

    **有没有发现这个最能够体现 我们AOP 的实现使用的是动态代理**

- `Advice parameters` :  如果您希望显式指定通知方法的参数名称（不依赖于前面描述的检测策略），那么这是使用`arg-names`属性完成的 

  

  

- 一种特殊的用法

  ```java
  package x.y.service;
  
  public interface FooService {
  
      Foo getFoo(String fooName, int age);
  }
  
  public class DefaultFooService implements FooService {
  
      public Foo getFoo(String name, int age) {
          return new Foo(name, age);
      }
  }
  ```

  上面的接口不是必须的，但是在接口编程中是推荐使用的

  接下来是方面。请注意，该`profile(..)`方法接受许多强类型参数，第一个参数恰好是用于继续方法调用的连接点：此参数的存在表明该参数 `profile(..)`将用作`around`建议： 

  ```java
  import org.aspectj.lang.ProceedingJoinPoint;
  import org.springframework.util.StopWatch;
  
  public class SimpleProfiler {
  
      public Object profile(ProceedingJoinPoint call, String name, int age) throws Throwable {
          StopWatch clock = new StopWatch("Profiling for '" + name + "' and '" + age + "'");
          try {
              clock.start(call.toShortString());
              return call.proceed();
          } finally {
              clock.stop();
              System.out.println(clock.prettyPrint());
          }
      }
  }
  ```

  最后，这是为特定连接点执行上述建议所需的XML配置：  

  ```xml
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="
          http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
  
      <!-- this is the object that will be proxied by Spring's AOP infrastructure -->
      <bean id="fooService" class="x.y.service.DefaultFooService"/>
  
      <!-- this is the actual advice itself -->
      <bean id="profiler" class="x.y.SimpleProfiler"/>
  
      <aop:config>
          <aop:aspect ref="profiler">
  
              <aop:pointcut id="theExecutionOfSomeFooServiceMethod"
                  expression="execution(* x.y.service.FooService.getFoo(String,int))
                  and args(name, age)"/>
  
              <aop:around pointcut-ref="theExecutionOfSomeFooServiceMethod"
                  method="profile"/>
  
          </aop:aspect>
      </aop:config>
  
  </beans>
  ```

### 6.3.2 注解的方式

和xml 方式相识，spring 中也存在相对应的注解，使用方式都差不多

* `@Before`:
* 

## 6.4  Introduction 

### 6.4.1 xml

```xml
<aop:aspect id="usageTrackerAspect" ref="usageTracking">

    <aop:declare-parents
        types-matching="com.xzy.myapp.service.*+"
        implement-interface="com.xyz.myapp.service.tracking.UsageTracked"
        default-impl="com.xyz.myapp.service.tracking.DefaultUsageTracked"/>

    <aop:before
        pointcut="com.xyz.myapp.SystemArchitecture.businessService()
            and this(usageTracked)"
            method="recordUsage"/>

</aop:aspect>
```

使用`aop:declare-parents`用于声明匹配类型具有新父级 

`types-matching `: 描述匹配的类型，上面的 +号表示包括子类，比如` com.xzy.myapp.service.MyService+`

匹配 MyService 以及 MyService 的子类 

### 6.4.2 注解的方式

## 6.5 advisor

# 6.6 spring AOP API

这事Spring AOP 的基础。https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#aop-api  有需要的时候请仔细阅读 官文的这部分内容

