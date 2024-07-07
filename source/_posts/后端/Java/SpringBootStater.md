---
title: SpringBoot自定义 Starter
date: 2024-06-08 15:15:02
tags: [ Java ,Spring, SpringBoot]
categories: [ 后端, Java, Spring]

---



适用于将独立模块,封装成spring-boot-stater 自动配置的模块



### 1. 创建 Maven 项目

创建一个普通的 Maven 项目 `custom-example-starter`。

### 2. 导入 Spring Boot 相关依赖

```xml
		<properties>
    <java.version>17</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <spring-boot.version>3.0.2</spring-boot.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <scope>provided</scope>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 3. 声明需要配置的属性和配置名前缀

```java
/**
 * admin 属性
 */
@ConfigurationProperties(prefix = "custom.user.admin")
@Data
public class AdminProperties {
    private Integer id = 1;
    private String username = "admin";
    private String password = "password";
}
```

### 4. 创建自定义配置类，并配置 Bean

```java
@Configuration
@EnableConfigurationProperties(AdminProperties.class)
@ComponentScan("com.example.utils")
public class UserAutoConfiguration {
    @Bean
    User admin(AdminProperties adminProperties) {
        System.err.println("adminProperties: " + adminProperties);
        final User user = new User();
        user.setId(adminProperties.getId());
        user.setUsername(adminProperties.getUsername());
        user.setPassword(adminProperties.getPassword());
        return user;
    }
}
```

### 5. 导入指定的配置类

创建文件：`resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`，并添加以下内容：

```
com.example.utils.UserAutoConfiguration
```

SpringBoot 2.7前后配置有所差异

```
示例 
原spring.factories文件
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.xxx.iot.common.config.SaTokenConfigure,\
com.xxx.iot.common.config.SecurityProperties

在resource/META-INF目录下新建spring目录，并添加org.springframework.boot.autoconfigure.AutoConfiguration.imports文件
com.xxx.iot.common.config.SaTokenConfigure
com.xxx.iot.common.config.SecurityProperties
```



### 6. 在其他 Spring Boot 项目中引用此模块直接使用 Bean

```java
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        final ConfigurableApplicationContext context = SpringApplication.run(Main.class, args);
        final User user = context.getBean(User.class);
        System.out.println("user = " + user);
    }
}
```

### 7. 在 YAML 文件中修改默认值

```yaml
custom:
	user:
		admin:
			id: 22
			username: inyxin
```

通过以上步骤，你就创建了一个自定义的 Spring Boot Starter，并能在其他项目中直接使用对应的 Bean。

