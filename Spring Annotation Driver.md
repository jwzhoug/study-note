#  申明

内容来源于spring官网，我只是做一下笔记加深印象以及方便自己产找，以及后续使用中问题的记录。这里不介绍Spring IOC 基础的东西，以及XML的配置这戏额内容官网都有，需要的情趣查看 https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/core.html#beans

# 1 使用注解定义Bean

## 1.1 使用注解的前置条件：

* 使用`<context:component-scan base-package="xxx.xxx.xxx"/>`,**这样配置之后类上的注解，以及类中字段方法等的注解才能被spring发现兵注册成bean** 

  这个标签还有很多别的属性可以指定这里介绍一部分

  * `base-package=""`属性用来指定具体的包路径

  * `scoped-proxy=""` 这个属性指定使用什么杨的动态代理方式来实现，取值：
    * `interfaces` : 使用 jdk 的基于接口的动态代理
    * `targetClass` : 使用 cglib的基于类的方法拦截调用的动态代理
  * `name-generator=""`: 用于自定义命名规则的属性，1.4.1 节 会有介绍
  * `use-default-filters=""` : 默认为true , 会扫描指定下的所有被 1.2 节中说所注解修饰的类，并且注册成bean, 指定 false 请看 1.1.1节 的具体说明
  * `resource-pattern="" : 暂时不知道用法`
  * `scope-resolver=""` : 用于自定义Scope策略，具体的看 1.5.1 节

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <!--<bean id="bean2" class="com.stu.springdemo.annotation.resource.Bean1"/>-->
    
    <context:component-scan base-package="com.stu.springdemo.annotation"/>
</beans>
```

* `<context:annotation-config></context:annotation-config>`  这个标签只能在bean注册之后，费用与发现类中属性或则方法等的注解。

**总结**：推荐使用上面第一种方式。不推荐使用`<context:annotation-config></context:annotation-config>`

### 1.1.1 使用过滤器进行自定义扫描

`<context:component-scan/>`这个标签`use-default-filters="true"默认为true`,会扫描指定下的所有被 1.2节 中说所注解修饰的类，并且注册成bean，但是有的情况我们需要一些更灵活的使用，那么它有有两个子标签：

* `<context:include-filter type="" expression=""/>`:用于提供指定需要包含的项
* `<context:exclude-filter type="" expression=""/>` :用于提供指定需要过滤的项

| Filter Type | Examples Expression        | Description                                                  |
| :---------: | -------------------------- | ------------------------------------------------------------ |
| annotation  | org.example.SomeAnnotation | 符合SomeAnnoation的target class                              |
| assignable  | org.example.SomeClass      | 指定class或interface的全名                                   |
|   aspectj   | org.example..*Service+     | AspetJ语法                                                   |
|    regex    | org.example.Default.*      | 正则表达式                                                   |
|   custom    | org.example.MyTypeFilter   | Spring3新增自订Type,称作org.springframework.core.type.TypeFilter |

**注意：使用两个子标签，指定`<context:component-scan/>`的`use-default-filters="false"`,才有效果**

## 1.2 作用于类的注解

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

* `@Component` :作为最基础的注解，也说明他是一个通用的注解，可以用于定义任何的bean
* `@Configuration`: 配合@Bean 注解达到 让 java代码类似XMl配置bean一样的效果
* `@Repository` ：常用于修饰 Dao类(持久层)
* `@Service` ： 常用于修饰 Service类（服务层）
* `@Controller` ：常用于Controller类（控制层）
* 以及各种用`@Component`修饰的注解（包括自定义的注解），相当于他的子注解

## 1.3 作用方法的注解 `@Bean`

这个注解需要搭配 `@Configuration`注解使用，当然也可以搭配通用的`@Component`注解使用,效果和XMl中配置的效果类似，但是少了部分功能，这部分功能需要使用 别的注解来补充（这些注解后续会有介绍）

实例：

```java
public interface Animal {
    void eat();
}
```

```java
public class Lion implements Animal{

    @Value("鸡肉")
    private String foodName;

    public void initMethod(){
        System.out.println("I am a small lion");
    }
    public void destroyMethod(){
        System.out.println("I am a small lion,I am hungry");
    }
    @Override
    public void eat() {
        System.out.println("Lion eat "+foodName);
    }
}
```

```java
public class Tiger implements Animal{
    public void initMethod(){
        System.out.println("I am a small tiger");
    }
    @PostConstruct
    public void postConstruct(){
        System.out.println("I am a small tiger,PostConstruct");
    }
    @PreDestroy
    public void preDestroy(){
        System.out.println("I am a small tiger,I am digesting food.");
    }
    public void destroyMethod(){
        System.out.println("I am a small tiger,I am hungry");
    }
    @Override
    public void eat() {
        System.out.println("tiger eat meat");
    }
}
```

```java
@Configuration
public class ConfigyrationDemo {

    @Bean(name="tiger")
    public Tiger getTiger(){
        return new Tiger();
    }

    @Bean(name="lion",initMethod = "initMethod",destroyMethod = "destroyMethod")
    public Lion lion(){
        return new Lion();
    }
}
```

## 1.4 Bean 名称的生成策略

在使用 1.2 中所说的注解的时候如果不给注解显示的指定名称，spring中默认的beanName 的生成器 BeanNamegenerator 会使用类名且雷鸣第一个字母小写作为beanName.比如 类 BeanDemo,它的beanName=beanDemo。

### 1.4.1 自定义命名策略

实现BeanNamegenerator接口，并且包含一个无参数的构造方法。然后通过配置文件：

```xml
<context:component-scan base-package="xxx.xxx.xxx" name-generator="xxx.xxx.MyNamegenerator"/>
```

指定即可

## 1.5 作用域 @Scope

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {
    @AliasFor("scopeName")
    String value() default "";

    @AliasFor("value")
    String scopeName() default "";

    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}
```

和xml中的作用域相同，通过`@Scope`注解也能达到相同的效果，

**从源码可以看出这个注解可以用于方法和累上面，相对应的就是配合上文的 1.2节 和 1.3节 中的注解使用了，他们默认情况下都是单例的。**

```java
@Configuration
public class ConfigyrationDemo {

    @Bean(name="tiger")
    public Tiger getTiger(){
        return new Tiger();
    }

    @Bean(name="lion",initMethod = "initMethod",destroyMethod = "destroyMethod")
    @Scope("prototype")
    public Lion lion(){
        return new Lion();
    }
}
```

```java
@Component
@Scope("prototype")
public class BeanAnnotation {

    public void say(){
        System.out.println(" I am a spring bean by Component Annotation "+this);
    }

    public void myHashCode(){
        System.out.println(this.hashCode());
    }
}
```

### 1.5.1 自定义命名策略

实现ScopeMetaDataResolver接口，并且包含一个无参数的构造方法。然后通过配置文件：

```xml
<context:component-scan base-package="xxx.xxx.xxx" name-generator="xxx.xxx.MyScopeMetaDataResolverr"/>
```