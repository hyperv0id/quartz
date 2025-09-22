---
tags: ["#java/spring #java/spring/spring-cloud"]
---

## 搭建注册中心

项目结构：
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/17/20221117170639-56beb8.png)

## 添加服务端

### 导包

```xml

<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
    <parent>        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-parent</artifactId>  
        <version>2.3.12.RELEASE</version>  
        <relativePath/> <!-- lookup parent from repository -->  
    </parent>  
    <groupId>org.example</groupId>  
    <artifactId>eureka-server</artifactId>  
    <version>0.0.1-SNAPSHOT</version>  
    <name>01-eureka-server</name>  
    <description>Demo project for Spring Boot</description>  
  
  
    <properties>        <java.version>1.8</java.version>  
        <spring-cloud.version>Hoxton.SR12</spring-cloud.version>  
    </properties>  
  
    <dependencies>        <dependency>            <groupId>org.springframework.cloud</groupId>  
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>  
        </dependency>  
        <dependency>            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>    </dependencies>  
    <!--  依赖管理 只是帮你去管理版本号以及子模块的依赖-->  
    <dependencyManagement>  
        <dependencies>            <dependency>                <groupId>org.springframework.cloud</groupId>  
                <artifactId>spring-cloud-dependencies</artifactId>  
                <version>${spring-cloud.version}</version>  
                <type>pom</type>  
                <scope>import</scope>  
            </dependency>        </dependencies>    </dependencyManagement>  
  
    <build>        <plugins>            <plugin>                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
            </plugin>        </plugins>    </build>  
</project>

```

### 配置

```yaml

server:  
  port: 8761 #eureka 默认端口  
  
spring:  
  application:  
    name: eureka-server-01

```

### Java

```java
@SpringBootApplication  
@EnableEurekaServer // 开启eureka注册中心功能  
public class EurekaServer01Application {  
  
    public static void main(String[] args) {  
        SpringApplication.run(EurekaServer01Application.class, args);  
    }  
  
}
```




## 运行

### 运行主程序
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/17/20221117172824-1b5db6.png)
### 浏览器查看

浏览器打开 8761 端口，可以查看`eureka`面板

![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/17/20221117172958-2ed5fe.png)


![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/17/20221117183858-df676a.png)

## 添加客户端

### 代码

需要eureka客户端依赖

```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  
</dependency>
```

在启动类中添加client注解
```java
@SpringBootApplication  
@EnableEurekaClient //eureka 客户端  
public class EurekaClientBApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(EurekaClientBApplication.class, args);  
    }  
  
}
```

### 运行
可以添加多个客户端，只要保证不会端口冲突就行
ok，咱们的客户端和服务端都跑成功了
![](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2022/11/17/20221117185402-e12b4a.png)

