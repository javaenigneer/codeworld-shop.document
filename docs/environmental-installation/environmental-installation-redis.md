## Redis的安装和使用 
### Redis介绍
#### 什么是Redis
Redis是用C语言开发的一个开源的高性能键值对（key-value）数据库。
它通过提供多种键值数据类型来适应不同场景下的存储需求，目前为止Redis支持的键值数据类型如下：
```text
字符串类型
散列类型
列表类型
集合类型
有序集合类型
```
#### Redis的使用场景
```text
缓存（数据查询、短连接、新闻内容、商品内容等）----》最多使用
分布式集群架构中的session分离
聊天室在线好友列表
任务队列（秒杀、抢购、12306等）
应用排行榜
网站访问统计
数据过期处理（Redis的过期时间监听）
```
#### Redis安装
Redis一般我们都是在服务器上的，本地很少使用，那么我们就直接在Linux上安装Redis
相信你已经对Linux的常用命令都熟练了
```java
Redis是C语言开发，建议在Linux上运行，本教程使用Centos7作为安装环境。
安装redis需要先将官网下载的源码进行编译，编译依赖gcc环境，如果没有gcc环境，需要安装gcc：
yum install gcc-c++
- 版本说明
 本教程使用Redis3.0版本。3.0版本主要增加了集群功能
- 源码下载
从官网下载
http://download.redis.io/releases/redis-3.0.0.tar.gz
将redis-3.0.0.tar.gz拷贝到 /usr/local
- 解压源码
tar -zxvf redis-3.0.0.tar.gz 
- 进入解压后的目录进行编译
cd /usr/local/redis-3.0.0
make
- 安装到指定目录,如 /usr/local/redis
cd /usr/local/redis-3.0.0
make PREFIX=/usr/local/redis install
- redis.conf
redis.conf是Redis的配置文件，redis.conf在redis源码目录
注意修改port作为redis进程的端口，port默认为6379
- 拷贝配置文件到安装目录下
进入源码目录，里面有一份配置文件 redis.conf，然后将其拷贝到安装路径下
cd /usr/local/redis
makdir conf
cp /usr/local/redis-3.0.0/redis.conf /usr/local/redis/bin
```
#### Redis的启动
##### 前端启动
```text
直接运行bin/redis-server将以前端模式启动，前端模式启动的缺点是ssh命令窗口关闭则redis-server程序结束，不推荐使用此方法
```
##### 后端启动
```text
修改redis.conf配置文件， daemonize yes 以后端模式启动
执行如下命令启动Redis
cd /usr/local/redis
./bin/redis-server ./redis.conf
redis启动的默认端口是6379
```
**注意的是，我们看看我们的防火墙是否开启，如果开启我们就需要开启我们的端口6379**
[防火墙的使用](../firewall/firewall-use.md)
### Redis的基本使用(相信你已经会使用SpringBoot)
#### 引入依赖
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
#### application.yml文件
```java
redis:
  host: 192.168.2.4(你虚拟机主机ip地址)
  database: 3(使用的哪一个库)
```
#### 创建User对象
```java
private Integer id;
private String username;
```
#### Json的一个工具类
```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.lang.Nullable;

import java.io.IOException;
import java.util.List;
import java.util.Map;

public class JsonUtils {

    public static final ObjectMapper mapper = new ObjectMapper();

    private static final Logger logger = LoggerFactory.getLogger(JsonUtils.class);

    @Nullable
    public static String serialize(Object obj) {
        if (obj == null) {
            return null;
        }
        if (obj.getClass() == String.class) {
            return (String) obj;
        }
        try {
            return mapper.writeValueAsString(obj);
        } catch (JsonProcessingException e) {
            logger.error("json序列化出错：" + obj, e);
            return null;
        }
    }

    @Nullable
    public static <T> T parse(String json, Class<T> tClass) {
        try {
            return mapper.readValue(json, tClass);
        } catch (IOException e) {
            logger.error("json解析出错：" + json, e);
            return null;
        }
    }

    @Nullable
    public static <E> List<E> parseList(String json, Class<E> eClass) {
        try {
            return mapper.readValue(json, mapper.getTypeFactory().constructCollectionType(List.class, eClass));
        } catch (IOException e) {
            logger.error("json解析出错：" + json, e);
            return null;
        }
    }

    @Nullable
    public static <K, V> Map<K, V> parseMap(String json, Class<K> kClass, Class<V> vClass) {
        try {
            return mapper.readValue(json, mapper.getTypeFactory().constructMapType(Map.class, kClass, vClass));
        } catch (IOException e) {
            logger.error("json解析出错：" + json, e);
            return null;
        }
    }

    @Nullable
    public static <T> T nativeRead(String json, TypeReference<T> type) {
        try {
            return mapper.readValue(json, type);
        } catch (IOException e) {
            logger.error("json解析出错：" + json, e);
            return null;
        }
    }
}
```
#### Service方法
```java
// 首先注入redis
@Autowired
private StringRedisTemplate stringRedisTemplate;

// 创建一个存放user信息的USER_KEY
private static final String USER_KEY = "USER_KEY";

// 查询用户信息
try {
            // 先从redis中获取用户
            // 判断是否有这个Key
            Boolean hasKey = this.stringRedisTemplate.hasKey(USER_KEY);

            // 若有，则直接从redis中获取数据
            if (hasKey) {

                // 获取数据转换为json数据
                String json = this.stringRedisTemplate.opsForValue().get(USER_KEY);

                // 将json数据转化为list对象
                List<User> users = JsonUtils.parseList(json, User.class);
                // 直接返回数据
                return users;
            }
           // 从数据库中查询数据信息（通过你的持久层）
           // 保存到Redis中
           // 将从数据库中查询的数据放入到redis中
                       // 将list集合转换成json对象
                       String json = JsonUtils.serialize(users);
           
                       // 保存到redis中
                       this.stringRedisTemplate.opsForValue().set(USER_KEY, json);
                        return users;
        } catch (Exception e) {

            e.printStackTrace();

            return null;
        }
```
Redis🉐️简单使用就到这里。想继续了解其中的数据类型需要你自己去研究，亲自操作印象更深刻，是不？

好了，本次的技术分享就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！
### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)
