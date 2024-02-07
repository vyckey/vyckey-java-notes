---
title: Jackson Polymorphic
tags: [java, json, jackson]
sidebar_label: Polymorphic
sidebar_position: 5
---

自定义属性多态反序列化

```java
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.ObjectCodec;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.deser.std.StdDeserializer;
import com.fasterxml.jackson.databind.node.ObjectNode;
import com.google.common.collect.Maps;

import java.io.IOException;
import java.util.Map;
import java.util.Optional;

public class PropertyPolymorphicDeserializer<T> extends StdDeserializer<T> {
    /**
     * the registry of unique type id to class types
     */
    private final Map<String, Class<? extends T>> registry = Maps.newHashMap();
    private final String property;

    public PropertyPolymorphicDeserializer(Class<T> superClass, String property) {
        super(superClass);
        this.property = property;
    }

    public void register(String typeId, Class<? extends T> typeClass) {
        registry.put(typeId, typeClass);
    }

    protected String resolveTypeId(JsonNode jsonNode) {
        return Optional.ofNullable(property).map(jsonNode::get).map(JsonNode::textValue).orElse(null);
    }

    @Override
    public T deserialize(JsonParser jsonParser, DeserializationContext context) throws IOException {
        ObjectCodec codec = jsonParser.getCodec();
        ObjectNode jsonNode = codec.readTree(jsonParser);

        String typeId = resolveTypeId(jsonNode);
        if (typeId == null) {
            context.reportBadDefinition(handledType(), "No type id for class \"" + handledType() + "\" deserialization");
        }
        Class<? extends T> subtypeClass = registry.get(typeId);
        if (subtypeClass == null) {
            context.reportBadDefinition(handledType(), "No corresponding \"" + handledType() + "\" subtype class found for type id" + typeId);
        }

        return codec.treeToValue(jsonNode, subtypeClass);
    }
}
```

嵌套多层级多态反序列化

```java
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.ObjectCodec;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.deser.std.StdDeserializer;

import java.io.IOException;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class NestedPolymorphicDeserializer extends StdDeserializer<Object> {
    public NestedPolymorphicDeserializer(Class<?> vc) {
        super(vc);
    }

    public NestedPolymorphicDeserializer(JavaType valueType) {
        super(valueType);
    }

    /**
     * navigates through the given class hierarchy to find the concrete implementation based
     * in the provided discriminator properties defined in the class
     */
    protected Class<?> resolveConcreteType(Class<?> last, Class<?> current, Function<String, String> fieldValueExtractor) {
        JsonTypeInfo currentTypeInfo = current.getAnnotation(JsonTypeInfo.class);
        if (currentTypeInfo == null) {
            return current;
        }

        if (current.equals(last)) {
            return current;
        }

        JsonSubTypes subTypes = current.getAnnotation(JsonSubTypes.class);
        Map<String, Class<?>> subTypesByDiscriminatorValue = Stream.of(subTypes.value())
                .collect(Collectors.toMap(JsonSubTypes.Type::name, JsonSubTypes.Type::value));

        // gets the JSON property to use as the discriminator
        String subTypeDiscriminatorProperty = currentTypeInfo.property();

        // gets the discriminator value using the JSON property obtained above
        String discriminatorValue = fieldValueExtractor.apply(subTypeDiscriminatorProperty);

        // finally, through the map created above, it gets the class mapped to the discriminator value.
        Class<?> subType = subTypesByDiscriminatorValue.get(discriminatorValue);

        // recursively calls the method with the subType found and the JSON field value extractor.
        return this.resolveConcreteType(current, subType, fieldValueExtractor);
    }

    @Override
    public Object deserialize(final JsonParser jsonParser, final DeserializationContext context) throws IOException {
        // gets the json node being deserialized
        ObjectCodec oc = jsonParser.getCodec();
        JsonNode node = oc.readTree(jsonParser);

        // gets the type of the node being serialized
        Class<?> deserializingType = handledType() != null ? handledType() : getValueType().getRawClass();

        // resolve the concrete type based in the node being serialized and the super type of the field
        Class<?> concreteType = this.resolveConcreteType(null, deserializingType, fieldName -> {
            JsonNode jsonNode = node.get(fieldName);
            return jsonNode.asText();
        });

        // traverses the node to get the node parser and finally, reads the node providing the concrete class that must be created
        JsonParser parserOfNode = node.traverse(oc);

        return parserOfNode.readValueAs(concreteType);
    }
    
}
```