# Spring Boot

## Maven 依赖

```xml
<dependency>
    <groupId>com.bailizhang.lynxdb</groupId>
    <artifactId>lynxdb-spring-boot-starter</artifactId>
    <version>2023.10.5-snapshot</version>
</dependency>
```

## 包扫描配置

`com.bailizhang.lynxdb.springboot.starter` 是 lynxdb-spring-boot-starter 需要配置扫描的包，`com.bailizhang.website` 是当前应用需要扫描的包。

```java
@ComponentScan({ "com.bailizhang.lynxdb.springboot.starter", "com.bailizhang.website" })
@SpringBootApplication
public class WebsiteApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebsiteApplication.class, args);
    }

}
```

## application.yml

```yaml
com:
  bailizhang:
    lynxdb:
      host: "127.0.0.1"
      port: 7820
```
