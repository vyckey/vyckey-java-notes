---
title: Jackson Annotation
tags: [java, jackson, anotation]
sidebar_label: Annotation Examples
sidebar_position: 3
---

[toc]

# Jackson注解使用样例

## 1.Jackson序列化注解

### 1.1 @JsonGetter

使用位置：`ANNOTATION_TYPE`、`METHOD`

对于non-static、无参和non-void返回的方法，可以使用该注解来作为一个逻辑getter方法。更加通用的注解`@JsonProperty`也同样可以实现相同的功能。

`@JsonGetter` 注解是 `@JsonProperty` 注解的替代方案，它将方法标记为 `getter` 方法。

在下面的示例中，我们指定方法 `getTheName()` 作为 `MyBean` 实体的 `name` 属性的 `getter` 方法：

```java
public class MyBean {
    public int id;
    private String name;

    @JsonGetter("myname")
    public String getTheName() {
        return name;
    }
}

@Test
public void whenSerializingUsingJsonGetter_thenCorrect() throws JsonProcessingException {
 
    MyBean bean = new MyBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
 
    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));
}
```

```json
{
    "id": 1,
    "myname":"My bean"
}
```

### 1.2 @JsonValue

使用位置：`ANNOTATION_TYPE`、`METHOD`

该注解用于指定序列化输出的值结果，要求是无参和non-void返回的方法。一个类最多只能有一个这样的注解，否则会抛出异常。一般结合`@JsonCreator`注解一起使用。

Example1:
```java
@Getter
@AllArgsConstructor
enum StatusEnum {
    NOT_START("not_start"),
    STARTED("started"),
    PAUSE("pause"),
    FINISHED("finished"),
    ;
    private final String code;

    @JsonValue
    public String getCode() {
        return code;
    }

    @JsonCreator
    public static StatusEnum of(String code) {
        for (StatusEnum statusEnum : values()) {
            if (statusEnum.getCode().equals(code)) {
                return statusEnum;
            }
        }
        return null;
    }

    public static void main(String[] args) {
        String json = JsonUtils.toJson(StatusEnum.PAUSE);
        System.out.println(json); // 输出："pause"
        StatusEnum statusEnum = JsonUtils.fromJson(json, StatusEnum.class);
        System.out.println(statusEnum); // 输出：PAUSE
    }
}
```

Example2:
```java
public class MyBean {
    private String name;

    @JsonCreator
    public MyBean(String name) {
        this.name = name;
    }

    @JsonValue
    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "MyBean{name='" + name + "'}";
    }

    public static void main(String[] args) {
        MyBean bean = new MyBean("William");
        String json = JsonUtils.toJson(bean); // 输出："William"
        System.out.println(JsonUtils.fromJson(json, MyBean.class)); // 输出：MyBean{name='William'}
    }
}
```

### 1.3 @JsonPropertyOrder

使用位置：`ANNOTATION_TYPE`、`TYPE`、`METHOD`、`CONSTRUCTOR`、`FIELD`

该注解用于显式指定属性的序列化顺序。该注解对反序列化没有影响。可使用`@JsonPropertyOrder(alphabetic=true)`方便地指定输出按字母序排序。

如下样例：

```java
// ensure that "id" and "name" are output before other properties
@JsonPropertyOrder(value = {"id", "name"})
@Getter
public class MyBean {
    private String name;
    private Long id;

    public static void main(String[] args) {
        MyBean bean = new MyBean();
        bean.id = 1023L;
        bean.name = "Google";
        String json = JsonUtils.toJson(bean);
        System.out.println(json);
    }
}
```

这是序列化的输出：

```json
{
    "id": 1023,
    "name": "Google"
}
```

### 1.4 @JsonRootName

使用位置：`ANNOTATION_TYPE`、`TYPE`

该注解用于指示根层级的包装名字。需要强调的是，只有`@JsonRootName`注解不会生效，需要开启`SerializationFeature.WRAP_ROOT_VALUE`功能。

Example1:
```java
// namespace可选，用于XML格式的命名空间指定。
@JsonRootName(value = "user", namespace = "xxx1.xxx2")
@Getter
class MyBean {
    private Long id;
    private String name;

    public static void main(String[] args) throws JsonProcessingException {
        MyBean bean = new MyBean();
        bean.id = 1023L;
        bean.name = "Google";
        ObjectMapper objectMapper = new ObjectMapper().enable(SerializationFeature.WRAP_ROOT_VALUE);
        String json = objectMapper.writeValueAsString(bean);
        System.out.println(json);
    }
}
```

Result1:
```json
{
    "user": {
        "id": 1023,
        "name": "Google"
    }
}
```

### 1.5 @JsonAnyGetter

使用位置：`ANNOTATION_TYPE`、`TYPE`

该注解用于像普通属性一样序列化为Map，注解在non-static无参的方法或者成员字段，方法返回类型必须是Map。与该注解相反的一个操作是`@JsonAnySetter`注解。

`@JsonAnyGetter` 注释允许灵活地使用 `Map` 字段作为标准属性。

例如， `ExtendableBean` 实体具有name属性和一组键/值对形式的可扩展属性：

```java
@Getter
@ToString
public class MyBean {
    private Long id;
    private String name;
    private Map<String, Object> properties = new HashMap<>();

    @JsonAnySetter
    public void setProperty(String key, Object value) {
        properties.put(key, value);
    }

    @JsonAnyGetter
    public Map<String, Object> getProperties() {
        return properties;
    }

    public static void main(String[] args) {
        MyBean bean = new MyBean();
        bean.id = 1023L;
        bean.name = "Alice";
        bean.properties.put("age", 24);
        bean.properties.put("from", "China");

        String json = JsonUtils.toJson(bean);
        System.out.println(json);
        MyBean bean2 = JsonUtils.fromJson(json, MyBean.class);
        System.out.println(bean2); // 输出：MyBean(id=1023, name=Alice, properties={from=China, age=24})
    }
}
```

当我们序列化该实体的实例时，我们将 `Map` 中的所有键值作为标准的普通属性获取：

```json
{
    "id": 1023,
    "name": "Alice",
    "from": "China",
    "age": 24
}
```

我们还可以使用可选参数启用为 `false` 来禁用 `@JsonAnyGetter`。在这种情况下，`Map` 将转换为 JSON，并在序列化后出现在属性变量下。


### 1.6 @JsonRawValue

使用位置：`ANNOTATION_TYPE`、`METHOD`、`TYPE`

该注解用于把一个json格式的字符串方法或者字段，序列化为json字符串本身的结构。

如下样例：

```java
@Getter
public class MyBean {
    private Long id;
    private String name;
    @JsonRawValue
    private String json;

    public static void main(String[] args) {
        MyBean bean = new MyBean();
        bean.id = 1023L;
        bean.name = "Alice";
        bean.json = "{\"extra\":{\"key1\": \"val1\", \"key2\":234}}";

        String json = JsonUtils.toJson(bean);
        System.out.println(json);
    }
}
```

序列化输出是：

```json
{
    "id": 1023,
    "name": "Alice",
    "json": {
        "extra": {
            "key1": "val1", 
            "key2": 234
        }
    }
}
```

### 1.7 @JsonSerialize

该注解用于自定义json序列化方式，可高度自定义。

让我们看一个简单的例子。我们将使用@JsonSerialize通过CustomDateSerializer序列化eventDate属性：

```java
public class EventWithSerializer {
    public String name;

    @JsonSerialize(using = CustomDateSerializer.class)
    public Date eventDate;
}

class CustomDateSerializer extends StdSerializer<Date> {

    private static SimpleDateFormat formatter 
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateSerializer() { 
        this(null); 
    } 

    public CustomDateSerializer(Class<Date> t) {
        super(t); 
    }

    @Override
    public void serialize(
      Date value, JsonGenerator gen, SerializerProvider arg2) 
      throws IOException, JsonProcessingException {
        gen.writeString(formatter.format(value));
    }
}
```

现在让我们在测试中使用它们：

```java
@Test
public void whenSerializingUsingJsonSerialize_thenCorrect() throws JsonProcessingException, ParseException {
 
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    EventWithSerializer event = new EventWithSerializer("party", date);

    String result = new ObjectMapper().writeValueAsString(event);
    assertThat(result, containsString(toParse));
}
```

## 2.Jackson反序列化注解

### 2.1 @JsonCreator

我们可以使用 `@JsonCreator` 注解来调整反序列化中使用的构造函数/工厂。

当我们需要反序列化一些与我们需要获取的目标实体不完全匹配的 JSON 时，它非常有用。

让我们看一个例子。假设我们需要反序列化以下 JSON：

```json
{
    "id":1,
    "theName":"My bean"
}
```

但是，我们的目标实体中没有theName字段，只有一个name字段。现在我们不想更改实体本身，我们只需要通过使用@JsonCreator注释构造函数并使用@JsonProperty注释来对解组过程进行更多控制：

```java
public class BeanWithCreator {
    public int id;
    public String name;

    @JsonCreator
    public BeanWithCreator(
      @JsonProperty("id") int id, 
      @JsonProperty("theName") String name) {
        this.id = id;
        this.name = name;
    }
}

@Test
public void whenDeserializingUsingJsonCreator_thenCorrect() throws IOException {
 
    String json = "{\"id\":1,\"theName\":\"My bean\"}";

    BeanWithCreator bean = new ObjectMapper().readerFor(BeanWithCreator.class).readValue(json);
    assertEquals("My bean", bean.name);
}
```

### 2.2 @JsonAlias

`@JsonAlias` 在反序列化期间为属性定义一个或多个替代名称。

让我们通过一个简单的例子来看看这个注释是如何工作的：

```java
public class AliasBean {
    @JsonAlias({ "fName", "f_name" })
    private String firstName;   
    private String lastName;
}
```

这里我们有一个 POJO，我们希望将包含 `fName`、`f_name `和 `firstName` 等值的 JSON 反序列化到POJO的 `firstName` 变量中。

以下是确保此注释按预期工作的测试：

```java
@Test
public void whenDeserializingUsingJsonAlias_thenCorrect() throws IOException {
    String json = "{\"fName\": \"John\", \"lastName\": \"Green\"}";
    AliasBean aliasBean = new ObjectMapper().readerFor(AliasBean.class).readValue(json);
    assertEquals("John", aliasBean.getFirstName());
}
```

### 2.3 @JsonSetter

`@JsonSetter` 是 `@JsonProperty` 的替代方案，它将方法标记为 `setter` 方法。

当我们需要读取一些 JSON 数据，但目标实体类与该数据不完全匹配，因此我们需要调整流程以使其适合时，这非常有用。

在下面的示例中，我们将指定方法 `setTheName()` 作为 `MyBean` 实体中 `name` 属性的设置器：

```java
public class MyBean {
    public int id;
    private String name;

    @JsonSetter("name")
    public void setTheName(String name) {
        this.name = name;
    }
}
```

现在，当我们需要解组一些 JSON 数据时，这非常有效：

```java
@Test
public void whenDeserializingUsingJsonSetter_thenCorrect() throws IOException {
 
    String json = "{\"id\":1,\"name\":\"My bean\"}";

    MyBean bean = new ObjectMapper().readerFor(MyBean.class).readValue(json);
    assertEquals("My bean", bean.getTheName());
}
```

### 2.4 @JsonAnySetter

`@JsonAnySetter` 允许我们灵活地使用 `Map` 作为标准属性。反序列化时，JSON 中的属性将简单地添加到映射中。

首先，我们将使用 `@JsonAnySetter` 反序列化实体 `ExtendableBean` ：

```json
{
    "name":"My bean",
    "attr2":"val2",
    "attr1":"val1"
}
```

```java
public class ExtendableBean {
    public String name;
    private Map<String, String> properties;

    @JsonAnySetter
    public void add(String key, String value) {
        properties.put(key, value);
    }
}

@Test
public void whenDeserializingUsingJsonAnySetter_thenCorrect() throws IOException {
    String json = "{\"name\":\"My bean\",\"attr2\":\"val2\",\"attr1\":\"val1\"}";

    ExtendableBean bean = new ObjectMapper().readerFor(ExtendableBean.class).readValue(json);
    
    assertEquals("My bean", bean.name);
    assertEquals("val2", bean.getProperties().get("attr2"));
}
```

### 2.5 @JsonAlias

`@JsonAlias` 在反序列化期间为属性定义一个或多个替代名称。

让我们通过一个简单的例子来看看这个注释是如何工作的：

```java
public class AliasBean {
    @JsonAlias({ "fName", "f_name" })
    private String firstName;   
    private String lastName;
}
```

这里我们有一个 POJO，我们希望将包含 `fName` 、 `f_name` 和 `firstName` 等值的 JSON 反序列化到POJO的 `firstName` 变量中。

以下是确保此注释按预期工作的测试：

```java
@Test
public void whenDeserializingUsingJsonAlias_thenCorrect() throws IOException {
    String json = "{\"fName\": \"John\", \"lastName\": \"Green\"}";
    AliasBean aliasBean = new ObjectMapper().readerFor(AliasBean.class).readValue(json);
    assertEquals("John", aliasBean.getFirstName());
}
```

### 2.6 @JacksonInject

`@JacksonInject` 指示属性将从注入而不是从 JSON 数据获取其值。

在下面的示例中，我们使用 `@JacksonInject` 来注入属性 `id` ：

```java
public class BeanWithInject {
    @JacksonInject
    public int id;
    
    public String name;
}

@Test
public void whenDeserializingUsingJsonInject_thenCorrect() throws IOException {
 
    String json = "{\"name\":\"My bean\"}";
    
    InjectableValues inject = new InjectableValues.Std().addValue(int.class, 1);
    BeanWithInject bean = new ObjectMapper().reader(inject)
      .forType(BeanWithInject.class)
      .readValue(json);
    
    assertEquals("My bean", bean.name);
    assertEquals(1, bean.id);
}
```

### 2.7 @JsonDeserialize

`@JsonDeserialize` 表示使用自定义反序列化器。

首先，我们将使用 `@JsonDeserialize` 通过 `CustomDateDeserializer` 反序列化 `eventDate` 属性：

```java
public class EventWithSerializer {
    public String name;

    @JsonDeserialize(using = CustomDateDeserializer.class)
    public Date eventDate;
}

public class CustomDateDeserializer
  extends StdDeserializer<Date> {

    private static SimpleDateFormat formatter
      = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");

    public CustomDateDeserializer() { 
        this(null); 
    } 

    public CustomDateDeserializer(Class<?> vc) { 
        super(vc); 
    }

    @Override
    public Date deserialize(
      JsonParser jsonparser, DeserializationContext context) 
      throws IOException {
        
        String date = jsonparser.getText();
        try {
            return formatter.parse(date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}

@Test
public void whenDeserializingUsingJsonDeserialize_thenCorrect() throws IOException {
 
    String json = "{"name":"party","eventDate":"20-12-2014 02:30:00"}";

    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    EventWithSerializer event = new ObjectMapper().readerFor(EventWithSerializer.class).readValue(json);
    
    assertEquals("20-12-2014 02:30:00", df.format(event.eventDate));
}
```

## 3.Jackson属性注解

### 3.1 @JsonIgnore

`@JsonIgnore` 注解用于标记要在字段级别忽略的属性。

让我们使用 `@JsonIgnore` 来忽略序列化中的属性 `id` ：

```java
public class BeanWithIgnore {
    @JsonIgnore
    public int id;

    public String name;
}

@Test
public void whenSerializingUsingJsonIgnore_thenCorrect() throws JsonProcessingException {
    BeanWithIgnore bean = new BeanWithIgnore(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
    
    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

### 3.2 @JsonIgnoreProperties

`@JsonIgnoreProperties` 是一个类级注释，标记 Jackson 将忽略的属性或属性列表。

让我们看一个忽略序列化属性 `id` 的简单示例：

```java
@JsonIgnoreProperties({ "id" })
public class BeanWithIgnore {
    public int id;
    public String name;
}

@Test
public void whenSerializingUsingJsonIgnoreProperties_thenCorrect() throws JsonProcessingException {
    BeanWithIgnore bean = new BeanWithIgnore(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
    
    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

要无一例外地忽略 JSON 输入中的任何未知属性，我们可以设置 `@JsonIgnoreProperties` 注释的 `ignoreUnknown=true` 。

### 3.3 @JsonIgnoreType

`@JsonIgnoreType` 标记要忽略的带注释类型的所有属性。

我们可以使用注释来标记所有 `Name` 类型的属性被忽略：

```java
public class User {
    public int id;
    public Name name;

    @JsonIgnoreType
    public static class Name {
        public String firstName;
        public String lastName;
    }
}

@Test
public void whenSerializingUsingJsonIgnoreType_thenCorrect() throws JsonProcessingException, ParseException {
    User.Name name = new User.Name("John", "Doe");
    User user = new User(1, name);

    String result = new ObjectMapper().writeValueAsString(user);

    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("name")));
    assertThat(result, not(containsString("John")));
}
```

我们也可以定义一个 MixIn 的方式来对某个类型进行忽略。

```java
@JsonIgnoreType
public class MyMixInForIgnoreType {}

mapper.addMixInAnnotations(String[].class, MyMixInForIgnoreType.class);
```

如下是使用示例：

```java
public class MyDtoWithSpecialField {
    private String[] stringValue;
    private int intValue;
    private boolean booleanValue;
}

@Test
public final void givenFieldTypeIsIgnored_whenDtoIsSerialized_thenCorrect() throws JsonParseException, IOException {
    ObjectMapper mapper = new ObjectMapper();
    mapper.addMixIn(String[].class, MyMixInForIgnoreType.class);
    MyDtoWithSpecialField dtoObject = new MyDtoWithSpecialField();
    dtoObject.setBooleanValue(true);

    String dtoAsString = mapper.writeValueAsString(dtoObject);

    assertThat(dtoAsString, containsString("intValue"));
    assertThat(dtoAsString, containsString("booleanValue"));
    assertThat(dtoAsString, not(containsString("stringValue")));
}
```

### 3.4 @JsonInclude

我们可以使用 `@JsonInclude` 来排除具有空/空/默认值的属性。

让我们看一个从序列化中排除空值的示例：

```java
@JsonInclude(Include.NON_NULL)
public class MyBean {
    public int id;
    public String name;
    // @JsonInclude(Include.NON_NULL)
    public String description;
}

public void whenSerializingUsingJsonInclude_thenCorrect() throws JsonProcessingException {
    MyBean bean = new MyBean(1, null);

    String result = new ObjectMapper().writeValueAsString(bean);
    
    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("name")));
}
```

你也可以用过 `objectMapper.setSerializationInclusion(Include.NON_NULL);` 来全局地控制不序列化空字段。

### 3.5 @JsonIncludeProperties

`@JsonIncludeProperties` 是最受欢迎的 Jackson 功能之一。它是在 Jackson 2.12 中引入的，可用于标记 Jackson 将在序列化和反序列化期间包含的属性或属性列表。

让我们看一个简单的示例，其中包括序列化中的属性名称：

```java
@JsonIncludeProperties({ "name" })
public class BeanWithInclude {
    public int id;
    public String name;
}

@Test
public void whenSerializingUsingJsonIncludeProperties_thenCorrect() throws JsonProcessingException {
    final BeanWithInclude bean = new BeanWithInclude(1, "My bean");
    final String result = new ObjectMapper().writeValueAsString(bean);
    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
    assertThat(result, containsString("name"));
}
```

### 3.6 @JsonAutoDetect

`@JsonAutoDetect` 可以覆盖哪些属性可见、哪些属性不可见的默认语义。

首先，让我们通过一个简单的示例来了解注释如何发挥很大作用；让我们启用私有属性的序列化：

```java
@JsonAutoDetect(fieldVisibility = Visibility.ANY)
public class PrivateBean {
    private int id;
    private String name;
}

@Test
public void whenSerializingUsingJsonAutoDetect_thenCorrect() throws JsonProcessingException {
    PrivateBean bean = new PrivateBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
    
    assertThat(result, containsString("1"));
    assertThat(result, containsString("My bean"));
}
```

## 4. Jackson多态类型处理注解

接下来我们看一下Jackson多态类型处理注解：

* `@JsonTypeInfo` – 指示序列化中包含的类型信息的详细信息
* `@JsonSubTypes` – 表示注释类型的子类型
* `@JsonTypeName` – 定义用于带注释的类的逻辑类型名称

让我们检查一个更复杂的示例，并使用所有三个 `@JsonTypeInfo` 、 `@JsonSubTypes` 和 `@JsonTypeName` 来序列化/反序列化实体 `Zoo` ：

```java
public class Zoo {
    public Animal animal;

    @JsonTypeInfo(
      use = JsonTypeInfo.Id.NAME, 
      include = As.PROPERTY, 
      property = "type")
    @JsonSubTypes({
        @JsonSubTypes.Type(value = Dog.class, name = "dog"),
        @JsonSubTypes.Type(value = Cat.class, name = "cat")
    })
    public static class Animal {
        public String name;
    }

    @JsonTypeName("dog")
    public static class Dog extends Animal {
        public double barkVolume;
    }

    @JsonTypeName("cat")
    public static class Cat extends Animal {
        boolean likesCream;
        public int lives;
    }
}
```

当我们进行序列化时：

```java
@Test
public void whenSerializingPolymorphic_thenCorrect() throws JsonProcessingException {
    Zoo.Dog dog = new Zoo.Dog("lacy");
    Zoo zoo = new Zoo(dog);

    String result = new ObjectMapper().writeValueAsString(zoo);

    assertThat(result, containsString("type"));
    assertThat(result, containsString("dog"));
}
```

以下是使用Dog序列化Zoo实例将导致的结果：

```json
{
    "animal": {
        "type": "dog",
        "name": "lacy",
        "barkVolume": 0
    }
}
```

现在进行反序列化。让我们从以下 JSON 输入开始：

```json
{
    "animal":{
        "name":"lacy",
        "type":"cat"
    }
}
```

然后让我们看看如何将其解组到Zoo实例：

```java
@Test
public void whenDeserializingPolymorphic_thenCorrect() throws IOException {
    String json = "{\"animal\":{\"name\":\"lacy\",\"type\":\"cat\"}}";

    Zoo zoo = new ObjectMapper().readerFor(Zoo.class).readValue(json);

    assertEquals("lacy", zoo.animal.name);
    assertEquals(Zoo.Cat.class, zoo.animal.getClass());
}
```

## 5.Jackson一般注解

### 5.1 @JsonProperty

我们可以添加 `@JsonProperty` 注解来指示 JSON 中的属性名称。

当我们处理非标准的 `getter` 和 `setter` 时，让我们使用 `@JsonProperty` 来序列化/反序列化属性名称：

```java
public class MyBean {
    public int id;
    private String name;

    @JsonProperty("name")
    public void setTheName(String name) {
        this.name = name;
    }

    @JsonProperty("name")
    public String getTheName() {
        return name;
    }
}

@Test
public void whenUsingJsonProperty_thenCorrect() throws IOException {
    MyBean bean = new MyBean(1, "My bean");

    String result = new ObjectMapper().writeValueAsString(bean);
    
    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));

    MyBean resultBean = new ObjectMapper()
      .readerFor(MyBean.class)
      .readValue(result);
    assertEquals("My bean", resultBean.getTheName());
}
```

### 5.2 @JsonFormat

`@JsonFormat` 注解可以指定序列化日期/时间值时的格式。

在下面的示例中，我们使用 `@JsonFormat` 来控制属性 `eventDate` 的格式：

```java
public class EventWithFormat {
    public String name;

    @JsonFormat(
      shape = JsonFormat.Shape.STRING,
      pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;
}

@Test
public void whenSerializingUsingJsonFormat_thenCorrect() throws JsonProcessingException, ParseException {
    SimpleDateFormat df = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    df.setTimeZone(TimeZone.getTimeZone("UTC"));

    String toParse = "20-12-2014 02:30:00";
    Date date = df.parse(toParse);
    EventWithFormat event = new EventWithFormat("party", date);
    
    String result = new ObjectMapper().writeValueAsString(event);
    
    assertThat(result, containsString(toParse));
}
```

### 5.3 @JsonUnwrapped

`@JsonUnwrapped` 定义序列化/反序列化时应展开/展平的值。

让我们看看它是如何工作的；我们将使用注释来解开属性名称：

```java
public class UnwrappedUser {
    public int id;

    @JsonUnwrapped
    public Name name;

    public static class Name {
        public String firstName;
        public String lastName;
    }
}
@Test
public void whenSerializingUsingJsonUnwrapped_thenCorrect() throws JsonProcessingException, ParseException {
    UnwrappedUser.Name name = new UnwrappedUser.Name("John", "Doe");
    UnwrappedUser user = new UnwrappedUser(1, name);

    String result = new ObjectMapper().writeValueAsString(user);
    
    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("name")));
}
```

最后，输出如下所示 - 静态嵌套类的字段与其他字段一起展开：

```json
{
    "id":1,
    "firstName":"John",
    "lastName":"Doe"
}
```

### 5.4 @JsonView

`@JsonView` 指示将包含该属性以进行序列化/反序列化的视图。

例如，我们将使用 `@JsonView` 来序列化 `Item` 实体的实例。

```java
public class Views {
    public static class Public {}
    public static class Internal extends Public {}
}
public class Item {
    @JsonView(Views.Public.class)
    public int id;

    @JsonView(Views.Public.class)
    public String itemName;

    @JsonView(Views.Internal.class)
    public String ownerName;
}

@Test
public void whenSerializingUsingJsonView_thenCorrect() throws JsonProcessingException {
    Item item = new Item(2, "book", "John");

    String result = new ObjectMapper().writerWithView(Views.Public.class).writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("2"));
    assertThat(result, not(containsString("John")));
}
```

### 5.5 @JsonManagedReference和@JsonBackReference

`@JsonManagedReference` 和 `@JsonBackReference` 注释可以处理父/子关系并解决循环问题。

在下面的示例中，我们使用 `@JsonManagedReference` 和 `@JsonBackReference` 来序列化我们的 `ItemWithRef` 实体：

```java
public class ItemWithRef {
    public int id;
    public String itemName;

    @JsonManagedReference
    public UserWithRef owner;
}
public class UserWithRef {
    public int id;
    public String name;

    @JsonBackReference
    public List<ItemWithRef> userItems;
}
@Test
public void whenSerializingUsingJacksonReferenceAnnotation_thenCorrect() throws JsonProcessingException {
    UserWithRef user = new UserWithRef(1, "John");
    ItemWithRef item = new ItemWithRef(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, not(containsString("userItems")));
}
```

### 5.6 @JsonIdentityInfo

`@JsonIdentityInfo` 指示在序列化/反序列化值时应使用对象标识，例如在处理无限递归类型的问题时。

在以下示例中，我们有一个与 `UserWithIdentity` 实体具有双向关系的 `ItemWithIdentity` 实体：

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class,
  property = "id")
public class ItemWithIdentity {
    public int id;
    public String itemName;
    public UserWithIdentity owner;
}
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class,
  property = "id")
public class UserWithIdentity {
    public int id;
    public String name;
    public List<ItemWithIdentity> userItems;
}

@Test
public void whenSerializingUsingJsonIdentityInfo_thenCorrect() throws JsonProcessingException {
    UserWithIdentity user = new UserWithIdentity(1, "John");
    ItemWithIdentity item = new ItemWithIdentity(2, "book", user);
    user.addItem(item);

    String result = new ObjectMapper().writeValueAsString(item);

    assertThat(result, containsString("book"));
    assertThat(result, containsString("John"));
    assertThat(result, containsString("userItems"));
}
```

以下是序列化项目和用户的完整输出：

```json
{
    "id": 2,
    "itemName": "book",
    "owner": {
        "id": 1,
        "name": "John",
        "userItems": [
            2
        ]
    }
}
```

### 5.7 @JsonFilter

`@JsonFilter` 注解指定序列化期间要使用的过滤器。

首先，我们定义实体并指向过滤器：

```java
@JsonFilter("myFilter")
public class BeanWithFilter {
    public int id;
    public String name;
}
```

现在，在完整测试中，我们定义过滤器，该过滤器从序列化中排除除name之外的所有其他属性：

```java
@Test
public void whenSerializingUsingJsonFilter_thenCorrect() throws JsonProcessingException {
    BeanWithFilter bean = new BeanWithFilter(1, "My bean");

    FilterProvider filters = new SimpleFilterProvider().addFilter("myFilter", 
        SimpleBeanPropertyFilter.filterOutAllExcept("name"));

    String result = new ObjectMapper().writer(filters).writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, not(containsString("id")));
}
```

## 6.自定义Jackson注解

接下来让我们看看如何创建自定义 Jackson 注释。我们可以使用 `@JacksonAnnotationsInside` 注解：

```java
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonInclude(Include.NON_NULL)
@JsonPropertyOrder({ "name", "id", "dateCreated" })
public @interface CustomAnnotation {}
```

现在，如果我们在实体上使用新注释：

```java
@CustomAnnotation
public class BeanWithCustomAnnotation {
    public int id;
    public String name;
    public Date dateCreated;
}
```

我们可以看到它如何将现有注释组合成一个简单的自定义注释，我们可以将其用作简写：

```java
@Test
public void whenSerializingUsingCustomAnnotation_thenCorrect() throws JsonProcessingException {
    BeanWithCustomAnnotation bean = new BeanWithCustomAnnotation(1, "My bean", null);

    String result = new ObjectMapper().writeValueAsString(bean);

    assertThat(result, containsString("My bean"));
    assertThat(result, containsString("1"));
    assertThat(result, not(containsString("dateCreated")));
}
```

序列化过程的输出：

```json
{
    "name":"My bean",
    "id":1
}
```

## 7.Jackson MixIn 注释

接下来我们看看如何使用 Jackson MixIn 注释。

例如，让我们使用 MixIn 注释来忽略User类型的属性：

```java
public class Item {
    public int id;
    public String itemName;
    public User owner;
}
@JsonIgnoreType
public class MyMixInForIgnoreType {}
```

然后让我们看看实际效果：

```java
@Test
public void whenSerializingUsingMixInAnnotation_thenCorrect() throws JsonProcessingException {
    Item item = new Item(1, "book", null);

    String result = new ObjectMapper().writeValueAsString(item);
    assertThat(result, containsString("owner"));

    ObjectMapper mapper = new ObjectMapper();
    mapper.addMixIn(User.class, MyMixInForIgnoreType.class);

    result = mapper.writeValueAsString(item);
    assertThat(result, not(containsString("owner")));
}
```

## 8.禁用Jackson注释

最后，让我们看看如何禁用所有 Jackson 注释。我们可以通过禁用MapperFeature 来做到这一点。USE_ANNOTATIONS 如下例所示：

```java
@JsonInclude(Include.NON_NULL)
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```

现在，禁用注释后，这些应该不起作用，并且应该应用库的默认值：

```java
@Test
public void whenDisablingAllAnnotations_thenAllDisabled() throws IOException {
    MyBean bean = new MyBean(1, null);

    ObjectMapper mapper = new ObjectMapper();
    mapper.disable(MapperFeature.USE_ANNOTATIONS);
    String result = mapper.writeValueAsString(bean);
    
    assertThat(result, containsString("1"));
    assertThat(result, containsString("name"));
}
```

禁用注解之前序列化的结果：

```json
{"id":1}
```

禁用注解后序列化的结果：

```json
{
    "id":1,
    "name":null
}
```

# 参考资料

* [Jackson Annotation Examples](https://www.baeldung.com/jackson-annotations#bd-jackson-serialization-annotations)