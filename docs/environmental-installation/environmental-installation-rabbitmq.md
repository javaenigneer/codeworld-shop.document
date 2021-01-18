## RabbitMQ的安装和使用 {docsify-ignore-all}
### RabbitMQ的安装（我们也是在Linux下安装RabbitMQ）
#### 创建文件夹
```java
// 使用以下命令
cd /usr/local
// 创建一个文件夹
mkdir fcblog(随意起)
// 创建一个文件夹来存放rabbitmq
mkdir rabbitmq
// 移动到该文件夹下
cd rabbitmq
```
#### 安装Erlang
##### 在线安装
```java
yum install esl-erlang_17.3-1~centos~6_amd64.rpm
yum install esl-erlang-compat-R14B-1.el6.noarch.rpm
```
##### 离线安装
将文件上传到rabbitmq下
![Erlang](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/03/7191583232779369.png)
文件下载地址
[Erlang下载地址](https://pan.baidu.com/s/1fyHZ8PD1pDR1Lv7PLZSoMw)
提取码：`wvfw`
```java
// 依次执行以下命令
1）rpm -ivh esl-erlang-17.3-1.x86_64.rpm --force --nodeps
2）rpm -ivh esl-erlang_17.3-1~centos~6_amd64.rpm --force --nodeps
3）rpm -ivh esl-erlang-compat-R14B-1.el6.noarch.rpm --force --nodeps
```
#### 安装RabbitMQ
##### 上传rabbitmq到Linux下的rabbitmq文件夹下
![RabbitMQ](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/03/37821583238636587.png)
[RabbitMQ下载地址](https://pan.baidu.com/s/1FbEtqUILBmqVkfGES5JF6A)
提取码：`vse0`
##### 安装
` rpm -ivh rabbitmq-server-3.4.1-1.noarch.rpm`
##### 设置配置文件
`cp /usr/share/doc/rabbitmq-server-3.4.1/rabbitmq.config.example /etc/rabbitmq/rabbitmq.config`
##### 开启远程访问
`vi /etc/rabbitmq/rabbitmq.config`
![开始远程访问](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/03/8091583238809997.png)
**注意去掉后面的逗号**
##### 启动和停止
```java
service rabbitmq-server start
service rabbitmq-server stop
service rabbitmq-server restart
```
##### 开启Web界面管理工具
```java
rabbitmq-plugins enable rabbitmq_management
// 重启rabbitmq
service rabbitmq-server restart
```
##### 设置开机启动
```java
chkconfig rabbitmq-server on
```
**注意防火墙端口要打开 15672，5672**
### RabbitMQ管理界面
#### 主页预览
![主页预览](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/04/72931583282706452.png)
```java
connections：无论生产者还是消费者，都需要与RabbitMQ建立连接后才可以完成消息的生产和消费，在这里可以查看连接情况

channels：通道，建立连接后，会形成通道，消息的投递获取依赖通道。

Exchanges：交换机，用来实现消息的路由

Queues：队列，即消息队列，消息存放在队列中，等待消费，消费后被移除队列。

端口：

5672: rabbitMq的编程语言客户端连接端口

15672：rabbitMq管理界面端口

25672：rabbitMq集群的端口
```
#### 添加用户
如果不使用guest，我们也可以自己创建一个用户：
![添加用户](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/04/73701583282932776.png)
```java
超级管理员(administrator)：
可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

监控者(monitoring)：
可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)

策略制定者(policymaker)：
可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。

普通管理者(management)：
仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理

其他：
无法登陆管理控制台，通常就是普通的生产者和消费者。
```
#### 创建Virtual Hosts
虚拟主机：类似于mysql中的database。他们都是以“/”开头
![创建Virtual Hosts](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/04/41471583283046073.png)
#### 设置权限
![设置权限1](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/04/11551583283086308.png)
![设置权限2](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/04/89731583283099817.png)
### RabbitMQ基本使用
这里我就直接上实战教程
有兴趣学习的可以去我的博客学习
* [RabbitMQ基本使用一-基本介绍](https://feicheng.xyz/2020/04/05/RabbitMQ%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8%E4%B8%80(%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D)/)
* [RabbitMQ基本使用二-简单队列](https://feicheng.xyz/2020/04/06/RabbitMQ%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8%E4%BA%8C-%E7%AE%80%E5%8D%95%E9%98%9F%E5%88%97/)
* [RabbitMQ基本使用三-工作队列](https://feicheng.xyz/2020/04/07/RabbitMQ%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8%E4%B8%89-%E5%B7%A5%E4%BD%9C%E9%98%9F%E5%88%97/)
* [RabbitMQ基本使用四-发布/订阅模式](https://feicheng.xyz/2020/04/11/RabbitMQ%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8%E5%9B%9B-%E5%8F%91%E5%B8%83-%E8%AE%A2%E9%98%85%E9%98%9F%E5%88%97/)
* [RabbitMQ基本使用五-路由模式](https://feicheng.xyz/2020/04/12/RabbitMQ%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8%E4%BA%94-%E8%B7%AF%E7%94%B1%E6%A8%A1%E5%BC%8F/)
* [RabbitMQ基本使用六-主体模式](https://feicheng.xyz/2020/04/14/RabbitMQ%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8%E5%85%AD-%E4%B8%BB%E9%A2%98%E6%A8%A1%E5%BC%8F/)
* [RabbitMQ实践应用一](https://feicheng.xyz/2020/04/14/RabbitMQ%E5%AE%9E%E8%B7%B5%E5%BA%94%E7%94%A8%E4%B8%80/)
* [RabbitMQ实践应用二-消峰限流](https://feicheng.xyz/2020/04/19/RabbitMQ%E5%AE%9E%E8%B7%B5%E4%BA%8C-%E6%B6%88%E5%B3%B0%E9%99%90%E6%B5%81/)
* [RabbitMQ实践应用二-消峰限流补充](https://feicheng.xyz/2020/04/19/RabbitMQ%E5%AE%9E%E8%B7%B5%E4%BA%8C-%E6%B6%88%E5%B3%B0%E9%99%90%E6%B5%81%E8%A1%A5%E5%85%85/)
### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
