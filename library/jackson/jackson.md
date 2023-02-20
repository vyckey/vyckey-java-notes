#

## Jackson注解使用大全

### @JsonGetter

使用位置：`ANNOTATION_TYPE`、`METHOD`

对于non-static、无参和non-void返回的方法，可以使用该注解来作为一个逻辑getter方法。更加通用的注解`@JsonProperty`也同样可以实现相同的功能。

Example1:
```java
public class MyBean {
    private int id;
    private String name;

    @JsonGetter("name")
    public String getTheName() {
        return name;
    }

    @JsonGetter("name2")
    private String internalName() {
        return id + ":" + name;
    }

    public static void main(String[] args) {
        MyBean bean = new MyBean();
        bean.id = 1024;
        bean.name = "Jack";
        String json = JsonUtils.toJson(bean);
        System.out.println(json);
    }
}
```

Result1:
```json
{
    "name": "Jack",
    "name2":"1024:Jack"
}
```

### @JsonValue

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

### @JsonPropertyOrder

使用位置：`ANNOTATION_TYPE`、`TYPE`、`METHOD`、`CONSTRUCTOR`、`FIELD`

该注解用于显式指定属性的序列化顺序。该注解对反序列化没有影响。可使用`@JsonPropertyOrder(alphabetic=true)`方便地指定输出按字母序排序。

Example1:
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

Result1:
```json
{
    "id": 1023,
    "name": "Google"
}
```

### @JsonRootName

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

### @JsonAnyGetter

使用位置：`ANNOTATION_TYPE`、`TYPE`

该注解用于像普通属性一样序列化为Map，注解在non-static无参的方法或者成员字段，方法返回类型必须是Map。与该注解相反的一个操作是`@JsonAnySetter`注解。

Example1:
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

Result1:
```json
{
    "id": 1023,
    "name": "Alice",
    "from": "China",
    "age": 24
}
```

### @JsonRawValue

使用位置：`ANNOTATION_TYPE`、`METHOD`、`TYPE`

该注解用于把一个json格式的字符串方法或者字段，序列化为json字符串本身的结构。

Example1:
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

Result1:
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

### @JsonSerialize