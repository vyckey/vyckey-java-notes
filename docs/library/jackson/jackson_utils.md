---
title: Jackson Utils
tags: [java, jackson, json, xml]
sidebar_label: JSON/XML Utils
sidebar_position: 2
---

# JSON 工具类

## JsonObjectMapper

```groovy
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.3</version>
</dependency>
```

```java
package xxx.utils;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.PropertyNamingStrategy;
import com.fasterxml.jackson.databind.type.TypeFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.lang.reflect.Type;

/**
 * {@link ObjectMapper} for JSON
 *
 * <p>example 1:</p>
 * <pre>{@code
 * @Data
 * class Person {
 *     private Integer myAge;
 *     private String myName;
 * }
 *
 * Person person = JsonObjectMapper.CAMEL_CASE.fromJson("{\"myAge\": 25, \"myName\":\"Bob\"}", Person.class);
 * String json = JsonObjectMapper.SNAKE_CASE.toJson(person);
 * Assert.assertEquals("{\"my_age\":25,\"my_name\":\"Bob\"}", json);
 * }</pre>
 *
 * <p>example 2:</p>
 * <pre>{@code
 * class MyJsonUtils {
 *     public static final JsonObjectMapper INSTANCE = new JsonObjectMapper(new ObjectMapper()
 *             // custom features for yourself
 *             .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
 *     );
 * }
 * }</pre>
 *
 * @author vyckey
 */
public class JsonObjectMapper {
    private static final Logger log = LoggerFactory.getLogger(JsonObjectMapper.class);
    public static final JsonObjectMapper CAMEL_CASE = new JsonObjectMapper(new ObjectMapper()
            .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
            .enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT)
    );
    public static final JsonObjectMapper SNAKE_CASE = new JsonObjectMapper(new ObjectMapper()
            .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
            .enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT)
            .setPropertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE)
    );

    private final ObjectMapper objectMapper;

    public JsonObjectMapper(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }

    protected ObjectMapper getObjectMapper() {
        return objectMapper;
    }

    protected void handleException(IOException e) {
        log.error(e.getMessage(), e);
    }

    public String toJson(Object object) {
        try {
            return objectMapper.writeValueAsString(object);
        } catch (IOException e) {
            handleException(e);
        }
        return null;
    }

    public <T> T fromJson(String json, Class<T> valueType) {
        try {
            return objectMapper.readValue(json, valueType);
        } catch (IOException e) {
            handleException(e);
        }
        return null;
    }

    public <T> T fromJson(String json, TypeReference<T> valueTypeRef) {
        try {
            return objectMapper.readValue(json, valueTypeRef);
        } catch (IOException e) {
            handleException(e);
        }
        return null;
    }

    public <T> T fromJson(String json, JavaType valueType) {
        try {
            return objectMapper.readValue(json, valueType);
        } catch (IOException e) {
            handleException(e);
        }
        return null;
    }

    public <T> T fromJson(String json, Type type) {
        try {
            JavaType javaType = TypeFactory.defaultInstance().constructType(type);
            return objectMapper.readValue(json, javaType);
        } catch (IOException e) {
            handleException(e);
        }
        return null;
    }

    public <T> T fromJson(byte[] json, Class<T> clazz) {
        if (json == null) {
            return null;
        }
        try {
            return objectMapper.readValue(json, clazz);
        } catch (IOException e) {
            handleException(e);
        }
        return null;
    }

    public <T> T fromJson(byte[] json, TypeReference<T> type) {
        if (json == null) {
            return null;
        }
        try {
            return objectMapper.readValue(json, type);
        } catch (IOException e) {
            handleException(e);
        }
        return null;
    }

    public <T> T convert(Object object, Class<T> clazz) {
        return objectMapper.convertValue(object, clazz);
    }

    public <T> T convert(Object object, TypeReference<T> type) {
        return objectMapper.convertValue(object, type);
    }
}
```

# XML 工具类

## XmlUtils

```groovy
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.11.1</version>
</dependency>
```

```java
package xxx.utils;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.dataformat.xml.XmlMapper;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;

/**
 * description is here
 *
 * @author vyckey
 */
@Slf4j
public class XmlUtils {
    private static final XmlMapper XML_MAPPER = new XmlMapper();

    private XmlUtils() {
    }

    public static String toXml(Object object) {
        try {
            return XML_MAPPER.writeValueAsString(object);
        } catch (IOException e) {
            log.error(e.getMessage(), e);
        }
        return null;
    }

    public static <T> T fromXml(String xml, Class<T> valueType) {
        try {
            return XML_MAPPER.readValue(xml, valueType);
        } catch (IOException e) {
            log.error(e.getMessage(), e);
        }
        return null;
    }

    public static <T> T fromXml(String xml, TypeReference<T> valueTypeRef) {
        try {
            return XML_MAPPER.readValue(xml, valueTypeRef);
        } catch (IOException e) {
            log.error(e.getMessage(), e);
        }
        return null;
    }
}
```