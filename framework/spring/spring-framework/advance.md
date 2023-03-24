# 循环依赖

Spring循环依赖问题是指两个或多个bean之间相互依赖，形成了循环依赖的情况。这种情况下，Spring容器无法完成bean的实例化，会抛出异常。

## 循环依赖的出现

以下是解决Spring循环依赖产生的几种场景：
1. 通过构造函数注入：`A` 和 `B` 的构造参数都互相依赖。

```java
@Component
public class A {
    private B b;

    public A(B b) {
        this.b = b;
    }
}

@Component
public class B {
    private A a;

    public B(A a) {
        this.a = a;
    }
}
```

2. 通过setter注入：`A` 和 `B` 的setter方法参数都互相依赖。

```java
@Component
public class A {
    private B b;

    @Autowired
    public void setB(B b) {
        this.b = b;
    }
}

@Component
public class B {
    private A a;

    @Autowired
    public void setA(A a) {
        this.a = a;
    }
}
```

3. 通过 `@DependensOn` 依赖：`A` 和 `B` 的bean构造都互相依赖。

```java
@DependsOn("b")
@Component
public class A {
    private B b;

    public void setB(B b) {
        this.b = b;
    }
}

@DependsOn("a")
@Component
public class B {
    private A a;

    public void setA(A a) {
        this.a = a;
    }
}
```

4. 使用@Lazy注解：在需要循环依赖的bean上添加@Lazy注解，使其变为懒加载，这样Spring容器会先实例化其他bean，然后再实例化需要循环依赖的bean。
5. 使用代理对象：使用代理对象来解决循环依赖问题，即在需要循环依赖的bean上添加@Scope(\"prototype\")注解，使其变为原型模式，然后使用代理对象来获取bean实例。

## 如何解决循环依赖

### 重新设计

如果bean之间存在循环引用，很有可能是代码设计问题，没有做好职责分离，你应该重新设计，抽离共有的一部分代码，这样就可以从根源上解决循环依赖问题。

### 使用@Lazy注解

### 使用@PostConstruct注解

### 使用ApplicationContextAware and InitializingBean
