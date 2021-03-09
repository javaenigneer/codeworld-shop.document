### CodeWorld-Cloud-Shop项目功能概览 {docsify-ignore}

#### CodeWorld-Cloud-Shop项目简介
CodeWorld-Cloud-shop是一套比较完整的商城系统，采用的是目前流行的框架技术完成(还在继续开发中)

本套CodeWorld-Cloud-Shop是一个电商项目，后端采用微服务的形式实现，主要采用SpringBoot+MyBatis实现，
融合了多种技术，Nacos(服务注册中心)、Gateway(网关)、OpenFeign（远程调用）Redis(缓存)、RabbitMQ（消息队列）、
ElasticSearch（搜索引擎）、XXL-JOB（任务调度）等技术。
前台商城采用uniapp模板，有商城首页、广告轮播图、分类展示、商品搜索、商品展示、商品规格选择、购物车、订单查询、订单流程等模块。
后台系统采用vue，使用多商户登录平台、有系统管理、商品管理、商户管理、订单管理、营销管理等模块
#### CodeWorld-Cloud-Shop 项目启动方式
##### 准备环境
###### 安装Nacos注册中中心
详情请点击[Windows下安装单机版Nacos](/environmental-installation/windows-installation-nacos.md)
###### 安装Redis环境
详情请点击[Redis的安装和使用](/environmental-installation/environmental-installation-redis.md)
###### 安装RabbitMQ环境
详情请点击[RabbitMQ的安装和使用](/environmental-installation/environmental-installation-rabbitmq.md)
###### 安装ElasticSearch环境
详情请点击[ElasticSearch的安装和使用](/environmental-installation/environmental-installation-elasticsearch.md)
以上我们需要的环境就这些了
##### 需要单独启动XXL-JOB任务调度
详情请点击[XXL-JOB动态创建任务详解篇1](/technology/xxl-job-customize.md)
##### 启动项目中的各个服务
这里就不列出来了

好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！！
### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)
