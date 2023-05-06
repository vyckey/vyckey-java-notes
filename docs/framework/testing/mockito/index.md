---
title: Mockito
tags: [test, mockito]
sidebar_label: Mockito
sidebar_position: 1
---

# Mockito

## 基本概念

> Mocking is the act of describing (mandatory) interactions between the object under specification and its collaborators.

**`Mocking`** 是描述在规范下的对象和其合作者之间的（强制）交互行为，简单来说就是模拟的一个虚拟类，可以模拟方法调用的结果返回，以及方法的行为。需要注意的是，对于未模拟的方法返回值会以默认值的方式处理，数值基本类型返回 `0` ，布尔类型返回 `false` ，引用类型返回 `null` 。

> Stubbing is the act of making collaborators respond to method calls in a certain way. When stubbing a method, you don’t care if and how many times the method is going to be called; you just want it to return some value, or perform some side effect, whenever it gets called.

**`Stubbing`** 是让协作者以某种方式响应方法调用的行为。插桩方法时，您不关心该方法是否会被调用以及调用多少次；您只希望它在被调用时返回一些值或执行一些副作用。简单来说，`Stubbing` 也是一个模拟的虚拟类，只返回模拟的结果，并不提供交互验证（方法是否会被调用以及调用多少次），使用插桩只能验证状态（例如测试方法返回的结果数据是否正确，list大小等，是否符合断言）。总之，`Stubbing` 比 `Mocking` 简单一些，只验证返回结果使用 `Stubbing` 足矣。

> A spy is always based on a real object. Hence you must provide a class type rather than an interface type, along with any constructor arguments for the type. If no constructor arguments are provided, the type’s no-arg constructor will be used. Method calls on a spy are automatically delegated to the real object. Likewise, values returned from the real object’s methods are passed back to the caller via the spy.

**`Spy`** 总是基于真实的对象。 因此，您必须提供类类型而不是接口类型，以及该类型的任何构造函数参数。如果未提供构造函数参数，则将使用该类型的无参数构造函数。对间谍的方法调用会自动委托给真实对象。同样，从真实对象的方法返回的值通过间谍传递回调用者。区别于 `Mocking` 和 `Stubbing` ，`Spy` 会调用真是的对象，只是对部分方法进行模拟。

## Stubbing

### Stubing 方法返回结果

下面的代理示例mock了一个 `LinkedList` 类对象的 `get` 方法，对于未mock的方法参数，将采用默认值作为结果，也就是数值基本类型返回 `0` ，布尔类型返回 `false` ，引用类型返回 `null` 。

```java
//You can mock concrete classes, not just interfaces
LinkedList mockedList = mock(LinkedList.class);

//stubbing
when(mockedList.get(0)).thenReturn("first");
when(mockedList.get(1)).thenThrow(new RuntimeException());

//following prints "first"
System.out.println(mockedList.get(0));

//following throws runtime exception
System.out.println(mockedList.get(1));

//following prints "null" because get(999) was not stubbed
System.out.println(mockedList.get(999));

//Although it is possible to verify a stubbed invocation, usually it's just redundant
//If your code cares what get(0) returns, then something else breaks (often even before verify() gets executed).
//If your code doesn't care what get(0) returns, then it should not be stubbed.
verify(mockedList).get(0);
```

对于 `void` 返回类型的参数可mock异常的抛出：

```java
doThrow(new RuntimeException()).when(mockedList).clear();

//following throws RuntimeException:
mockedList.clear();
```

### Stubbing 连续调用（迭代风格）

针对一些迭代方法，Mockito 支持如下的插桩方式：

```java
when(mock.someMethod("some arg"))
    .thenThrow(new RuntimeException())
    .thenReturn("foo");

//First call: throws runtime exception:
mock.someMethod("some arg");

//Second call: prints "foo"
System.out.println(mock.someMethod("some arg"));

//Any consecutive call: prints "foo" as well (last stubbing wins).
System.out.println(mock.someMethod("some arg"));
```

```java
when(mock.someMethod("some arg"))
    .thenReturn("one", "two", "three");
```

```java
//All mock.someMethod("some arg") calls will return "two"
when(mock.someMethod("some arg"))
    .thenReturn("one")
when(mock.someMethod("some arg"))
    .thenReturn("two")
```

### 自定义回调返回值

实现 [`Answer`](https://javadoc.io/static/org.mockito/mockito-core/5.3.1/org/mockito/stubbing/Answer.html) 接口可以实现 Stubbing 回调功能。一般来说，更推荐使用 `thenReturn()` 或 `thenThrow()` 方法，代码更加清晰和简单。如果你有使用泛化回调的场景，可以使用该方式进行插桩。

```java
when(mock.someMethod(anyString())).thenAnswer(
    new Answer() {
        public Object answer(InvocationOnMock invocation) {
            Object[] args = invocation.getArguments();
            Object mock = invocation.getMock();
            return "called with arguments: " + Arrays.toString(args);
        }
    });

//Following prints "called with arguments: [foo]"
System.out.println(mock.someMethod("foo"));
```

### void 方法的特殊处理方式

对于 `void` 返回类型的方法，你可以使用 `doReturn()` 、 `doThrow()` 、 `doAnswer()` 、 `doNothing()` 、 `doCallRealMethod()` 等方法来进行插桩。如下示例：

```java
doThrow(new RuntimeException()).when(mockedList).clear();

//following throws RuntimeException:
mockedList.clear();
```

### 自定义未 Stubbing 方法的默认返回值

```java
Foo mock = mock(Foo.class, Mockito.RETURNS_SMART_NULLS);
Foo mockTwo = mock(Foo.class, new YourOwnAnswer());
```


## Mocking

### 验证方法调用

下面的代码示例mock了一个 `List` 作为示例，实际例子替换为用户待mock的类。下面的函数调用会被mock对象给记住，可以用来验证方法是否被调用。

```java
//Let's import Mockito statically so that the code looks clearer
import static org.mockito.Mockito.*;

//mock creation
List mockedList = mock(List.class);

//using mock object
mockedList.add("one");
mockedList.clear();

//verification
verify(mockedList).add("one");
verify(mockedList).clear();
```

### 参数匹配

Mockito 使用对象的方法 `equals` 来判断参数是否匹配，除此之外，还可以一些额外的参数匹配匹配方式提供。官方提供了内建的 [ArgumentMatchers](https://javadoc.io/static/org.mockito/mockito-core/5.3.1/org/mockito/ArgumentMatchers.html) 和 [MockitoHamcrest](https://javadoc.io/static/org.mockito/mockito-core/5.3.1/org/mockito/hamcrest/MockitoHamcrest.html) 实现可供使用，如果想自定义参数匹配，可实现 [ArgumentMatcher](https://javadoc.io/static/org.mockito/mockito-core/5.3.1/org/mockito/ArgumentMatcher.html) 接口。一般来说，出于合理性，更建议重写 `equals` 方法以实现自然地参数匹配。

```java
//stubbing using built-in anyInt() argument matcher
when(mockedList.get(anyInt())).thenReturn("element");

//stubbing using custom matcher (let's say isValid() returns your own matcher implementation):
when(mockedList.contains(argThat(isValid()))).thenReturn(true);

//following prints "element"
System.out.println(mockedList.get(999));

//you can also verify using an argument matcher
verify(mockedList).get(anyInt());

//argument matchers can also be written as Java 8 Lambdas
verify(mockedList).add(argThat(someString -> someString.length() > 5));
```

### 参数捕获

`ArgumentCaptor` 适用于自定义参数匹配不太可能重复使用，你需要断言参数值做完整的验证。

```java
ArgumentCaptor<Person> argument = ArgumentCaptor.forClass(Person.class);
verify(mock).doSomething(argument.capture());
assertEquals("John", argument.getValue().getName()); 
```

### 调用次数验证

Mockito 提供mock方法的参数调用次数验证，包含至少、至多次数或从未调用。

```java
//using mock
mockedList.add("once");

mockedList.add("twice");
mockedList.add("twice");

mockedList.add("three times");
mockedList.add("three times");
mockedList.add("three times");

//following two verifications work exactly the same - times(1) is used by default
verify(mockedList).add("once");
verify(mockedList, times(1)).add("once");

//exact number of invocations verification
verify(mockedList, times(2)).add("twice");
verify(mockedList, times(3)).add("three times");

//verification using never(). never() is an alias to times(0)
verify(mockedList, never()).add("never happened");

//verification using atLeast()/atMost()
verify(mockedList, atMostOnce()).add("once");
verify(mockedList, atLeastOnce()).add("three times");
verify(mockedList, atLeast(2)).add("three times");
verify(mockedList, atMost(5)).add("three times");
```

### 调用顺序的验证

Mockito 可提供方法调用顺序的验证功能，如下示例：

```java
// A. Single mock whose methods must be invoked in a particular order
List singleMock = mock(List.class);

//using a single mock
singleMock.add("was added first");
singleMock.add("was added second");

//create an inOrder verifier for a single mock
InOrder inOrder = inOrder(singleMock);

//following will make sure that add is first called with "was added first", then with "was added second"
inOrder.verify(singleMock).add("was added first");
inOrder.verify(singleMock).add("was added second");

 // B. Multiple mocks that must be used in a particular order
List firstMock = mock(List.class);
List secondMock = mock(List.class);

//using mocks
firstMock.add("was called first");
secondMock.add("was called second");

//create inOrder object passing any mocks that need to be verified in order
InOrder inOrder = inOrder(firstMock, secondMock);

//following will make sure that firstMock was called before secondMock
inOrder.verify(firstMock).add("was called first");
inOrder.verify(secondMock).add("was called second");

// Oh, and A + B can be mixed together at will
```

也可以验证一些方法从来没被调用过：

```java
//using mocks - only mockOne is interacted
mockOne.add("one");

//ordinary verification
verify(mockOne).add("one");

//verify that method was never called on a mock
verify(mockOne, never()).add("two");
```

```java
//using mocks
mockedList.add("one");
mockedList.add("two");

verify(mockedList).add("one");

//following verification will fail
verifyNoMoreInteractions(mockedList);
```

> 警告：不建议在每个测试方法中使用 `verifyNoMoreInteractions` 方法，虽然它是一个很方便的测试断言，但是滥用会导致过度指定和难以维护，建议只在相关的时候再使用。

### 超时验证

```java
//passes when someMethod() is called no later than within 100 ms
//exits immediately when verification is satisfied (e.g. may not wait full 100 ms)
verify(mock, timeout(100)).someMethod();
//above is an alias to:
verify(mock, timeout(100).times(1)).someMethod();

//passes as soon as someMethod() has been called 2 times under 100 ms
verify(mock, timeout(100).times(2)).someMethod();

//equivalent: this also passes as soon as someMethod() has been called 2 times under 100 ms
verify(mock, timeout(100).atLeast(2)).someMethod();
```

### 静态方法的 Mocking

```java
assertEquals("foo", Foo.method());
try (MockedStatic mocked = mockStatic(Foo.class)) {
    mocked.when(Foo::method).thenReturn("bar");
    assertEquals("bar", Foo.method());
    mocked.verify(Foo::method);
}
assertEquals("foo", Foo.method());
```

### 对象构造的 Mocking

```java
assertEquals("foo", new Foo().method());
try (MockedConstruction mocked = mockConstruction(Foo.class)) {
    Foo foo = new Foo();
    when(foo.method()).thenReturn("bar");
    assertEquals("bar", foo.method());
    verify(foo).method();
}
assertEquals("foo", new Foo().method());
```

## Spying

### Spying 真实对象

可以创建真实对象的间谍，没被插桩的函数将调用实际对象的方法。间谍的含义可以理解为部分 Mocking ，它核心的点是对真实对象的包装。

```java
List list = new LinkedList();
List spy = spy(list);

//optionally, you can stub out some methods:
when(spy.size()).thenReturn(100);

//using the spy calls *real* methods
spy.add("one");
spy.add("two");

//prints "one" - the first element of a list
System.out.println(spy.get(0));

//size() method was stubbed - 100 is printed
System.out.println(spy.size());

//optionally, you can verify
verify(spy).add("one");
verify(spy).add("two");

//Impossible: real method is called so spy.get(0) throws IndexOutOfBoundsException (the list is yet empty)
when(spy.get(0)).thenReturn("foo");

//You have to use doReturn() for stubbing
doReturn("foo").when(spy).get(0);
```

## 高级用法

### 注解简化

`@Mock` 注解可以减少 Mock 对象的代码量，让测试类更加可读，验证错误更加可读。

```java
public class ArticleManagerTest {
    @Mock private ArticleCalculator calculator;
    @Mock private ArticleDatabase database;
    @Mock private UserProvider userProvider;
    private ArticleManager manager;

    @org.junit.jupiter.api.Test
    void testSomethingInJunit5(@Mock ArticleDatabase database) {
        // do something
    }
}
```

很重要的是，你需要在基类或测试运行器中开启 `Mockito` 注解。你可以使用内建的运行器 `MockitoJUnitRunner` 或者一个规则 `MockitoRule` 。更多地，可以参考 [`MockitoAnnotations`](https://javadoc.io/static/org.mockito/mockito-core/5.3.1/org/mockito/MockitoAnnotations.html) 。

```java
MockitoAnnotations.openMocks(testClass);
```

# PowerMock

# 参考资料

* [Mockito官网](https://site.mockito.org/)
* [GitHub - mockito](https://github.com/mockito/mockito)
* [Mockito Java Docs](https://javadoc.io/static/org.mockito/mockito-core/5.3.1/org/mockito/Mockito.html)