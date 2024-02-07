---
title: Jackson Advanced Features
tags: [java, jackson]
sidebar_label: Advanced Features
sidebar_position: 4
---

# Jackson Advaned Features

## 序列化和反序列化Feature



## Map 的序列化和反序列化

### `Map<Object, Object>` 序列化

#### 方法1：使用 `@JsonValue`

让我们创建一个 `MyPair` 类来表示一对相关的 `String` 对象。我们用 `@JsonValue` 注释 `toString()` 以确保Jackson在序列化时使用这个自定义的 `toString()` ：

```java
public class MyPair {
    private String first;
    private String second;
    
    @Override
    @JsonValue
    public String toString() {
        return first + " and " + second;
    }
 
    // standard getter, setters, equals, hashCode, constructors
}
```

#### 方法2：自定义 `JsonSerializer` 

```java
public class MyPairSerializer extends JsonSerializer<MyPair> {
    private ObjectMapper mapper = new ObjectMapper();

    @Override
    public void serialize(MyPair value, JsonGenerator gen, SerializerProvider serializers) throws IOException, JsonProcessingException {
        StringWriter writer = new StringWriter();
        mapper.writeValue(writer, value);
        gen.writeFieldName(writer.toString());
    }
}
```

接下来，我们使用 `@JsonSerialize` 注释将 `MyPairSerializer` 应用于 `Map<MyPair, String>` 。请注意，我们只告诉 Jackson 如何序列化 `MyPair`，因为它已经知道如何序列化 `String` ：

```java
@JsonSerialize(keyUsing = MyPairSerializer.class) 
Map<MyPair, String> map;
```

然后让我们测试我们的 `Map` 序列化：

```java
Map<MyPair, String> map = new HashMap<>();
MyPair key = new MyPair("Abbott", "Costello");
map.put(key, "Comedy");

String json = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(map);
```

序列化的 JSON 输出为：

```json
{
  "Abbott and Costello" : "Comedy"
}
```

#### 方法3：使用 `@JsonKey` 注解

创建 `Map` 时，对象可以是键或值。此外，当对象作为 `Map` 中的键出现时与作为值出现时，我们可能需要不同的序列化策略。那么，让我们了解如何使用 `@JsonKey` 注释来做到这一点。

让我们首先定义包含两个成员的 `Fruit` 类，即 `variety` 和 `name` 。每当 `Fruit` 对象作为 `Map` 中的键出现时，我们都希望使用它的名称进行序列化。但是，当它作为值出现时，我们希望将其名称与品种一起使用。

```java
public class Fruit {
    public String variety;

    @JsonKey
    public String name;

    public Fruit(String variety, String name) {
        this.variety = variety;
        this.name = name;
    }

    @JsonValue
    public String getFullName() {
        return this.variety + " " + this.name;
    }
}
```

现在，我们初始化 `Fruit` 类的两个实例和 `ObjectMapper` 类的单个实例以进行序列化，然后验证对象的序列化是否使用 `fullName` 。

```java
private static final Fruit FRUIT1 = new Fruit("Alphonso", "Mango");
private static final Fruit FRUIT2 = new Fruit("Black", "Grapes");
private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

@Test
public void givenObject_WhenSerialize_ThenUseJsonValueForSerialization() throws JsonProcessingException {
    String serializedValueForFruit1 = OBJECT_MAPPER.writeValueAsString(FRUIT1);
    Assertions.assertEquals("\"Alphonso Mango\"", serializedValueForFruit1);
    String serializedValueForFruit2 = OBJECT_MAPPER.writeValueAsString(FRUIT2);
    Assertions.assertEquals("\"Black Grapes\"", serializedValueForFruit2);
}
```

接下来，我们构造一个以 `Fruit` 类实例作为键的 `selectionByFruit` ，然后验证序列化的键值是否使用 `@JsonKey` 注解：

```java
@Test
public void givenMapWithObjectKeys_WhenSerialize_ThenUseJsonKeyForSerialization() throws JsonProcessingException {
    // Given
    Map<Fruit, String> selectionByFruit = new LinkedHashMap<>();
    selectionByFruit.put(FRUIT1, "Hagrid");
    selectionByFruit.put(FRUIT2, "Hercules");
    // When
    String serializedValue = OBJECT_MAPPER.writeValueAsString(selectionByFruit);
    // Then
    Assertions.assertEquals("{\"Mango\":\"Hagrid\",\"Grapes\":\"Hercules\"}", serializedValue);
}
```
以上，我们已经成功地根据对象在 `Map` 中作为键或值的角色来切换对象的序列化策略。

### `Map<Object, Object>` 反序列化

我们可以使用 Jackson 的 KeyDeserializer 类来实现复杂类型作为键的 Map ，如下：

```java
public class ClassWithAMap {
    @JsonDeserialize(keyUsing = MyPairDeserializer.class)
    private Map<MyPair, String> map;

    @JsonCreator
    public ClassWithAMap(@JsonProperty("map") Map<MyPair, String> map) {
        this.map = map;
    }
 
    // public getters/setters omitted
}
public class MyPairDeserializer extends KeyDeserializer {
    @Override
    public MyPair deserializeKey(String key, DeserializationContext ctxt) throws IOException, JsonProcessingException {
      return new MyPair(key);
    }
}
```

然后我们可以使用readValue测试反序列化：

```java
String jsonInput = "{\"Abbott and Costello\":\"Comedy\"}";
ClassWithAMap classWithMap = mapper.readValue(jsonInput, ClassWithAMap.class);
```

## 按条件对字段进行序列化

### @JsonFilter

我们先定义一个带注解 `@JsonFilter` 的实体，然后自定义一个 `PropertyFilter` 。

```java
@JsonFilter("myFilter")
public class MyDto {
    private int intValue;

    public MyDto() {
        super();
    }

    public int getIntValue() {
        return intValue;
    }

    public void setIntValue(int intValue) {
        this.intValue = intValue;
    }
}
```

```java
PropertyFilter theFilter = new SimpleBeanPropertyFilter() {
   @Override
   public void serializeAsField(Object pojo, JsonGenerator jgen, SerializerProvider provider, PropertyWriter writer) throws Exception {
        if (include(writer)) {
         (!writer.getName().equals("intValue")) {
            writer.serializeAsField(pojo, jgen, provider);
            return;
        }
        int intValue = ((MyDtoWithFilter) pojo).getIntValue();
        if (intValue >= 0) {
            writer.serializeAsField(pojo, jgen, provider);
        }
        } else if (!jgen.canOmitFields()) { // since 2.3
            writer.serializeAsOmittedField(pojo, jgen, provider);
        }
    }
   
    @Override
    protected boolean include(BeanPropertyWriter writer) {
        return true;
    }
   
    @Override
    protected boolean include(PropertyWriter writer) {
        return true;
    }
};
```
```java
FilterProvider filters = new SimpleFilterProvider().addFilter("myFilter", theFilter);
MyDto dtoObject = new MyDto();
dtoObject.setIntValue(-1);

ObjectMapper mapper = new ObjectMapper();
String dtoAsString = mapper.writer(filters).writeValueAsString(dtoObject);
assertThat(dtoAsString, not(containsString("intValue")));
```

### 自定义JsonSerializer

假设我们有如下实体以及接口定义，`Hidable#isHidden` 代表是否隐藏信息：

```java
@JsonIgnoreProperties("hidden")
public interface Hidable {
    boolean isHidden();
}

public class Person implements Hidable {
    private String name;
    private Address address;
    private boolean hidden;
}

public class Address implements Hidable {
    private String city;
    private String country;
    private boolean hidden;
}
```

首先自定义一个 `HidableSerializer` ：

```java
public class HidableSerializer extends JsonSerializer<Hidable> {
    private JsonSerializer<Object> defaultSerializer;

    public HidableSerializer(JsonSerializer<Object> serializer) {
        defaultSerializer = serializer;
    }

    @Override
    public void serialize(Hidable value, JsonGenerator jgen, SerializerProvider provider)
      throws IOException, JsonProcessingException {
        if (value.isHidden())
            return;
        defaultSerializer.serialize(value, jgen, provider);
    }

    @Override
    public boolean isEmpty(SerializerProvider provider, Hidable value) {
        return (value == null || value.isHidden());
    }
}
```

然后使用自定义一个 `BeanSerializerModifier` ：

```java
ObjectMapper mapper = new ObjectMapper();
mapper.setSerializationInclusion(Include.NON_EMPTY);
mapper.registerModule(new SimpleModule() {
    @Override
    public void setupModule(SetupContext context) {
        super.setupModule(context);
        context.addBeanSerializerModifier(new BeanSerializerModifier() {
            @Override
            public JsonSerializer<?> modifySerializer(
                SerializationConfig config, BeanDescription desc, JsonSerializer<?> serializer) {
                if (Hidable.class.isAssignableFrom(desc.getBeanClass())) {
                    return new HidableSerializer((JsonSerializer<Object>) serializer);
                }
                return serializer;
            }
        });
    }
});
```

接下来我们构造对象进行序列化输出：

```java
Address ad1 = new Address("tokyo", "jp", true);
Address ad2 = new Address("london", "uk", false);
Address ad3 = new Address("ny", "usa", false);
Person p1 = new Person("john", ad1, false);
Person p2 = new Person("tom", ad2, true);
Person p3 = new Person("adam", ad3, false);

System.out.println(mapper.writeValueAsString(Arrays.asList(p1, p2, p3)));
```

输出JSON为：

```json
[
    {
        "name":"john"
    },
    {
        "name":"adam",
        "address":{
            "city":"ny",
            "country":"usa"
        }
    }
]
```

测试程序如下：

```java
@Test
public void whenNotHidden_thenCorrect() throws JsonProcessingException {
    Address ad = new Address("ny", "usa", false);
    Person person = new Person("john", ad, false);
    String result = mapper.writeValueAsString(person);

    assertTrue(result.contains("name"));
    assertTrue(result.contains("john"));
    assertTrue(result.contains("address"));
    assertTrue(result.contains("usa"));
}
@Test
public void whenAddressHidden_thenCorrect() throws JsonProcessingException {
    Address ad = new Address("ny", "usa", true);
    Person person = new Person("john", ad, false);
    String result = mapper.writeValueAsString(person);

    assertTrue(result.contains("name"));
    assertTrue(result.contains("john"));
    assertFalse(result.contains("address"));
    assertFalse(result.contains("usa"));
}
```

# 参考资料

* [GitHub - FasterXML/jackson-doc](https://github.com/FasterXML/jackson-docs)
* [Baeldung - jackson](https://www.baeldung.com/jackson)