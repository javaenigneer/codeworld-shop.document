## CodeWorld-Cloud-Shop XXL-JOB入门详解篇---奶妈级教学 
>今天我们来讲一讲定时任务调度，在我们项目中都会用到这个技术，在我们指定的时间内，自动帮我们
>完成一系列的动作，就不用我们自己手动输出，这是不是感觉很好啊，那么我们今天就来入门啦

### 技术选型
我们在使用任务调度的时候呢，我们都知道还有很多的框架，比如Quartz，Elastic-Job，TBSchedule等等
不过我们要通过自己的技术选型来决定我们想要选定哪一种
每个都有自己的优缺点，可能和这个不能满足你的需求，那么我们就要考虑一下另外一个能不能满足呢？
废话有点多啦
### 简介
XXL-JOB是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。
### 特性
- 1、简单：支持通过Web页面对任务进行CRUD操作，操作简单，一分钟上手；
- 2、动态：支持动态修改任务状态、启动/停止任务，以及终止运行中任务，即时生效；
- 3、调度中心HA（中心式）：调度采用中心式设计，“调度中心”自研调度组件并支持集群部署，可保证调度中心HA；
- 4、执行器HA（分布式）：任务分布式执行，任务"执行器"支持集群部署，可保证任务执行HA；
- 5、注册中心: 执行器会周期性自动注册任务, 调度中心将会自动发现注册的任务并触发执行。同时，也支持手动录入执行器地址；
- 6、弹性扩容缩容：一旦有新执行器机器上线或者下线，下次调度时将会重新分配任务；
- 7、路由策略：执行器集群部署时提供丰富的路由策略，包括：第一个、最后一个、轮询、随机、一致性HASH、最不经常使用、最近最久未使用、故障转移、忙碌转移等；
- 8、故障转移：任务路由策略选择"故障转移"情况下，如果执行器集群中某一台机器故障，将会自动Failover切换到一台正常的执行器发送调度请求。
- 9、阻塞处理策略：调度过于密集执行器来不及处理时的处理策略，策略包括：单机串行（默认）、丢弃后续调度、覆盖之前调度；
- 10、任务超时控制：支持自定义任务超时时间，任务运行超时将会主动中断任务；
- 11、任务失败重试：支持自定义任务失败重试次数，当任务失败时将会按照预设的失败重试次数主动进行重试；其中分片任务支持分片粒度的失败重试；
- 12、任务失败告警；默认提供邮件方式失败告警，同时预留扩展接口，可方便的扩展短信、钉钉等告警方式；
- 13、分片广播任务：执行器集群部署时，任务路由策略选择"分片广播"情况下，一次任务调度将会广播触发集群中所有执行器执行一次任务，可根据分片参数开发分片任务；
- 14、动态分片：分片广播任务以执行器为维度进行分片，支持动态扩容执行器集群从而动态增加分片数量，协同进行业务处理；在进行大数据量业务操作时可显著提升任务处理能力和速度。
- 15、事件触发：除了"Cron方式"和"任务依赖方式"触发任务执行之外，支持基于事件的触发任务方式。调度中心提供触发任务单次执行的API服务，可根据业务事件灵活触发。
- 16、任务进度监控：支持实时监控任务进度；
- 17、Rolling实时日志：支持在线查看调度结果，并且支持以Rolling方式实时查看执行器输出的完整的执行日志；
- 18、GLUE：提供Web IDE，支持在线开发任务逻辑代码，动态发布，实时编译生效，省略部署上线的过程。支持30个版本的历史版本回溯。
- 19、脚本任务：支持以GLUE模式开发和运行脚本任务，包括Shell、Python、NodeJS、PHP、PowerShell等类型脚本;
- 20、命令行任务：原生提供通用命令行任务Handler（Bean任务，"CommandJobHandler"）；业务方只需要提供命令行即可；
- 21、任务依赖：支持配置子任务依赖，当父任务执行结束且执行成功后将会主动触发一次子任务的执行, 多个子任务用逗号分隔；
- 22、一致性：“调度中心”通过DB锁保证集群分布式调度的一致性, 一次任务调度只会触发一次执行；
- 23、自定义任务参数：支持在线配置调度任务入参，即时生效；
- 24、调度线程池：调度系统多线程触发调度运行，确保调度精确执行，不被堵塞；
- 25、数据加密：调度中心和执行器之间的通讯进行数据加密，提升调度信息安全性；
- 26、邮件报警：任务失败时支持邮件报警，支持配置多邮件地址群发报警邮件；
- 27、推送maven中央仓库: 将会把最新稳定版推送到maven中央仓库, 方便用户接入和使用;
- 28、运行报表：支持实时查看运行数据，如任务数量、调度次数、执行器数量等；以及调度报表，如调度日期分布图，调度成功分布图等；
- 29、全异步：任务调度流程全异步化设计实现，如异步调度、异步运行、异步回调等，有效对密集调度进行流量削峰，理论上支持任意时长任务的运行；
- 30、跨语言：调度中心与执行器提供语言无关的 RESTful API 服务，第三方任意语言可据此对接调度中心或者实现执行器。除此之外，还提供了 “多任务模式”和“httpJobHandler”等其他跨语言方案；
- 31、国际化：调度中心支持国际化设置，提供中文、英文两种可选语言，默认为中文；
- 32、容器化：提供官方docker镜像，并实时更新推送dockerhub，进一步实现产品开箱即用；
- 33、线程池隔离：调度线程池进行隔离拆分，慢任务自动降级进入"Slow"线程池，避免耗尽调度线程，提高系统稳定性；
- 34、用户管理：支持在线管理系统用户，存在管理员、普通用户两种角色；
- 35、权限控制：执行器维度进行权限控制，管理员拥有全量权限，普通用户需要分配执行器权限后才允许相关操作；

以上呢直接可以在官网查看，嘿嘿嘿嘿嘿

好了，我就不多比比了，直接开始我们的正题

首先呢，我们要怎么开始我们的第一个任务呢

我们先去GitHub上拉取最新的代码 [XXL-JOB](https://github.com/xuxueli/xxl-job/)

### 快速入门
将代码以maven的方式导入到我们的IDE中，这里我相信大多数人都是用的IDEA吧，如果你的IDE不是这个，那么你就太落伍啦，赶快行动起来吧
![xxl-job](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E7%BB%93%E6%9E%84.png)
这里面就显示我们需要的模块
```text
xxl-job-admin: 调度中心
xxl-job-core:  公共依赖
xxl-job-executor-samples：执行器实例
```
主要的呢还是我们的调度中心和公共依赖这个两个模块
```text
xxl-job-executor-sample-spring：Spring版本，通过Spring容器管理执行器，比较通用
xxl-job-executor-sample-frameless：无框架版本
xxl-job-executor-sample-springboot：Springboot版本，通过Springboot管理执行器，推荐这种方式
```
#### 初始化数据库
```text
在我们解压后的文件夹中我们可以看见有一个sql文件
在/xxl-job/doc/db/tables_xxl_job.sql
```
#### 部署调度中心
```text
调度中心项目：xxl-job-admin
作用：统一管理任务调度平台上调度任务，负责触发调度执行，并且提供任务管理平台，可视化操作
```
##### 调度中心位置
首先找到我们的application.properties（我相信这个就不用多说了吧，相信学过SpringBoot的都知道它在哪儿）
修改一下配置文件里的信息
```java
### xxl-job, datasource
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
主要就是这个数据库的配置，其他先不做修改
##### 启动项目
```text
然后访问我们的url
Http://localhost:9999(端口)/xxl-job-admin
登录账户默认是admin 123456
就会出现这个首页信息
```
![xxl-job-index](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E8%B0%83%E5%BA%A6%E9%A6%96%E9%A1%B5.png)
到这里我们的调度中心就配置启动好了，是不是很简单呢

### 实现执行器
首先我们可以使用项目里面的模板，我们也可以自己实现
这里呢我们就单独出来做一个
那么这个xxl-job-admin一直运行着
那么我们就需要去手动创建一个SpringBoot项目
#### 引入主要的Pom文件
```java
 <dependency>
            <groupId>com.xuxueli</groupId>
            <artifactId>xxl-job-core</artifactId>
            <version>2.2.0</version>
        </dependency>
```
#### 在我们的配置文件中添加如下信息(我这里使用的是yml文件)
```java
xxl:
  job:
    accessToken:
    admin:
      addresses: http://localhost:9999/xxl-job-admin
    executor:
      address:
      appname: xxl-job-marketing
      ip:
      logpath: /data/applogs/xxl-job/jobhandler
      logretentiondays: 30
      port: 9004
```
字段说明
```text
accessToken：执行器通讯TOKEN [选填]：非空时启用；
admin.addresses：调度中心部署跟地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册；
exector.address：注册表地址，如果为空，则使用ip地址作为注册表地址
exector.appname：执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册
exector.ip：执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯实用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"
exector.logpath：执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
exector.logretentiondays：执行器日志保存天数 [选填] ：值大于3时生效，启用执行器Log文件定期清理功能，否则不生效
exector.port：执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
```
#### 任务执行器组件配置，初始化我们的配置信息
```java
@Configuration
@Slf4j
public class XxlJobConfig {

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.address}")
    private String address;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        log.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);
        return xxlJobSpringExecutor;
    }
```
#### 创建任务执行类
```java
@Component
@Slf4j
public class testTask {

    @XxlJob(value = "testTask")
    public ReturnT<String> testTask(String param){

        log.info("参数信息：" + param);

        return ReturnT.SUCCESS;
    }
}
```
这样我们的代码就写完啦

那我们我们怎么去测试呢

首先来到我们的可视化界面，点击执行器管理，添加执行器
![add-handler](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/add-handler.png)

添加执行器页面
![add-handler-index](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E6%B7%BB%E5%8A%A0%E9%A1%B5%E9%9D%A2.png)

添加成功
![add-handler-success](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E6%89%A7%E8%A1%8C%E5%99%A8%E6%B7%BB%E5%8A%A0%E6%88%90%E5%8A%9F.png)

来到我们的任务管理界面，添加一个新任务
![add-task](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E4%BB%BB%E5%8A%A1%E6%B7%BB%E5%8A%A0%E6%88%90%E5%8A%9F%E7%95%8C%E9%9D%A2.png)

![add-task-success](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E4%BB%BB%E5%8A%A1%E7%95%8C%E9%9D%A2.png)
至此我们的任务就添加成功了，那么我们怎么启动呢？

这个就好办啦，直接可以动一动手就完了
![result](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E6%89%A7%E8%A1%8C%E4%BB%BB%E5%8A%A1.png)
点击执行一次，我们就可以在我们的后台看见啦
![start](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E6%89%A7%E8%A1%8C%E6%88%90%E5%8A%9F.png)
以上就是我们的XXL-JOB基本教程啦，你学费了吗？

没有就多学吧，要多学多问

这样才能够快速成长，就会越学越费

好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！！
### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)




