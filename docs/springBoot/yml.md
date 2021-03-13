# yml
## 类引用yml属性
```java
  @Component 或者 @Bean
  @ConfigurationProperties(prefix = "") 可以放在方法或者类上
  @Value("{}") 加在属性上

  //yml中添加实体的提示  ctrl+F9后生效 optional为本项目专用不会传递给子项目
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional> 
  </dependency>
```

## 构造器绑定
```
  类需要@ConstructorBinding
  引用类需要@EnableConfigurationProperties(*.class)
```

## 校验
```
        加注解@Validated

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.2.0.Final</version>
        </dependency>

        @NotNull @Max(10) @Email

        内部类需要增加@Valid才可以
```

