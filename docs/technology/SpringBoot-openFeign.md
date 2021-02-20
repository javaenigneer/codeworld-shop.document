## CodeWorld-Cloud-Shop openFeign技术---奶妈级教学 {docsify-ignore}
> 本篇文章主要带领大家来了解openFeign的世界，进行奶妈级的教学

### 什么是openFeign
```text
1.Feign 是一个声明式的 Web 服务客户端，让编写 Web 服务客户端变得非常容易，只需要创建一个接口并在接口上添加注解即可。Feign 是 Spring Cloud 组件中的一个轻量级 RESTful 的 HTTP 服务客户端 Feign 内置了 Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。
2.Feign 的使用方式是：使用 Feign 的注解定义接口，调用这个接口就可以调用服务注册中心的服务
```
### 为什么要使用openFeign？
```text
现在都是微服务时代，所谓什么是微服务，字面意思就是微小的服务，在以前我们的都是一个单体的服务，这样就很局限，如果其中有错，那么可能会整个服务都无崩掉，那就完蛋了
那么现在微服务来了，那就是将我们的单体服务拆分成一个一个的微小的服务，让他们你不影响我，我不影响你，就算你挂了但是我并没有挂，我还可以继续执行任务
现在我们应该怎么去建立连接呢？那么就是他openFeign，远程调用，你可以调用我的服务，我也可以调用你的服务
```
### 现在就开始带你入门openFeign
#### 我们还是将我们的服务注册到nacos注册中心,启动nacos
![nacos-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/windows-start.png)
#### IDEA创建项目
结构如下（这里只是演示了部分代码，具体代码还请前往GitHub）
![openFeign-struct](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/openFeign/openfeign-struct.png)
##### 首先创建一个父级项目(codeworld-openfeign)
添加POM文件
```java
 <dependencies>
        <!-- nacos 注册中心 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>0.2.2.RELEASE</version>
        </dependency>
        <!-- openFeign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.0.2.RELEASE</version>
        </dependency>
        <!-- web模块 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--hystrix依赖，主要是用  @HystrixCommand -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>
    </dependencies>

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
##### 依次创建codeworld-service-user、codeworld-service-role
这里主要是强调这两个模块里pom文件，我们需要将这里修改为
```java
<parent>
        <groupId>com.codeworld.fc</groupId>
        <artifactId>codeworld-openfeign</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```
我们是codeworld-openfeign下添加的项目，那么把codeworld-openfeign作为父级项目
那么到这里项目就创建好了
#### 来看我们的codeworld-service-user
##### application.yml
```java
server:
  port: 10000
spring:
  application:
    name: codeworld-service-user
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        register-enabled: true
        weight: 1
```
##### 创建一个domain的ApiResult
这个主要就是用来返回我们的请求到的数据信息

>ApiResult

```java
@Data
public class ApiResult<T>{

    private Integer code;

    private String msg;

    private T data;

    public ApiResult(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```
##### 创建User对象
>User

```java
@Data
public class User {

    private Integer id;

    private String name;

    private String roleName;

}
```
##### 创建UserController
>UserController

```java
@RestController
@RequestMapping("codeworld-user")
public class UserController {

    @Autowired(required = false)
    private UserService userService;

    /**
     * 根据Id获取用户
     * @param id
     * @return
     */
    @GetMapping("get-user-id")
    public ApiResult<User> getUserById(@RequestParam("id") Integer id){
        return this.userService.getUserById(id);
    }
}
```
##### 创建UserService
>UserService

```java
public interface UserService {

    ApiResult<User> getUserById(Integer id);
}
```

##### 创建UserServiceImpl
>UserServiceImpl

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired(required = false)
    private RoleClient roleClient;


    @Override
    public ApiResult<User> getUserById(Integer id) {
        if (ObjectUtils.isEmpty(id)) {
            return new ApiResult<>(0, "参数错误", null);
        }
        // 从数据中查询用户，通过用户id，这里为了方便就不演示数据库操作
        if (id == 10000) {
            User user = new User();
            user.setId(1000);
            user.setName("code");
            // 实现远程调用role服务
            ApiResult<Role> apiResult = this.roleClient.getRoleByUserId(1000);
            if (apiResult.getCode() == 0) {
                return new ApiResult<>(0, apiResult.getMsg(), null);
            }
            if (ObjectUtils.isEmpty(apiResult.getData())) {
                // 若没有查询成功或者参数错误
                // 默认设置用户角色为test
                user.setRoleName("test");
            } else {
                user.setRoleName(apiResult.getData().getName());
            }
            return new ApiResult<>(1,"查询成功",user);
        }
        return new ApiResult<>(0, "查询失败", null);
    }
}
```
##### 创建RoleClient
>RoleClient

```java
@FeignClient(value = "codeworld-service-role")
public interface RoleClient {

    /**
     * 根据用户id获取角色信息
     * @param userId
     * @return
     */
    @GetMapping("/codeworld-role/get-role-user-id")
    ApiResult<Role> getRoleByUserId(@RequestParam("userId") Integer userId);
}
```
这里我需要讲解一下，这里的这个接口就是我们要远程调用的服务接口，既然要调用这个接口，那么我们的被调用的那个服务就应该有这个接口
@FeignClient这个注解指定这个为远程调用类，那么里面的value指定的就是我们要调用的是哪一个服务名字

##### codeworld-service-user的启动类
我们需要加上这几个注解信息
```java
@SpringBootApplication // SpringBoot启动器
@EnableFeignClients // 启动远程调用
@EnableDiscoveryClient // nacos服务注册发现
```
那么这里我们的codeworld-service-user就基本完成了
#### 接下来看看codeworld-service-role
##### application.yml
````java
server:
  port: 10001
spring:
  application:
    name: codeworld-service-role
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        register-enabled: true
        weight: 1
````
##### 创建一个domain的ApiResult
这个主要就是用来返回我们的请求到的数据信息
>ApiResult

```java
@Data
public class ApiResult<T>{

    private Integer code;

    private String msg;

    private T data;

    public ApiResult(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```
##### 创建Role对象
>Role

```java
@Data
public class Role {

    private Integer id;

    private String name;
}
```
##### 创建RoleController
>RoleController

```java
@Autowired(required = false)
    private RoleService roleService;

    /**
     * 根据用户id获取角色信息
     * @param userId
     * @return
     */
    @GetMapping("get-role-user-id")
    public ApiResult<Role> getRoleByUserId(@RequestParam("userId") Integer userId){
        return this.roleService.getRoleByUserId(userId);
    }
```
##### 创建RoleService
>RoleService

```java
public interface RoleService {
    ApiResult<Role> getRoleByUserId(Integer userId);
}
```
##### 创建RoleServiceImpl
>RoleServiceImpl

```java
@Service
public class RoleServiceImpl implements RoleService {


    @Override
    public ApiResult<Role> getRoleByUserId(Integer userId) {
        if (ObjectUtils.isEmpty(userId)){
            return new ApiResult<>(0,"参数错误",null);
        }
        // 这里就不演示数据库查询，直接设定
        if (userId == 1000){
            Role role = new Role();
            role.setId(10000);
            role.setName("admin");
            return new ApiResult<>(1,"查询成功",role);
        }
        return new ApiResult<>(0,"无数据",null);
    }
}
```
到这里我们的codeworld-service-role就完成了，比user服务少了很多
##### codeworld-service-role的启动类
也要加上一个注解信息
@EnableDiscoveryClient // 开始注册发现

#### 验证结论
##### 调用接口(http://localhost:10000/codeworld-user/get-user-id)
![get-user-id](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/openFeign/get-user-id.png)
这样我们的远程调用就成功啦

不知道你有木有理解到，应该通俗易懂了，没理解那就再来一次奶妈级教学

不过我们还没完，就会产生一个问题，刚才不是还在说，如果一个服务挂了，就不会影响我们本身的服务吗
那如果关掉role服务那怎么办呢？
结果就出现了这种情况
![get-user-id-error](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/openFeign/get-user-id-error.png)
那这在搞什么啊，什么鬼东西，还不是一样挂
那么别急，我们来看看openFeign给我们提供的熔断机制(Hystix)

##### 那么要在我们的父级pom文件中引入
```java
 <!--hystrix依赖，主要是用  @HystrixCommand -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```
##### 修改我们codeworld-service-user中的application.yml文件
```java
server:
  port: 10000
spring:
  application:
    name: codeworld-service-user
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        register-enabled: true
        weight: 1
ribbon:
  ReadTimeout: 10000 #处理请求的超时时间，默认为1秒
  ConnectTimeout: 10000 #连接建立的超时时长，默认1秒
  MaxAutoRetries: 1 #同一台实例的最大重试次数，但是不包括首次调用，默认为1次
feign:
  hystrix:
    enabled: true
```
##### 然后在我们的启动类上加上
@EnableCircuitBreaker // 开启熔断机制
##### 创建一个RoleClient的实现类RoleClientFallback
>RoleClientFallback

```java
@Component
public class RoleClientFallback implements RoleClient {
    /**
     * 根据用户id获取角色信息
     *
     * @param userId
     * @return
     */
    @Override
    public ApiResult<Role> getRoleByUserId(Integer userId) {
        return new ApiResult<>(0,"请求超时",null);
    }
}
```
##### 修改RoleClient
```java
@FeignClient(value = "codeworld-service-role",fallback = RoleClientFallback.class)
public interface RoleClient {

    /**
     * 根据用户id获取角色信息
     * @param userId
     * @return
     */
    @GetMapping("/codeworld-role/get-role-user-id")
    ApiResult<Role> getRoleByUserId(@RequestParam("userId") Integer userId);
}
```
这里就需要加上fallback这个属性值，主要就是用来解决服务宕机调用的方法
那么这样我们再重新启动我们的user服务，不启动role服务
![get-user-id-success](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/openFeign/get-user-id-success.png)
这样我们的服务就不会报错了，这样就会提示我们请求超时，请求超时

那么这样就算我们的服务宕机了也不会影响
[代码地址](https://github.com/javaenigneer/SpringBoot-demo)
好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！

### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)



