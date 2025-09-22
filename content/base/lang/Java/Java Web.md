---
tags: ["#java #java/javaweb"]
---
# Java Web

[TOC]

![image-20220708122820259](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/08/image-20220708122820259-af18e2.png)

## 数据库

### MySQL数据库

### JDBC

### Maven

#### 概述

Maven 是专门用于管理和构建 Java 项目的工具，主要功能有：

- 提供了标准化的项目结构
- 提供了标准化的构建流程：编译、测试、打包、发布...
- 提供了一套依赖管理机制

![image-20220708134431467](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/08/image-20220708134431467-c083a6.png)

#### 简介

maven 官网：https://maven.apache.org/

![379163696_0308_20220708-135934](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/08/379163696_0308_20220708-135934-813eb1.png)

如下图所示，`groupId`h和`artifactId`负责定位包的位置，而`version`则负责包的版本

![image-20220712224024153](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/12/image-20220712224024153-0306d7.png)

**仓库分类**

- 本地：存放在本地的仓库，表现为自己计算机上的一个目录
- 中央：maven团队维护的仓库，地址 https://repo1.maven.org.maven2/
- 私有：公司团队搭建或维护的仓库，如阿里云镜像

![image-20220712224409998](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/12/image-20220712224409998-ae56e7.png)

#### 安装配置

**配置本地仓库**

maven会把仓库存放在这里：

```xml
  <localRepository>D:\CS\env\maven\apache-maven-3.8.6\repo</localRepository>
```

**配置阿里云私服**：

在mirrors下添加下面代码

```xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

#### 基本使用

**常用命令**

- `compile`：编译
- `clean`：清理
- `test`：测试
- `package`：打包
- `install`：安装

**生命周期**

maven对项目构建的生命周期分为三部分

- clean：清理工作
- default：核心工作
- site：产生报告、发布站点等

在同一生命周期，maven在执行后面命令时会自动运行前面的**所有**命令

![image-20220712230601015](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/12/image-20220712230601015-a52056.png)

#### IDEA配置

idea自带一个maven，我们可以改成自己的maven

![image-20220712230947482](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/12/image-20220712230947482-41e14c.png)

先设置maven路径

![image-20220712231215232](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/12/image-20220712231215232-2c3124.png)

然后把xml配置和本地仓库改成自己的

![image-20220712231153913](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/12/image-20220712231153913-58fe13.png)

#### 依赖管理

在`pom.xml`中编写，在`dependencies`中添加

```xml
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.29</version>
        </dependency>
    </dependencies>
```

### MyBatis

#### 简介

一块**持久层**框架，用于简化JDBC开发

持久层：负责将数据保存到数据库的代码层。JavaEE分为：表现层、业务层和持久层

MaBatis通过配置文件优化了JDBC硬编码的缺点，并且简化了操作

![image-20220712231718521](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/12/image-20220712231718521-8d42ff.png)

#### 快速入门

1. 创建user表，添加数据
2. 创建模块，导入坐标
3. 编写MyBatis 核心配置文件 -- > 替换连接信息 解决硬编码问题
4. 编写 SQL映射文件一>统一管理sql语句，解决硬编码问题
5. 编码：
   1. 定义POJO类
   2. 加载核心配置文件，获取`SqlSessionFactory`对象
   3. 获取`SqlSession`对象，执行SQL语句
   4. 释放资源

1. 首先创建表，添加数据
   
   ```sql
   CREATE table tb_user(
       id int primary key auto_increment,
       username varchar(20),
       password varchar(20),
       gender varchar(20),
       addr varchar(30)
   );
   
   insert into tb_user values(1,'张三','123','男','北交');
   insert into tb_user values(2,'李四','234','男','北极');
   insert into tb_user values(3,'王五','111','女','北极星');
   ```

2. 导入模块
   
   ```xml
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.5.9</version>
           </dependency>
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>8.0.28</version>
           </dependency>
           <dependency>
               <groupId>ch.qos.logback</groupId>
               <artifactId>logback-classic</artifactId>
               <version>1.2.11</version>
           </dependency>
           <dependency>
               <groupId>ch.qos.logback</groupId>
               <artifactId>logback-core</artifactId>
               <version>1.2.11</version>
           </dependency>
   ```

3. 将`logback.xml`放到`resources`目录下
   
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE configuration>
   <configuration>
   
     <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
       <encoder>
         <pattern>%5level [%thread] - %msg%n</pattern>
       </encoder>
     </appender>
   
     <logger name="org.mybatis.example.BlogMapper">
       <level value="trace"/>
     </logger>
     <root level="error">
       <appender-ref ref="stdout"/>
     </root>
   
   </configuration>
   ```
   
   编写mybatis核心配置文件
   
   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
     PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
     <environments default="development">
       <environment id="development">
         <transactionManager type="JDBC"/>
         <dataSource type="POOLED">
           <property name="driver" value="${driver}"/>
           <property name="url" value="${url}"/>
           <property name="username" value="${username}"/>
           <property name="password" value="${password}"/>
         </dataSource>
       </environment>
     </environments>
     <mappers>
        <!-- 加载sql映射文件，待会修改 --> 
       <mapper resource="org/mybatis/example/BlogMapper.xml"/>
     </mappers>
   </configuration>
   ```
   
   修改连接信息
   
   ```xml
   <!--                连接信息-->
   <property name="driver" value="com.mysql.jdbc.Driver"/>
   <property name="url" value="jdbc:mysql:///mybatis?useSSL=false"/>
   <property name="username" value="root"/>
   <property name="password" value="mypassword"/>
   ```

4. 编写SQL映射文件
   
   创建User的Java类
   
   ```java
   package org.example;
   
   public class User {
       private Integer id;
       private String username;
       private String password;
       private String gender;
       private String addr;
       // 自动创建getter、setter、toString
   }
   ```
   
   填写`UserMapper.xml`
   
   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   <!--namespace：名称空间-->
   <mapper namespace="test">
       <select id="selectAllUser" resultType="org.example.User">
           select * from tb_user;
       </select>
   </mapper>
   ```

5. 写代码
   
   ```java
   package org.example;
   
   public class Main {
       public static void main(String[] args) throws IOException {
           // 1. 加载核心配置文件，获取 sqlSessionFactory
           String resource = "mybatis-config.xml";
           InputStream inputStream = Resources.getResourceAsStream(resource);
           SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
   
           // 2. 获取 sqlSessionFactory 对象
           SqlSession sqlSession = sqlSessionFactory.openSession();
           List<User> users = sqlSession.selectList("test.selectAllUser");
           System.out.println(users);
           // 3. 关闭session
           sqlSession.close();
       }
   }
   ```

**注意事项**

1. 若出现不允许检索公钥，则修改`mybatis-config`中的`SSL`为`false`
2. 若出现连接错误，需要仔细查看jdbc、账密等是否输入正确

#### Mapper代理开发

**问题引入**

```java
List<User> users = sqlSession.selectList("test.selectAllUser");
```

上面的代码不能修改，存在硬编码问题

> 诚然，这种方式能够正常工作，对使用旧版本 MyBatis 的用户来说也比较熟悉。但现在有了一种更简洁的方式——使用和指定语句的参数和返回值相匹配的接口（比如 BlogMapper.class），现在你的代码不仅更清晰，更加类型安全，还不用担心可能出错的字符串字面值以及强制类型转换。

我们将Main方法改成这样

```java
package org.example;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.example.mapper.UserMapper;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class Main {
    public static void main(String[] args) throws IOException {
        // 1. 加载核心配置文件，获取 sqlSessionFactory
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        // 2. 获取 sqlSessionFactory 对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> users = mapper.selectAllUser();
        System.out.println(users);
        // 3. 关闭session
        sqlSession.close();
    }
}
```

其中我们做了很多事情

比如UserMapper的配置：

1. `UserMapper`接口，方法名为配置文件中的名称

2. `UserMapper.xml`与`UserMapper.java`的路径改到`org.example.mapper`

3. 在`config.xml`中修改路径
   
   ```xml
   <package name="org/example/mapper"/>
   ```

#### Mapper核心配置文件

官网地址：https://mybatis.org/mybatis-3/zh/configuration.html

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

- configuration（配置）
  - [properties（属性）](https://mybatis.org/mybatis-3/zh/configuration.html#properties)
  - [settings（设置）](https://mybatis.org/mybatis-3/zh/configuration.html#settings)
  - [typeAliases（类型别名）](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)
  - [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)
  - [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory)
  - [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins)
  - environments（环境配置）
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
  - [databaseIdProvider（数据库厂商标识）](https://mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)
  - [mappers（映射器）](https://mybatis.org/mybatis-3/zh/configuration.html#mappers)

## 前端

### HTML

### CSS

### javascript

### AJAX

### Vue

### Element

## Web 核心

### B/S 架构

浏览器、服务器架构，客户端只需要浏览器就可以获取web资源，服务器将web资源分发给浏览器就行
好处：用户不需要部署
服务端存放资源分为

- 静态资源：html，css，js
- 动态资源：servelet，jsp
- 数据库：存放存储数据

### HTTP简介

HyperText Transform Protocol，超文本传输协议，规定了浏览器与服务器之间传输数据的规则
HTTP特点：

- 基于TCP协议，安全
- 基于请求响应模型，一次请求一次响应
- 无状态传输协议，每次请求响应都是独立的
  - 缺点：多次请求不能共享数据（在Java中会使用cookie和session解决）
  - 优点：速度快

#### HTTP请求数据格式

1. 请求行：请求数据第一行，Get表示请求方式，其后HTTP协议跟随版本
2. 请求头：`key：value`键值对     
3. 请求体：POST请求最后内容，存放请求参数

常见请求头：

- host：请求主机名
- User-Agent：浏览器版本
- Accept：浏览器能接受的文件类型 如`text/*`（所有文本），或者`*/*`（所有）
- Accept-Language：浏览器偏好语言
- Accept-Encoding：浏览器支持的压缩类型，如gzip、deflate等

get和post区别

- get没有请求体，请求参数在请求行中
- post请求参数在请求体中
- get的参数有长度限制，post没有
  
  ```
  GET / HTTP/1.1
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
  Accept-Encoding: gzip, deflate, br
  Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
  Cache-Control: max-age=0
  Connection: keep-alive
  Cookie: BAIDUID=C79CF9C0628681BB68FD0B9142329805:FG=1; BAIDUID_BFESS=C79CF9C0628681BB68FD0B9142329805:FG=1; BDUSS=ExZQ0xsM09seDBYazRZfllSWm1HTDRsUUVDM3lhc3R2aHlWMWxyTVlwZGNFcXBpRVFBQUFBJCQAAAAAAAAAAAEAAABn1f-waGNoanVnYgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFyFgmJchYJiU; BDUSS_BFESS=ExZQ0xsM09seDBYazRZfllSWm1HTDRsUUVDM3lhc3R2aHlWMWxyTVlwZGNFcXBpRVFBQUFBJCQAAAAAAAAAAAEAAABn1f-waGNoanVnYgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFyFgmJchYJiU; H_WISE_SIDS=110085_127969_176399_179346_184716_189755_190617_194085_194530_196427_197242_197711_197957_199022_199572_201193_203264_203310_203880_203882_203885_204122_204713_204715_204717_204720_204859_204914_205218_205420_205424_206928_207697_207729_208001_208064_208608_208721_208804_209345_209395_209437_209456_209568_209841_209956_210096_210292_210298_210359_210439_210443_210669_210733_210792_210851_210892_210969_211016_211023_211029_211111_211181_211207_211301_211412_211442_211456_211580_211691_211731_211759_212295_212618_212702_212773_212797_212830_212847_212912_212924_213003_213038_213094_213140_213216_213255_213274_213354_213393_213416_213558_213613; ZFY=nQKSBgBdrzS8tvOLJJseHrRwJP1ld4a:BNhyi3ZjzEj8:C; BIDUPSID=C79CF9C0628681BB68FD0B9142329805; PSTM=1652968812; BD_HOME=1; H_PS_PSSID=36428_36367_34812_36420_36167_35979_36055_36235_26350_36447; BD_UPN=12314753; BA_HECTOR=ah2l0h208g8ha425051h8cjbd15
  Host: www.baidu.com
  Referer: https://www.google.com/
  Sec-Fetch-Dest: document
  Sec-Fetch-Mode: navigate
  Sec-Fetch-Site: cross-site
  Sec-Fetch-User: ?1
  Upgrade-Insecure-Requests: 1
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4951.67 Safari/537.36
  sec-ch-ua: " Not A;Brand";v="99", "Chromium";v="101", "Google Chrome";v="101"
  sec-ch-ua-mobile: ?0
  sec-ch-ua-platform: "Windows"
  ```

#### HTTP响应数据格式

三部分

1. 响应行：格式为`HTTP协议 响应状态码 状态码描述`
2. 响应头：`key: value`
3. 响应体：最后一部分，存放响应数据
   
   | 状态码分类 | 说明                                  |
   | ----- | ----------------------------------- |
   | 1XX   | **响应中**——表示请求已接受，应继续请求或忽略           |
   | 2XX   | **成功**——请求成功接受，请求完成                 |
   | 3XX   | **重定向**——重定向到其他地方，它让客服端在发起一个请求以完成处理 |
   | 4XX   | **客户端错误**——发生错误，责任在客户端              |
   | 5XX   | **服务端错误**——发生错误，责任在服务端              |

常见状态码

| 状态码 | 状态码英文名称                         | 中文描述                                                                                 |
| --- | ------------------------------- | ------------------------------------------------------------------------------------ |
| 100 | Continue                        | 继续。客户端应继续其请求                                                                         |
| 101 | Switching Protocols             | 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议                                    |
|     |                                 |                                                                                      |
| 200 | OK                              | 请求成功。一般用于GET与POST请求                                                                  |
| 201 | Created                         | 已创建。成功请求并创建了新的资源                                                                     |
| 202 | Accepted                        | 已接受。已经接受请求，但未处理完成                                                                    |
| 203 | Non-Authoritative Information   | 非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本                                                 |
| 204 | No Content                      | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档                                         |
| 205 | Reset Content                   | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域                                    |
| 206 | Partial Content                 | 部分内容。服务器成功处理了部分GET请求                                                                 |
|     |                                 |                                                                                      |
| 300 | Multiple Choices                | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择                                  |
| 301 | Moved Permanently               | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替                |
| 302 | Found                           | 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI                                                 |
| 303 | See Other                       | 查看其它地址。与301类似。使用GET和POST请求查看                                                         |
| 304 | Not Modified                    | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源     |
| 305 | Use Proxy                       | 使用代理。所请求的资源必须通过代理访问                                                                  |
| 306 | Unused                          | 已经被废弃的HTTP状态码                                                                        |
| 307 | Temporary Redirect              | 临时重定向。与302类似。使用GET请求重定向                                                              |
|     |                                 |                                                                                      |
| 400 | Bad Request                     | 客户端请求的语法错误，服务器无法理解                                                                   |
| 401 | Unauthorized                    | 请求要求用户的身份认证                                                                          |
| 402 | Payment Required                | 保留，将来使用                                                                              |
| 403 | Forbidden                       | 服务器理解请求客户端的请求，但是拒绝执行此请求                                                              |
| 404 | Not Found                       | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面                              |
| 405 | Method Not Allowed              | 客户端请求中的方法被禁止                                                                         |
| 406 | Not Acceptable                  | 服务器无法根据客户端请求的内容特性完成请求                                                                |
| 407 | Proxy Authentication Required   | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权                                                    |
| 408 | Request Time-out                | 服务器等待客户端发送的请求时间过长，超时                                                                 |
| 409 | Conflict                        | 服务器完成客户端的 PUT 请求时可能返回此代码，服务器处理请求时发生了冲突                                               |
| 410 | Gone                            | 客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置               |
| 411 | Length Required                 | 服务器无法处理客户端发送的不带Content-Length的请求信息                                                   |
| 412 | Precondition Failed             | 客户端请求信息的先决条件错误                                                                       |
| 413 | Request Entity Too Large        | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息 |
| 414 | Request-URI Too Large           | 请求的URI过长（URI通常为网址），服务器无法处理                                                           |
| 415 | Unsupported Media Type          | 服务器无法处理请求附带的媒体格式                                                                     |
| 416 | Requested range not satisfiable | 客户端请求的范围无效                                                                           |
| 417 | Expectation Failed              | 服务器无法满足Expect的请求头信息                                                                  |
|     |                                 |                                                                                      |
| 500 | Internal Server Error           | 服务器内部错误，无法完成请求                                                                       |
| 501 | Not Implemented                 | 服务器不支持请求的功能，无法完成请求                                                                   |
| 502 | Bad Gateway                     | 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应                                              |
| 503 | Service Unavailable             | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中                              |
| 504 | Gateway Time-out                | 充当网关或代理的服务器，未及时从远端服务器获取请求                                                            |
| 505 | HTTP Version not supported      | 服务器不支持请求的HTTP协议的版本，无法完成处理                                                            |

### Tomcat

#### 简介&基本使用

Apache的开源项目，开源免费的轻量级Web服务器，支持Servelet/JSP少量的JAVAEE规范

默认端口号：8080

![image-20220713230401957](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/13/image-20220713230401957-393629.png)

#### Web项目结构

![image-20220713230925166](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/2022/07/13/image-20220713230925166-7a1b9f.png)

#### Idea集成本地Tomcat

#### Tomcat的maven插件

### Servelet

### Request&Response

### JSP

### 会话

### Filter&Listener

## 综合案例
