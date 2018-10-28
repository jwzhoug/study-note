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

* 切面（`Aspect`）: 一个关注点的模块化（实现），这个关注点可能会横切多个对象。

* 连接点（`JoinPoint`）：程序执行过程中的某一个阶段点（比如方法的调用、异常的抛出等），在Spring AOP中，连接点*始终* 表示方法执行。 

* 通知（`Advice`）: 在`Aspect`的某一个特定的连接点上**执行的动作（处理逻辑）**，也就是向连接点注入的代码，类型包括：

  * `around advice`: 围绕`JoinPoint`的`Advice`。这是最有力的`Advice`。`around advice`可以在方法调用之前和之后执行自定义行为。**建议使用下面的特定类型的`Advice`,在必要情况下 在使用 `around advice`**
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

  

