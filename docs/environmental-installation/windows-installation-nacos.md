## CodeWorld-Cloud-Shop Windows下安装单机版Nacos---奶妈级教学
>本篇文章主要讲解的是我们使用的nacos注册中心的安装教程，单机版奶妈级教学

### 下载
首先我们要去官网下载Nacos服务，推荐下载1.1.0版本，有可能版本高会出现错误
[nacos-server-1.1.0](https://github.com/alibaba/nacos/releases/tag/1.1.0)
![nacos-server](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-server.png)
### 解压文件
将下载的压缩包解压后
![nacos-file](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-file.png)
### 进入到conf文件下
![nacos-mysql](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-mysql.png)
我们看见一个nacos-mysql的sql文件，那么我们将这个sql文件导入到数据库中,就有这些数据表信息
![nacos-mysql-table](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-mysql-table.png)
### 修改application.properties
```java
spring.datasource.platform=mysql 
db.num=1 
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user=root
db.password=root
```
上面的改成你自己的信息，数据库ip地址，端口，数据库名以及数据库连接的用户名和密码
![nacos-conf-mysql](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-conf-mysql.png)
### 进入到bin目录下，双击启动start.cmd
![nacos-bin-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-bin-start.png)
### 启动成功
![windows-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/windows-start.png)
### 地址栏输入
`http://localhost:8848/nacos`
![nacos-login](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-login.png)
用户名和密码都是默认的nacos和nacos
### 登录成功后
![nacos-index](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-index.png)

这样我们就将nacos注册中心启动起来了，那么我们怎么将服务注册到nacos注册中心呢
### 导入需要的pom依赖信息
我们在父级模块下导入以下依赖信息
![nacos-server-pom](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-server-pom.png)
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
### 再来到codeworld-cloud-gateway模块
![nacos-gateway-pom](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-gateway-pom.png)
```java
 <!-- nacos注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.2.2.RELEASE</version>
        </dependency>
```
加入nacos的注册服务发现
### 再修改我们的启动类
![nacos-gateway-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-gateway-start.png)
添加一个注册发现注解
@EnableDiscoveryClient

### 启动模块
![nacos-gateway-start-success](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-gateway-start-succss.png)

### 刷新nacos服务端地址栏
![nacos-gateway-injection-success](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/nacos-gateway-injection-success.png)

那么这样我们的服务就注册到了nacos注册中心中

好了，本次的技术分享就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！
### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)
