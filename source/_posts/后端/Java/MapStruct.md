---
title: MapStruct 对象转换框架
date: 2024-06-13 12:10:45
tags: [ MapStruct]
categories: [ 后端 , Java  ]

---

# MapStruct 对象转换框架



## 导入依赖

```xml
            <mapstruct.version>1.4.1.Final</mapstruct.version>
    
    
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${mapstruct.version}</version>
    </dependency>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct-processor</artifactId>
        <version>${mapstruct.version}</version>
    </dependency>
```

## 原理

原理与 `lomback` 类似` ,都是通过自动生成代码;

## 使用方法

### 普通使用

##### 定义映射接口

```java
public interface GoodsInfoMapper {

    /**
     * 无状态且线程安全
     */
    GoodsInfoMapper INSTANCE = Mappers.getMapper(GoodsInfoMapper.class);
}	
```

##### 使用

```java
GoodsInfoPO goodsInfoPO = GoodsInfoMapper.INSTANCE.goodsInfoDtoToPo(goodsInfoDTO);
```

### 在Spring中使用

##### 定义映射接口

```java
@Mapper(componentModel = "spring") // 设置为 spring
public interface GoodsInfoMapper {

    /**
     * 无状态且线程安全
     */
    GoodsInfoMapper INSTANCE = Mappers.getMapper(GoodsInfoMapper.class);
}	
```

有了 `@Mapper(componentModel = "spring)` 这个后.生成的实现类上会自动加上  `@Component` 注解



#### @Mapping

`@Mapping` 注解是 MapStruct 框架中的一个关键注解，用于定义两个 bean 之间的字段映射关系。它可以应用在 Mapper 接口的方法上，指定源（source）字段到目标（target）字段的映射关系。

### 基本用法

```java
@Mapper
public interface CarMapper {

    @Mapping(source = "make", target = "manufacturer")
    CarDto carToCarDto(Car car);
}
```

在上述示例中，`@Mapping(source = "make", target = "manufacturer")` 表示将 `Car` 对象的 `make` 字段映射到 `CarDto` 对象的 `manufacturer` 字段上。

### 常见属性

- **source**：源对象的字段名。

- **target**：目标对象的字段名。

- **qualifiedByName**：通过一个命名转换器来转换字段值。

- **defaultValue**：指定目标字段的默认值。

- **ignore**：忽略某个字段，不进行映射。

- **expression**：使用 SpEL 表达式定义映射规则。

  `This attribute can not be used together with source(), defaultValue(), defaultExpression() or expression().`

### 示例

1. **简单映射**

```java
@Mapping(source = "name", target = "fullName")
PersonDto personToPersonDto(Person person);
```

这个示例将 `Person` 对象的 `name` 字段映射到 `PersonDto` 对象的 `fullName` 字段。

1. **使用转换器**

```jaVA
@Mapping(source = "birthDate", target = "birthDate", dateFormat = "dd-MM-yyyy")
PersonDto personToPersonDto(Person person);
```

这个示例中，`dateFormat` 属性指定了日期格式，MapStruct 会自动将 `Person` 对象的 `birthDate` 字段按照指定的格式转换为 `PersonDto` 对象的 `birthDate` 字段。

1. **忽略字段**

```JAVA
@Mapping(target = "id", ignore = true)
PersonDto personToPersonDto(Person person);
```

这个示例中，`ignore = true` 表示忽略 `PersonDto` 对象的 `id` 字段，不进行映射。

### 高级用法

- **多字段映射**：可以同时映射多个字段，通过多个 `@Mapping` 注解实现。
- **复杂类型映射**：支持复杂对象类型，如集合、嵌套对象等。
- **条件映射**：通过 SpEL 表达式在 `@Mapping` 的 `expression` 属性中定义条件，根据条件动态进行映射。

`@Mapping` 注解使得 MapStruct 在生成映射代码时具有灵活性和可配置性，能够应对多种复杂的映射需求。
