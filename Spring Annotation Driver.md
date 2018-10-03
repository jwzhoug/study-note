#  申明

内容来源于spring官网，我只是做一下笔记加深印象以及方便自己产找，以及后续使用中问题的记录。这里不介绍Spring IOC 基础的东西，以及XML的配置这戏额内容官网都有，需要的情趣查看 https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans

# 1 使用注解定义Bean

定义Bean 的注解有很多，大致分类两类：

## 1.1 作用于类的注解

这类注解的最基础的注解（不考虑spring 对JSR 330支持的情况下 ）是spring的 `@Component` 这个注解（元注解）。然后就是用这个注解修饰的注解了，比如`@Repository`  ，`Servise`

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}
```

可以看出他们都是使用了`@Component`这个注解（元注解）修饰的注解，所以我才会说`@Component`这是一个最基础的注解

总结一下这样的注解有很多，我先列举一部分我目前知道的，后续会做补充

* `@Componen` :作为最基础的注解，也说明他是一个通用的注解，可以用于定义任何的bean
* `@Configuration`: 配合@Bean 注解达到 让 java代码类似XMl配置bean一样的效果
* `@Repository``
* ``
* ``
* ``
* ``
* 

## 1.2 作用方法的注解

