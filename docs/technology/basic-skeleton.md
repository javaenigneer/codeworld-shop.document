## CodeWorld-Cloud-Shop 项目基本骨架 {docsify-ignore-all}
> 本篇文章主要讲解的是CodeWorld-Cloud-Shop项目的基本骨架，我们需要采用哪些技术来支持，我们怎么整合这些技术的

### CodeWorld-Cloud-Shop项目使用的框架介绍（这里我们就不用去详细介绍）
### SpringBoot
#### 什么？
```text
Spring Boot使创建独立的、基于生产级Spring的应用程序变得很容易，您可以“直接运行”这些应用程序。
我们对Spring平台和第三方库有自己的见解，这样您就可以轻松入门了。大多数Spring引导应用程序只需要很少的Spring配置。
```
#### 特点
```text
1.搭建项目快
2.让测试变的更简单,内置了JUnit、Spring Boot Test等多种测试框架，方便测试
3.Spring Boot让配置变的简单，Spring Boot的核心理念：约定大约配置，约定了某种命名规范，可以不用配置，就可以完成功能开发
比如模型和表名一致就可以不用配置，直接进行CRUD（增删改查）的操作，只有表名和模型不一致的时候，配置名称即可
4.内嵌容器，省去了配置Tomcat的繁琐
5.方便监控，使用Spring Boot Actuator组件提供了应用的系统监控，可以查看应用配置的详细信息
```
### SpringCloud
#### 什么？
```text
SpringCloud是基于SpringBoot的一整套实现微服务的框架。它提供了微服务开发所需的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等组件。
最重要的是，基于SpringBoot，会让开发微服务架构非常方便
```
#### 特点
```text
1.Spring Cloud 来源于 Spring，质量、稳定性、持续性都可以得到保证
2.Spirng Cloud 天然支持 Spring Boot，更加便于业务落地
3.Spring Cloud 发展非常的快，从 2016 年开始接触的时候相关组件版本为 1.x，到现在将要发布 2.x 系列
4.Spring Cloud 是 Java 领域最适合做微服务的框架
5.相比于其它框架，Spring Cloud 对微服务周边环境的支持力度最大
6.对于中小企业来讲，使用门槛较低
7.Spring Cloud 是微服务架构的最佳落地方案
```
### MyBatis
#### 什么？
```text
1.mybatis是一个优秀的基于java的持久层框架，它内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程
2.mybatis通过xml或注解的方式将要执行的各种statement配置起来，并通过java对象和statement中sql的动态参数进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射为java对象并返回。
```
#### 特点
```text
1.mybatis是一种持久层框架，也属于ORM映射。前身是ibatis
2.相比于hibernatehibernate为全自动化，配置文件书写之后不需要书写sql语句，但是欠缺灵活，很多时候需要优化
3.mybatis为半自动化，需要自己书写sql语句，需要自己定义映射。增加了程序员的一些操作，但是带来了设计上的灵活，并且也是支持hibernate的一些特性，如延迟加载，缓存和映射等
```
### SpringGateway
#### 什么？
```text
1.Spring Cloud Gateway 是 Spring 官方基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关
2.Spring Cloud Gateway 旨在为微服务架构提供一种简单而有效的统一的 API 路由管理方式
3.Spring Cloud Gateway 作为 Spring Cloud 生态系中的网关，目标是替代 Netflix ZUUL，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等
```
#### 特点
```text
1.基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0
2.动态路由
3.Predicates 和 Filters 作用于特定路由
4.集成 Hystrix 断路器
5.集成 Spring Cloud DiscoveryClient
6.易于编写的 Predicates 和 Filters
7.限流
8.路径重写
```
### Nacos
#### 什么？
```text
英文全称Dynamic Naming and Configuration Service，Na为naming/nameServer即注册中心,co为configuration即注册中心，service是指该注册/配置中心都是以服务为核心。服务在nacos是一等公民
```
#### 特点
```text
1.Nacos = Spring Cloud Eureka + Spring Cloud Config
2.Nacos 可以与 Spring, Spring Boot, Spring Cloud 集成，并能代替 Spring Cloud Eureka, Spring Cloud Config
3.通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-config 实现配置的动态变更
4.通过 Nacos Server 和 spring-cloud-starter-alibaba-nacos-discovery 实现服务的注册与发现
```
### OpenFeign
#### 什么？
```text
1.Feign 是一个声明式的 Web 服务客户端，让编写 Web 服务客户端变得非常容易，只需要创建一个接口并在接口上添加注解即可。Feign 是 Spring Cloud 组件中的一个轻量级 RESTful 的 HTTP 服务客户端 Feign 内置了 Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。
2.Feign 的使用方式是：使用 Feign 的注解定义接口，调用这个接口就可以调用服务注册中心的服务
```
#### 特点
```text
1.可插拔的注解支持，包括Feign注解和JAX-RS注解
2.支持可插拔的HTTP编码器和解码器（Gson，Jackson，Sax，JAXB，JAX-RS，SOAP）
3.支持Hystrix和它的Fallback
4.支持Ribbon的负载均衡
5.支持HTTP请求和响应的压缩
6.灵活的配置：基于 name 粒度进行配置
7.支持多种客户端：JDK URLConnection、apache httpclient、okhttp，ribbon）
8.支持日志
9.支持错误重试
10.url支持占位符
11.可以不依赖注册中心独立运行
```
### PageHelper
#### 什么？
```text
Mybatis分页插件 - PageHelper
如果你也在用Mybatis，建议尝试该分页插件，这一定是最方便使用的分页插件。
```
#### 特点
```text
一个字，好（哈哈哈哈）
```
### Redis
#### 什么？
```text
Redis是用C语言开发的一个开源的高性能键值对（key-value）数据库。
它通过提供多种键值数据类型来适应不同场景下的存储需求，目前为止Redis支持的键值数据类型如下:
1.字符串类型
2.散列类型
3.列表类型
4.集合类型
6.有序集合类型
```
#### 特点
```text
1.速度快
2.基于键值对的数据结构服务器
3.丰富的功能
4.简单稳定
5.客户端语言多
6.持久化
7.主从复制
8.高可用和分布式
```
### RabbitMQ
#### 什么？
````text
RabbitMQ是支持持久化消息队列的消息中间件。应用在上下游的层次级业务逻辑中，上级业务逻辑相当于生产者发布消息，下级业务逻辑相当于消费者接受到消息并且消费消息。
````
#### 特点
```text
1.可靠性:RabbitMQ使用一些机制来保证可靠性，如持久化、传输确认及发布确认等
2.灵活的路由:在消息进入队列之前，通过交换器来路由消息。对于典型的路由功能，RabbitMQ己经提供了一些内置的交换器来实现。针对更复杂的路由功能，可以将多个交换器绑定在一起，也可以通过插件机制来实现自己的交换器
3.扩展性:多个RabbitMQ节点可以组成一个集群，也可以根据实际业务情况动态地扩展集群中节点。
4.高可用性:队列可以在集群中的机器上设置镜像，使得在部分节点出现问题的情况下队仍然可用。
5.多种协议:RabbitMQ除了原生支持AMQP协议，还支持STOMP，MQTT等多种消息中间件协议
6.多语言客户端:RabbitMQ几乎支持所有常用语言，比如Jav a、Python、Ruby、PHP、C#、JavaScript等。
7.管理界面:RabbitMQ提供了一个易用的用户界面，使得用户可以监控和管理消息、集群中的节点等。
8.插件机制:RabbitMQ提供了许多插件，以实现从多方面进行扩展，当然也可以编写自己的插件。
```
### ElasticSearch
#### 什么？
```text
Elasticsearch是构建在Apache Lucene之上的开源分布式搜索引擎。Lucene是开源的搜索引擎包，使用JAVA实现，允许你通过自己的Java应用程序实现索引存储和搜索功能。Elashticsearch充分利用Lucene，并在其基础上进行了拓展，使得存储，索引，搜索都变得更快更容易更准确。
Elasticsearch除了直接与Java应用程序对接外，其还具备了一套Restful风格的接口，不管是使用什么计算机语言，都能够容易的通过JSON格式的HTTP请求来使用和管理Elasticsearch
```
#### 特点
```text
1.Elasticsearch对复杂分布式机制的透明隐藏特性
2.Elasticsearch的垂直扩容与水平扩容
3.增减或减少节点时的数据rebalance
4.保持负载均衡
5.节点平等的分布式架构
```
### XXL-JOB
#### 什么？
```text
XXL-JOB是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用
```
#### 特点
```text
1.xxl-job 首先采用分布式任务调度框架，可以进行分布式部署
2.xxl-job 简单，界面化对任务进行CRUD操作
3.架构是讲执行器注册到注册中心，也成调度中心，通过调度中心调度任务
4.调度可支持HA
.....
```
目前就使用到了这些基主要的技术，当然还没完，后面会继续更新

### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)
