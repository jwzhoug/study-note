# 1 介绍

*面向方面编程*（AOP）通过提供另一种思考程序结构的方式来补充面向对象编程（OOP）。OOP中模块化的关键单元是类，而在AOP中，模块化单元是*方面*。方面可以实现关注的模块化，例如跨越多种类型和对象的事务管理。

**实现AOP的方式有两种：**

* **预编译**：主要代表有 AspectH
* **动态代理**: springAOP , JbossAOP

**spring 中主实现方式是通过动态代理实现的，动态代理又主要包含两种方式：**

* **jdk的动态代理**
* **第三方包 cglib**

# 2 使用的场景

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



