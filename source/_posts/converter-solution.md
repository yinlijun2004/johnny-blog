---
title: Spring Boot的枚举最佳实践
date: 2019-04-18 17:31:39
tags: [spring boot, enum]
---

在跟前端交互的过程中，会经常遇到枚举值的处理，这些枚举值往往还会存入数据库，经过一段时间的摸索，自己总结了一套不错的实践方法。

首先在DAO层，定义一个枚举内省
```java

public enum UserOrderType {
    SELL("0"),
    BUY("1");

    private String value;

    private UserOrderType(String value) {
        this.value = value;
    }

    public static UserOrderType fromValue(String value) {
        for(UserOrderType type : values()) {
            if(type.value.equalsIgnoreCase(value)) {
                return type;
            }
        }
        return null;
    }

    @JsonValue
    public String getValue() {
        return value;
    }
}
```

上面的定义中，为每个枚举类型声明一个字符串，另外
```java
    @JsonValue
    public String getValue() {
        return value;
    }
```
这个是避免rest接口返回给客户端的时候，枚举类型默认转换为字面量，如SELL枚举类型就会转换为”SELL“字符串。

另外，定义一个与之对应的converter类。
```java
import org.springframework.core.convert.converter.Converter;

import javax.persistence.AttributeConverter;

public class UserOrderTypeConverter implements Converter<String, UserOrderType>, AttributeConverter<UserOrderType, String> {
    @Override
    public UserOrderType convert(String s) {
        return UserOrderType.fromValue(s);
    }

    @Override
    public String convertToDatabaseColumn(UserOrderType attribute) {
        return attribute.getValue();
    }

    @Override
    public UserOrderType convertToEntityAttribute(String dbData) {
        return UserOrderType.fromValue(dbData);
    }
}
```
这个convert类，实现了两个接口，一个org.springframework.core.convert.converter.Converter，用于转换客户端传过来的字符串值，需要注册
```java
@Configuration
public class GlobalParamConverterConfig {
    @Autowired
    private RequestMappingHandlerAdapter handlerAdapter;

    @PostConstruct
    public void addConversionConfig() {
        ConfigurableWebBindingInitializer initializer = (ConfigurableWebBindingInitializer)handlerAdapter.getWebBindingInitializer();
        if (initializer.getConversionService() != null) {
            GenericConversionService genericConversionService = (GenericConversionService)initializer.getConversionService();
            genericConversionService.addConverter(new UserOrderTypeConverter());
        }
    }
}
```

另一个接口是javax.persistence.AttributeConverter，用于数据库的持久化工作。

所以，在DAO层里的引用该枚举的时候，可以像这样：
```java
@Entity
@Table(name = "t_user_order")
@Builder
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserOrder implements Serializable {
    @Column(name = "order_type", nullable = false)
    @Convert(converter = UserOrderTypeConverter.class)
    private UserOrderType orderType;
}
```

这里不使用@Enumerated注解，而是用@Convert注解。

@Enumerated有两种保存方式，
- ORDINAL 保存序号，如果往中间插入枚举值的话，会引起乱序问题。
保存的是索引值， 
- STRING 会增加数据量。

