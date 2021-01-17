# CodeWorld-Cloud-Shop商城系统文档学习 {docsify-ignore-all}
## CodeWorld-Cloud-Shop项目简介
CodeWorld-Cloud-Shop是一套比较完整的商城系统，采用的是目前流行的框架技术完成(还在继续开发中)

本套CodeWorld-Cloud-Shop是一个电商项目，后端采用微服务的形式实现，主要采用SpringBoot+MyBatis实现，
融合了多种技术，Nacos(服务注册中心)、Gateway(网关)、OpenFeign（远程调用）Redis(缓存)、RabbitMQ（消息队列）、
ElasticSearch（搜索引擎）、XXL-JOB（任务调度）等技术。
前台商城采用uniapp模板，有商城首页、广告轮播图、分类展示、商品搜索、商品展示、商品规格选择、购物车、订单查询、订单流程等模块。
后台系统采用vue，使用多商户登录平台、有系统管理、商品管理、商户管理、订单管理、营销管理等模块
## CodeWorld-Cloud-Shop项目地址
[后端项目](https://github.com/javaenigneer/codeworld-cloud-shop-api)
[前端项目](https://github.com/javaenigneer/code-shop-system)
[App端项目](https://github.com/javaenigneer/code-shop-app)
## CodeWorld-Cloud-Shop项目技术选型
### 后端技术

|         技术         |        说明         |                         官网                         |
| :------------------: | :-----------------: | :--------------------------------------------------: |
|      SpringBoot      |       MVC框架       |        https://spring.io/projects/spring-boot        |
|       MyBatis        |       ORM框架       |          http://www.mybatis.org/mybatis-3/           |
|  SpringBoot-Gateway  |        网关         |   https://spring.io/projects/spring-cloud-gateway    |
|        Redis         |      缓存服务       |     https://spring.io/projects/spring-data-redis     |
|       RabbitMQ       |      消息队列       |              https://www.rabbitmq.com/               |
|      OpenFeign       |      远程调用       |  https://spring.io/projects/spring-cloud-openfeign   |
|    ElasticSearch     |      搜索引擎       | https://spring.io/projects/spring-data-elasticsearch |
|       XXL-JOB        |      任务调度       |           https://www.xuxueli.com/xxl-job/           |
|         JWT          |     JWT登录支持     |             https://github.com/jwtk/jjwt             |
|        Lombok        |    简化对象封装     |        https://github.com/rzwitserloot/lombok        |
|        Hutool        |     Java工具类      |           https://github.com/looly/hutool            |
|      PageHelper      | MyBatis物理分页插件 |    http://git.oschina.net/free/Mybatis_PageHelper    |
|      Swagger-UI      |    文档生成工具     |      https://github.com/swagger-api/swagger-ui       |
| Hibernator-Validator |      验证框架       |            http://hibernate.org/validator            |
### 前端技术

|    技术    |       说明       |                         官网                          |
| :--------: | :--------------: | :---------------------------------------------------: |
|    Vue     |     前端框架     |                  https://vuejs.org/                   |
| Vue-router |     路由框架     |               https://router.vuejs.org/               |
|    Vuex    | 全局状态管理框架 |                https://vuex.vuejs.org/                |
|  Element   |    前端UI框架    | [https://element.eleme.io](https://element.eleme.io/) |
|   Axios    |   前端HTTP框架   |            https://github.com/axios/axios             |

### App技术
[Uniapp](https://uniapp.dcloud.net.cn/)




