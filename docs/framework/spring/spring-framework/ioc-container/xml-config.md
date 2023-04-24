---
title: XML Configuration
tags: [java, spring, spring-configuration]
sidebar_label: XML Configuration
sidebar_position: 4
---

# 基于XML的配置

## 基本流程

### 编写XML配置元数据

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

### 实例化IoC容器并使用

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

## 创建Bean

创建Bean有几种方式：使用构造器、使用static工厂方法、使用一个实例的工厂方法。

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}

public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

```xml
<!-- Instantiation with a Constructor -->
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>

<!-- Instantiation with a Static Factory Method -->
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>


<!-- Instantiation by Using an Instance Factory Method -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>
<bean id="clientService" factory-bean="serviceLocator" factory-method="createClientServiceInstance"/>
<bean id="accountService" factory-bean="serviceLocator" factory-method="createAccountServiceInstance"/>
```

### Bean的Name

```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

## 依赖注入

### 基于构造器的依赖注入

如果入参类型可以推导，可使用如下方式：
```java
public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>
    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

如果入参不能推导，可使用如下方式：
```java
public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private final int years;

    // The Answer to Life, the Universe, and Everything
    private final String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

```xml
<!-- 1. 指定类型的写法 -->
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>

<!-- 2. 使用index的写法 -->
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>

<!-- 3. 使用参数name的写法 -->
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

工厂实例注入样例：
```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

### 基于Setter方法的依赖注入

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

```xml
<bean id="simpleMovieLister" class="xxx.SimpleMovieLister">
    <property name="movieFinder" ref="movieFinder"/>
</bean>
<bean id="movieFinder" class="xxx.MovieFinder"/>
```

### 更多配置用法

#### 简单属性值注入

通过`<property/>`元素注入属性值：
```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="misterkaoli"/>
</bean>
```

通过**p-namespace**方式注入属性值：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="misterkaoli"/>
</beans>
```

类似地，可以使用**c-namespace**方式进行构造参数的注入。

也可以通过配置`java.util.Properties`实例：
```xml
<bean id="mappings"
    class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

#### idref元素

下面两种方式在runtime是等价的。但在部署时，第一种优于第二种形式，因为`idref`标签会让容器在部署的时候检验被引用的bean是否实际存在，而第二种却没有校验。

```xml
<bean id="theTargetBean" class="..."/>
<bean id="theClientBean" class="...">
    <!-- 方式1 -->
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
    <!-- 方式2 -->
    <property name="targetName" value="theTargetBean"/>
</bean>
```

#### 引用其他Bean

使用`<ref/>`元素是最普遍引用其他bean的形式。
```xml
<ref bean="someBean"/>
```

当涉及到跨容器引用的时候，`parent`属性用于引用父容器的bean。
```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required here -->
</bean>
```
```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

#### 内部Beans

在`<property/>`或者`<constructor-arg/>`元素中可以使用`<bean/>`定义内部的bean：
```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部的bean定义不需要定义ID或者name，如果指定了，容器也不会使用这个值作为标识符。容器也会忽略scope。

#### 集合

`<list/>`、`<set/>`、`<map/>`和`<props/>`元素用于设置对应类型`List`、`Set`、`Map`和`Properties`的属性和参数。
```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

##### 集合合并

使用`merge="true"`可以对`<list/>`、`<set/>`、`<map/>`和`<props/>`的值进行合并：

```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```

合并后的结果为：
```text
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

##### 类型转换

在java中指定集合的泛型参数类型：
```java
public class SomeClass {
    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

集合的属性值类型会根据泛型类型做自动转换：
```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```

### Null和Emtpy字符串值

```xml
<bean class="ExampleBean">
    <!-- exampleBean.setEmail(""); -->
    <property name="email" value=""/>

    <!-- exampleBean.setEmail(null); -->
    <property name="email">
        <null/>
    </property>
</bean>
```

## 更多依赖配置

### 控制Bean加载顺序

如果一个bean依赖另外一个bean，通常来说，使用`<ref/>`元素就可以引用另外一个bean，从而控制依赖的顺序。但是，在有些情况下，两个bean没有属性或者参数上的依赖，而是静态初始化的，在逻辑上两者有依赖关系，那么可以通过`depends-on`属性来显式地指定一个或多个bean在这个bean初始化之前先初始化。`depends-on`属性可以控制初始化的顺序，同时也能控制销毁的顺序，但是它只针对singleton的bean。

```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

### 懒加载Bean

默认地，`ApplicationContext`的实现会在初始化的过程中创建并配置所有的单例beans。如果你希望IoC容器在你需要这个bean的时候在进行初始化，那么可以使用懒加载的方式，即指定`lazy-init`属性，如下：

```xml
<!-- 全局指定懒加载beans -->
<!-- <beans default-lazy-init="true"> -->
<beans>
    <bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
    <bean name="not.lazy" class="com.something.AnotherBean"/>
</beans>
```

值得注意的是，如果一个指定为懒加载的bean，被另外一个非懒加载的bean依赖，那么也会在容器初始化的时候进行初始化bean并注入给依赖的bean。也就是懒加载在这种情况会失效。

### 自动注入

在定义bean时我们可以通过bean元素的`autowire`属性来指定是否需要给当前bean来自动注入其所关联的bean。`autowire`属性的可选值有四个：

| 模式 | 说明 |
| --- | ----|
| `no` | 默认值。表示不进行自动注入。 |
| `byName` | 根据名称进行自动注入。如`beanA`有一个`setBeanB()`方法，指定`autowire="byname"`时Spring将自动在`bean`容器中寻找名为`"beanB"`的bean通过`setBeanB`方法自动注入给`beanA`。 |
| `byType` | 根据类型进行自动注入。如`beanA`有一个`setBeanB(BeanB b)`方法，指定`autowire="byType"`时Spring将自动在bean容器中类型为`BeanB`的bean通过`setBeanB()`方法注入给`beanA`。但是如果此时容器中有两个以上类型为`BeanB`的bean时就将抛出异常。 |
| `constructor` | 等同于`byType`，只是当指定`autowire="constructor"`时表示将通过构造方法根据类型进行自动注入。 |

```java
public class Example {
    private final ClassA classA;

    public Example(ClassA classA) {
        this.classA = classA;
    }
}
```

```xml
<bean id="example" class="xxx.Example" autowire="constructor"/>
<bean name="classA" class="xxx.ClassA"/>
```

#### default-autowire

相当于全局指定注入模式。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
    default-autowire="byName">

</beans>
```

#### autowire-candidate和default-autowire-candidates

`autowire-candidate`是配合`autowire`使用的，是定义在`bean`上的一个属性，表示是否作为自动注入`bean`的候选者，默认为`true`，即默认所有的`bean`都可以是自动注入机制的候选者。有时候我们可能不希望某一个`bean`经过自动注入机制注入给其它`bean`，这个时候我们就可以指定该`bean`的`autowire-candidate`属性为`false`。比如有一个`beanA`定义了需要根据类型自动注入一个类型为`BeanB`的`bean`，但是容器中存在两个类型为`BeanB`的`bean`，分别为`beanB1`和`beanB2`，这种情况`Spring`是不能自动注入的。实际上我们想注入给`beanA`的是`beanB1`，这个时候除了像之前所说的显示的将`beanB1`注入给`beanA`外，我们还可以通过在`beanB2`上定义`autowire-candidate="false"`将`beanB2`排除在自动注入的候选者行列。`autowire-candidate`只对自动注入起作用，对于手动注入而言，无论`autowire-candidate`是什么值，手动注入都是会起作用的。

`default-autowire-candidates`的作用就像`default-autowire`的作用类似，也是用来定义全局的`autowire-candidate`行为的，其可选值也有`ture`和`false`两种，默认为`true`。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
    default-autowire="byName"
    default-autowire-candidates="true">

    <bean name="example" class="xxx.Example" autowire-candidate="false">
</beans>
```

### 方法注入

大多数情况下，大多数在容器中的`beans`都是单例。当合作的多个`beans`之间的生命周期不一样的时候，比如单例`beanA`需要使用非单例`beanB`，或许就是`beanA`的每次方法调用，使用之前的方法并不能实现这种需求，因为容器不能为`beanA`提供一个全新的`beanB`的实例。

一种方式就是放弃IoC，可以为`beanA`实现`ApplicationContextAware`接口，然后调用容器的`#getBean(nameB)`接口，然后就可以获取到新的实例，如下：
```java
// Spring-API imports

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

#### Lookup方法注入

**Lookup**方法注入是容器提供的一种能力，它可以重写容器管理的`bean`的方法并返回另一个满足`name`的`bean`作为查询结果。Spring框架通过CGLIB库动态生成一个子类来重写这个方法，以达到方法注入的能力。

`CommandManager#createCommand()`是一个实际的例子，每次调用这个方法将会创建一个全新的bean实例。
```java
// no more Spring imports!
public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

使用Lookup方法注入的注意点：
* bean对应的类不能是final的，方法也不能是final的。
* 方法签名要满足`<public|protected> [abstract] <return-type> theMethodName(no-arguments);`。
* 不适应于工厂方法，特别是注解了`@Bean`的方法。


#### 任意方法替换

另一种不太常用的方式就是方法替换，通过实现`MethodReplacer`接口来替换原有方法的逻辑。

```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```
```java
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

## 更多配置细节

### bean的scope配置

```xml
<!-- 默认scope为singleton -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```
