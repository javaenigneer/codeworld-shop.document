## CodeWorld-Cloud-Shop 系统后台登录详解篇---奶妈级教学 {docsify-ignore}
> 本篇文章我们主要讲述我们在登录时怎么获取我们的权限信息，怎么实现登录认证

相信在学习这篇文章时，你已经装好了全部的环境
* [CodeWorld-Cloud-Shop Redis的安装和使用](environmental-installation/environmental-installation-redis.md)
* [CodeWorld-Cloud-Shop RabbitMQ的安装和使用](environmental-installation/environmental-installation-rabbitmq.md)
* [CodeWorld-Cloud-Shop ElasticSearch的安装和使用](environmental-installation/environmental-installation-elasticsearch.md)
### POM文件
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

        <!--Jwt-->
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.4.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.4.0</version>
        </dependency>

        <!-- Jwt-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
        <!-- lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <!-- Hibernate校验 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.1.5.Final</version>
        </dependency>

        <dependency>
            <groupId>org.hibernate.common</groupId>
            <artifactId>hibernate-commons-annotations</artifactId>
            <version>5.0.1.Final</version>
        </dependency>

        <!-- Hutool工具类 -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>4.6.2</version>
        </dependency>
        <dependency>
            <groupId>com.codeworld.fc</groupId>
            <artifactId>codeworld-cloud-common</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
```
### application.yml文件
```java
server:
  port: 8004 # 端口
spring:
  application:
    name: codeworld-cloud-auth # 服务名称
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 # nacos 服务注册地址
        register-enabled: true
        weight: 1
  main:
    allow-bean-definition-overriding: true # 允许覆盖重名的bean
codeworld:  # jwt密钥信息
  jwt:
    secret: codeworld@Login(Auth}*^31)&codeworld% # 用于生存rsa公钥和私钥的密文,越复杂越好
    pubKeyPath: D:\\temp\\rsa\\rsa.pub # 公钥地址
    priKeyPath: D:\\temp\\rsa\\rsa.pri # 私钥地址
    expire: 30   # 过期时间,单位分钟
    cookieMaxAge: 180000
    cookieName: token
```

### 首先来看我们common模块里的工具类
> JwtConstans 主要是用来作为设置用户信息

```java
public abstract class JwtConstans {
    public static final String JWT_KEY_ID = "id";
    public static final String JWT_KEY_PHONE = "phone";
    public static final String JWT_KEY_RESOURCES = "resources";
}
```

> JwtUtils 生成token信息，并从token中获取用户信息,使用公钥解析

```java
/**
     * 私钥加密token
     *
     * @param loginInfoData      载荷中的数据
     * @param privateKey    私钥
     * @param expireMinutes 过期时间，单位秒
     * @return
     * @throws Exception
     */
    public static String generateToken(LoginInfoData loginInfoData, PrivateKey privateKey, int expireMinutes) throws Exception {
        return Jwts.builder()
                .claim(JwtConstans.JWT_KEY_ID, loginInfoData.getId())
                .claim(JwtConstans.JWT_KEY_PHONE, loginInfoData.getPhone())
                .claim(JwtConstans.JWT_KEY_RESOURCES, loginInfoData.getResources())
                .setExpiration(DateTime.now().plusMinutes(expireMinutes).toDate())
                .signWith(SignatureAlgorithm.RS256, privateKey)
                .compact();
    }

 /**
     * 公钥解析token
     *
     * @param token     用户请求中的token
     * @param publicKey 公钥
     * @return
     * @throws Exception
     */
    private static Jws<Claims> parserToken(String token, PublicKey publicKey) {
        return Jwts.parser().setSigningKey(publicKey).parseClaimsJws(token);
    }
```
> LoingInfoData 来存放用户的基本信息
```java
@Data
public class LoginInfoData {

    /**
     * id
     */
    private Long id;

    /**
     * 手机号
     */
    private String phone;

    /**
     * 对象数据
     */
    private String resources;

    public LoginInfoData(Long id, String phone, String resources) {
        this.id = id;
        this.phone = phone;
        this.resources = resources;
    }

    public LoginInfoData() {
    }

    public LoginInfoData(String resources) {
        this.resources = resources;
    }
}
```
那么这样我们的登录前缀就已经好了
用户通过手机号和密码来实现登录
那么我们就可以通过手机和密码来查询用户信息并通过用户的id和手机号来设置token信息
id、phone ---> token

### 我们直接从我们的Controller进入
```java
   @PostMapping("/web/system/system-login")
       @ApiOperation("系统后台登录web端")
       @PassToken
       public FCResponse<Map<String, Object>> systemLogin(@RequestBody @Valid SystemLoginRequest systemLoginRequest,
                                                          HttpServletRequest request,
                                                          HttpServletResponse response){
           return this.authService.systemLogin(systemLoginRequest,request,response);
       }
```
```text
相信这里面的注解应该都知道什么意思吧（有部分注解会单独讲解）
@PostMapping ---> post请求
@ApiOperation---> Swagger的注解，操作接口名称
@RequestBody---> 前端请求发送Json数据，后端使用它来接收
@Valid---> Hiberate的一个信息校验
```
那么到了这一步我们就基本了解接口的作用啦
### Service方法
```java
/**
     * 系统后台登录
     * @param systemLoginRequest
     * @param request
     * @param response
     * @return
     */
    FCResponse<Map<String, Object>> systemLogin(SystemLoginRequest systemLoginRequest, HttpServletRequest request, HttpServletResponse response);
```
这个没什么讲解
### ServiceImpl方法
```java
        // 根据用户名查询用户（这里也是通过远程调用来实现）
        FCResponse<User> userFCResponse = this.userClient.getUserByName(systemLoginRequest.getUsername());
        if (!userFCResponse.getCode().equals(HttpFcStatus.DATASUCCESSGET.getCode())) {
            return FCResponse.dataResponse(HttpFcStatus.AUTHFAILCODE.getCode(), userFCResponse.getMsg(), null);
        }
        User user = userFCResponse.getData();
        if (user.getUserStatus().equals(StatusEnum.USER_DISABLE)) {
            return FCResponse.dataResponse(HttpFcStatus.AUTHFAILCODE.getCode(), HttpMsg.user.USER_DISABLE.getMsg(), null);
        }
        // 校验密码
        if (!StringUtils.equals("123456", systemLoginRequest.getPassword())) {
            return FCResponse.dataResponse(HttpFcStatus.AUTHFAILCODE.getCode(), HttpMsg.user.USER_MESSAGE_ERROR.getMsg(), null);
        }
        // 执行登录
        UserInfo userInfo = new UserInfo();
        userInfo.setUserId(user.getUserId());
        userInfo.setPhone(user.getUserPhone());
        LoginInfoData loginInfoData = new LoginInfoData();
        loginInfoData.setId(userInfo.getUserId());
        loginInfoData.setPhone(userInfo.getPhone());
        // 设置为系统标识
        loginInfoData.setResources("system");
        try {
            String token = JwtUtils.generateToken(loginInfoData, this.jwtProperties.getPrivateKey(), this.jwtProperties.getExpire());
            CookieUtils.setCookie(request, response, jwtProperties.getCookieName(), token, jwtProperties.getCookieMaxAge() * 60);
            Map<String, Object> map = new HashMap<>();
            map.put("token", token);
            return FCResponse.dataResponse(HttpFcStatus.DATASUCCESSGET.getCode(), HttpMsg.user.USER_LOGIN_SUCCESS.getMsg(), map);
        } catch (Exception e) {
            e.printStackTrace();
            throw new FCException("系统错误");
        }
```
```java
这里我们是用了openFeign远程调用（后面讲解）
1.FCResponse<User> userFCResponse = this.userClient.getUserByName(systemLoginRequest.getUsername());
2.校验密码
3.校验通过，设置登录信息LoginInfoData
4.调用生成token的工具类（上面我们已经描述）
5.返回一个Token信息
6.将其设置到cookie中，并设置名字为token，并设置失效时间
```
在这里我们还要用到这个JwtProperties类
```java
/**
 * ClassName JwtProperties
 * Description 保存此授权中心微服务的公钥和私钥以及全局配置
 * Author Lenovo
 * Date 2020/12/23
 * Version 1.0
**/
@Component
@ConfigurationProperties(prefix = "codeworld.jwt")
@Data
public class JwtProperties {

    private String secret; // 用于生存rsa公钥和私钥的密文,越复杂越好

    private String pubKeyPath;// 公钥
    private String priKeyPath;// 私钥

    private int expire;// token过期时间

    private PublicKey publicKey; // 公钥
    private PrivateKey privateKey; // 私钥

    private Integer cookieMaxAge;
    private String cookieName;

    private static final Logger logger = LoggerFactory.getLogger(JwtProperties.class);

    /**
     * 在构造函数后(即从配置文件获取到属性后)读取私钥、公钥
     * 构造函数 -> 自动注入@Autowired -> @PostConstruct -> InitializingBean -> xml中配置的init-method="init"方法
     */
    @PostConstruct
    public void init(){
        try {
            File pubKey = new File(pubKeyPath);
            File priKey = new File(priKeyPath);
            //若不存在公钥和私钥
            if (!pubKey.exists() || !priKey.exists()) {
                // 生成公钥和私钥
                RsaUtils.generateKey(pubKeyPath, priKeyPath, secret);
            }
            // 从文件读取公钥和私钥
            this.publicKey = RsaUtils.getPublicKey(pubKeyPath);
            this.privateKey = RsaUtils.getPrivateKey(priKeyPath);
        } catch (Exception e) {
            logger.error("初始化公钥和私钥失败！", e);
            throw new RuntimeException();
        }
    }
}
```
```text
使用@Component注解加入到组件中，那么我们就可以在类中注解使用@Autowired
@ConfigurationProperties这个注解可以从我们的application.yml配置文件中获取信息,这里我们是以codeworld.jwt开头下的属性值
并且名称要和配置文件中的属性名一致
使用init()方法来初始化一个公钥和密钥
```
### 实践操作
#### 首先要启动我们的nacos（我是安装在window上的，推荐安装在Linux上）
启动能看到这个界面就启动成功了
![windows-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/nacos/windows-start.png)
这样我们的nacos注册中心就准备好了，现在就只需要启动我们的项目就可以
不过也不是全部都要启动，看需要什么就启动什么，这样电脑可能才跑的动
如果你的电脑横nb，当我没说
#### 启动项目里面的网关模块(codeworld-cloud-gateway)
启动成功看见如下信息，这样就代表网关启动成功，并成功的注入到了nacos注册中心中
![gateway-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-gateway/gateway-start.png)
#### 启动项目里面的认证中心模块(codeworld-cloud-auth)
启动成功看见如下信息，这样就代表认证中心启动成功，并成功的注入到了nacos注册中心中
![auth-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-auth/auth-start.png)
#### 启动项目里面的系统模块(codeworld-cloud-system)
启动成功看见如下信息，这样就代表系统启动成功，并成功的注入到了nacos注册中心中
![system-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-system/system-start.png)
#### 启动项目里面的商户模块(codeworld-cloud-merchant)
启动成功看见如下信息，这样就代表商户启动成功，并成功的注入到了nacos注册中心中
![merchant-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-merchant/merchant-start.png)
这样我们的项目就启动完成了
接下来打开我们的postman接口调试工具
#### 调用我们的系统登录接口（测试系统用户登录）
![system-login-user](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-auth/system-login-user.png)
这里呢，使用的是json数据，username:1111,password:123456
发送请求
接着就会看见如下信息,就可以看见返回的token信息，那么这个token信息是用来交换用户信息的，这样就认证成功了
![system-login-user-success](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-auth/system-login-user-success.png)
```text
那么到这里我们的用户认证信息就完成了，不过还没有给用户授权，那么为什么这样做登录呢？
这样做的是我们我们可以通过不同的登录标识来对应回去用户的权限信息。
比如：商户可以添加商品，而其他的管理员就不可以
```
那么怎么才能给用户授权呢？请看对应章节用户授权[CodeWorld-Cloud-Shop 系统授权详解篇](system-authorization.md)
好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！！

### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
