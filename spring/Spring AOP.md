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

* 切面（`Aspect`）:
* 连接点（`Joinpoint`）：
* 通知（`Advice`）:
* 切入点（`PointCut`）:
* 引入/引用（`Introduction`）:匹配
* 目标对象（`Taget Object`）: 被一个或则多个`Aspect`所`Advice`的对象,也叫做 `Advice Object`，由于Spring AOP是使用运行时代理实现的，因此该对象始终是*代理*对象（`AOP Proxy Object`）。 
* AOP代理（`AOP Proxy`）: 由AOP框架创建的对象，用于实现`Aspect Contract`（`Advice`方法执行等功能）。在Spring Framework中，AOP代理将是JDK动态代理或CGLIB代理。 
* 织入（`Weaving`）: