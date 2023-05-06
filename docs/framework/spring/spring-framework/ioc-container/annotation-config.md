---
title: Annotation Configuration
tags: [java, spring, spring-configuration]
sidebar_label: Annotation Configuration
sidebar_position: 3
---

# 基于注解的配置

## 用于创建Bean的注解

### @Component

只能注解到class上，表示会创建一个这种类型的Bean，默认使用类名首字母小写的形式作为Bean的name，可手动指定name。

```java
@Component // 等价于@Component("classA")
public class ClassA {
    // ....
}
```

### @Repository

同`@Component`功能一样，唯一的区别是指示该类是一个Repository，和数据库相关的一个bean。

```java
@Repository
public class UserRepositoryImpl {
    // ....
}
```

### @Service

同`@Component`功能一样，唯一的区别是指示该类是一个Service，和`@Repository`类似。

### @Controller

同`@Component`功能一样，唯一的区别是指示该类是一个Controller，和`@Repository`类似。一般结合`@RequestMapping`等使用。

## 用于注入Bean的注解

### @Autowired

用于Bean的依赖注入，可以通过构造器、字段、setter方法、配置方法等，优先按java类型进行注入。

```java
public class AutowiredExample {
    private final Example1 example1;
    private final Example2 example2;
    /**
     * 字段注入方式，访问级别不必是public的
     */
    @Autowired
    private Example3 example3;
    private Example4 example4;
    private final List<Listener> listeners = Lists.newArrayList();

    /**
     * 构造函数的参数可以加@Autowired，但不是必须的。构造器不必是public的。
     */
    public AutowiredExample(Example1 example1, @Autowired Example2 example2) {
        this.example1 = example1;
        this.exmaple2 = example2;
    }

    /**
     * setter方法的注入，同字段注入的功能差不多，但不推荐字段注入，因为setter注入可以在没有spring的时候也使用。
     * 同样，访问级别不必是public的。required=false表示该类型的bean不是必须的，可以不存在。
     */
    @Autowired(required = false)
    public void setExample4(Example4 example4) {
        this.example4 = example4;
    }

    /**
     * Collection方式的集合注入，要求listeners是非empty的，如可以为empty，设置required=false的属性即可
     */
    @Autowired
    public void initListeners(List<Listener> listeners) {
        this.listeners.addAll(listeners);
    }

    /**
     * 同Collection注入方式类似，Map注入的Entry的Key为bean的name
     */
    @Autowired
    public void initListeners(Map<String, Listener> listenerMap) {
        this.listeners.addAll(listeners.values());
    }
}
```

需要注意的是`@Autowired`是使用`BeanPostProcessor`来实现的，即`AutowiredAnnotationBeanPostProcessor`，所以不能使用`@Autowired`来注入bean到`BeanPostProcessor`或者`BeanFactoryPostProcessor`的实现类中。

### @Qualifier

指定注入方式为name注入，一般结合`@Autowired`使用，等价于`@Resource`方式的注入。

### @Resource

指定注入方式为name注入，和`@Autowired`的区别主要就是这个点。另外值得一提的是，这个注解并不是spring的原生注解，是jdk提供的。

## 和生命周期有关的注解

### @PostConstruct

用于注解bean中待初始化调用的方法，顺序是构造函数->字段和setter注入->@PostConstruct方法。被注解的方法要求是无参的，访问级别不必是public。同一个类中可配置多个`@PostConstruct`注解，按顺序执行。另外，这个注解来源于javax.annotation包。

```java
public class Example {
    // ...

    @PostConstuct
    public void initFirst() {
        // do some initializing things
    }

    @PostConstuct
    public void initAfter() {
        // do some initializing things
    }
}
```

### @PreDestory

与`@PostConstruct`注解类似，用于bean在销毁前调用。

## 配置相关注解

### @Configuration

### @ComponentScan

### @Import

只能注解到类上，用于导入一个或多个 `Component` 。允许导入 `@Configuration` 注解的类，`ImportSelector` 和 `ImportBeanDefinitionRegistrar` 的实现类，也包含一些常规的 `Component` 类，功能类似于 `AnnotationConfigApplicationContext#register` 。

需要注意的是，在 `@ComponentScan` 扫描的包之内的 `Bean` ，没有必要单独进行 `import` ，只有当不在扫描包内，再加 `@Import` 注解进行导入。

```java
@Component
public class MysqlConnConfig {
    private String username;
    private String password;
}

@Configuration
@Import(value = {MysqlConnConfig.class})
public class MysqlConfiguration {
    // ...
}
```

### @Bean

## 改变Bean的作用范围的注解

### @Scope

指示bean的作用域，默认作用域是单例`ConfigurableBeanFactory.SCOPE_SINGLETON`，可以指定其他可选作用域，例如`ConfigurableBeanFactory.SCOPE_PROTOTYPE`等。