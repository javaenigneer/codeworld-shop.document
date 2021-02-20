## CodeWorld-Cloud-Shop Gateway技术---奶妈级教学 {docsify-ignore}
>今天呢我们来讲一讲GateWay网关的技术，所谓网关可以理解为统一管理的接口，我们作为入网的第一防线
>将所有的请求统一起来，然后分别转发到各个模块的接口中去

### 简介
```text
Spring Cloud Gateway 是 Spring 官方基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关
Spring Cloud Gateway 旨在为微服务架构提供一种简单而有效的统一的 API 路由管理方式。Spring Cloud Gateway 作为 Spring Cloud 生态系中的网关
目标是替代 Netflix ZUUL，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。
```
### 特性
- 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0
- 动态路由
- Predicates 和 Filters 作用于特定路由
- 集成 Hystrix 断路器
- 集成 Spring Cloud DiscoveryClient
- 易于编写的 Predicates 和 Filters
- 限流
- 路径重写
### 工作流程
```text
客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，
将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（pre）或之后（post）执行业务逻辑。
```
好了，说了这么多，道理谁都懂是吧，那么具体实战效果是怎样的呢？
### 创建一个父级工程
引入依赖
```java
<dependencyManagement>
        <dependencies>
            <!-- springCloud -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR8</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!-- spring-cloud-alibab-dependencies -->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba</artifactId>
                <version>2.2.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
父工程就完成了
### 创建Gateway工程
#### 引入依赖
```java
<!-- nacos注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.2.2.RELEASE</version>
        </dependency>
        <!-- gateway -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
            <version>2.0.3.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-web</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```
#### bootstrap.yml文件
```java
server:
  port: 8888
spring:
  application:
    name: codeworld-gateway
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        service: ${spring.application.name}
        enabled: true
        register-enabled: true
        weight: 1
```
#### application.yml文件
```java
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedHeaders: "*"
            allowedOrigins: "*"
            allowedMethods:
              - GET
                POST
                DELETE
                PUT
                OPTION
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      routes:
        - id: cloud-user
          uri: lb://codeworld-user
          filters:
            - AddRequestHeader=X-Request-Foo, Bar
          predicates:
            - Path=/codeworld-user/**
```
我们来讲讲application.yml文件的配置信息

```text
1.配置网关全局跨域配置信息，是否携带header，设置跨域请求，设置跨域请求的方法
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedHeaders: "*"
            allowedOrigins: "*"
            allowedMethods:
              - GET
                POST
                DELETE
                PUT
                OPTION
2.配置服务启用以及服务名称小写id
discovery:
        locator:
          enabled: true
          lower-case-service-id: true
3.配置服务路由
routes:
        - id: cloud-user # 采用自定义路由 ID
          uri: lb://codeworld-user # 采用 LoadBalanceClient 方式请求，以 lb:// 开头，后面的是注册在 Nacos 上的服务名
          filters:
            - AddRequestHeader=X-Request-Foo, Bar
          predicates:
            - Path=/codeworld-user/** # Predicate 翻译过来是“谓词”的意思，必须，主要作用是匹配用户的请求，有很多种用法
```
#### 在启动器上添加一个注解
```java
@SpringBootApplication
@EnableDiscoveryClient
public class CodeworldGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(CodeworldGatewayApplication.class, args);
    }
}
```
```text
@EnableDiscoveryClient 启动服务发现
```
#### 启动我们的网关模块
在启动之前，我们需要先启动我们的nacos注册服务中心
![nacos-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/windows-start.png)
启动网关模块
![gateway-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-gateway/gateway-demo-start.png)
### 创建user工程
#### 引入依赖
```java
<!-- nacos注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.2.2.RELEASE</version>
        </dependency>
        <!-- web启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.16</version>
        </dependency>
```
#### 修改application.yml文件
```java
spring:
  application:
    name: codeworld-user
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        register-enabled: true
        weight: 1
server:
  port: 8001
```
#### 创建User对象
>User

```java
@Data
public class User {

    private Integer id;

    private String username;
}
```
#### 创建UserController接口
```java
@RestController
@RequestMapping("codeworld-user")
public class UserController {

    @GetMapping("get-user")
    public User getUser(){
       User user = new User();
       user.setId(1);
       user.setUsername("zs");
       return user;
    }
}
```
我们这里可以看见创建了一个user对象，然后通过json数据返回前端
#### 在启动器上添加一个注解
```java
@SpringBootApplication
@EnableDiscoveryClient
public class CodeworldUserApplication {

    public static void main(String[] args) {
        SpringApplication.run(CodeworldUserApplication.class, args);
    }
}
```
启动用户模块
![user-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-gateway/user-demo-start.png)
### 实践
#### 首先我们先通过用户接口来测 ---8001端口
![user-demo-test-8001](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-gateway/user-demo-test-8001.png)
#### 通过gateway网关来测 ---8888端口
![gateway-demo-test-8888](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-gateway/gateway-demo-test-8888.png)
我们在这里可以看见访问的两个接口都可以请求到数据，通过网关来实现接口的转发，可以转发到我们的服务接口上面

我们这里呢只是对gateway的简单讲解，后面还会继续追加
[项目地址](https://github.com/javaenigneer/SpringBoot-demo)
好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！！
### 欢迎加入QQ群(964285437)
 ![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
 ![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)

