## CodeWorld-Cloud-Shop 系统授权详解篇---奶妈级教学 {docsify-ignore}
> 本篇文章我们主要讲述我们在登录成功后怎么来给用户进行授权

相信你在学习这篇这篇文章时，你已经对登录认证有了全面的认识
没有的话，那你还要继续进行奶妈级教学
[CodeWorld-Cloud-Shop 系统登录详解篇](system-login.md)

### POM文件(和登录一样没变的，因为是在同一个模块里面的，还是上一下代码)
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
### application.yml(也是一样一样的)
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
### 其他这里就不用一一讲述了，都在同一个模块里面
### 这里我们直接从Controller进入
```java
 // 获取登录用户信息
    @PostMapping("get-system-login-info")
    @ApiOperation("获取登录用户信息")
    public FCResponse<SystemLoginInfoResponse> getSystemLoginInfo(@RequestBody @Valid TokenRequest tokenRequest,
                                                                  HttpServletRequest request,
                                                                  HttpServletResponse response){
        return this.authService.getSystemLoginInfo(tokenRequest,request,response);
    }
```
这里呢使用到的注解是一样的，不过少了一个@PassToken(后面讲解)
### Service方法
```java
/**
     * 获取登录用户信息
     * @param tokenRequest
     * @param request
     * @param response
     * @return
     */
    FCResponse<SystemLoginInfoResponse> getSystemLoginInfo(TokenRequest tokenRequest,HttpServletRequest request,
                                                           HttpServletResponse response);
```
这个也没啥说的
### ServiceImpl方法
```java
try {
            LoginInfoData loginInfoData = JwtUtils.getInfoFromToken(tokenRequest.getToken(), this.jwtProperties.getPublicKey());

            if (ObjectUtils.isEmpty(loginInfoData)) {
                throw new FCException("登录过期");
            }
            SystemLoginInfoResponse systemLoginInfoResponse = new SystemLoginInfoResponse();
            Set<String> roles = new HashSet<>();
            Set<MenuVO> menuVOS = new HashSet<>();
            Set<ButtonVO> buttonVOS = new HashSet<>();
            // 根据用户id查询角色信息
            FCResponse<Role> roleFcResponse = this.roleClient.getRoleByUserId(loginInfoData.getId());
            if (roleFcResponse.getCode().equals(HttpFcStatus.PARAMSERROR.getCode())) {
                return FCResponse.dataResponse(HttpFcStatus.PARAMSERROR.getCode(), roleFcResponse.getMsg());
            }
            roles.add(roleFcResponse.getData().getRoleCode());
            // 根据角色id查询角色权限
            FCResponse<List<Menu>> menuFcResponse = this.menuClient.getMenuByRoleId(roleFcResponse.getData().getRoleId());
            if (menuFcResponse.getCode().equals(HttpFcStatus.PARAMSERROR.getCode())) {
                return FCResponse.dataResponse(HttpFcStatus.PARAMSERROR.getCode(), menuFcResponse.getMsg());
            }
            if (ObjectUtils.isEmpty(menuFcResponse.getData())) {
                return FCResponse.dataResponse(HttpFcStatus.DATASUCCESSGET.getCode(), menuFcResponse.getMsg());
            }
            List<Menu> menus = menuFcResponse.getData();
            // 菜单不为空
            if (!CollectionUtils.isEmpty(menus)) {
                menus.stream().filter(Objects::nonNull).forEach(menu -> {
                    // 按钮权限
                    if (StringUtils.equalsIgnoreCase("button", menu.getType())) {
                        // 添加到按钮权限
                        ButtonVO buttonVO = new ButtonVO();
                        BeanUtil.copyProperties(menu, buttonVO);
                        buttonVOS.add(buttonVO);
                    }
                    // 菜单权限
                    if (StringUtils.equalsIgnoreCase("menu", menu.getType())) {
                        // 添加到菜单权限
                        MenuVO menuVO = new MenuVO();
                        BeanUtil.copyProperties(menu, menuVO);
                        menuVOS.add(menuVO);
                    }
                });
            }
            systemLoginInfoResponse.getRoles().addAll(roles);
            systemLoginInfoResponse.getButtons().addAll(buttonVOS);
            systemLoginInfoResponse.getMenus().addAll(TreeBuilder.buildTree(menuVOS));
            systemLoginInfoResponse.setLoginIp(IPUtil.getIpAddr(request));
            systemLoginInfoResponse.setLoginLocation(AddressUtil.getCityInfo(IPUtil.getIpAddr(request)));
            systemLoginInfoResponse.setLoginTime(new Date());
            return FCResponse.dataResponse(HttpFcStatus.DATASUCCESSGET.getCode(), HttpMsg.user.USER_AUTH_SUCCESS.getMsg(), systemLoginInfoResponse);
        } catch (Exception e) {
            e.printStackTrace();
            throw new FCException("系统错误");
        }
```
上面的代码有点多，我们一步一步的来
在完成这一步时，你需要去了解数据库之间的关系[CodeWorld-Cloud-Shop数据库基本概览](../database/basic-overview.md)
### 代码详解
- 首先我们知道我们用户登录后是保存起来的，我们需要通过我们的token来交换用户信息
- 那么我们直接通过这个jwt工具类来实现获取用户信息(JwtUtils.getInfoFromToken)
- 判断是否为空
- 这几行代码
  * SystemLoginInfoResponse systemLoginInfoResponse = new SystemLoginInfoResponse(); // 这个是用来保存用户认证成功后的所有信息
  * Set<String> roles = new HashSet<>(); // 这个是用来保存用户的角色信息
  * Set<MenuVO> menuVOS = new HashSet<>(); // 这个是用来保存用户拥有菜单信息
  * Set<ButtonVO> buttonVOS = new HashSet<>(); // 这个是用来保存用户拥有按钮信息
- 首先也使用通过我们的OpenFeign来进行远程调用
```java
// 根据用户id查询角色信息
FCResponse<Role> roleFcResponse = this.roleClient.getRoleByUserId(loginInfoData.getId());
```
- 那么我们继续往下追踪，来到我们的roleClient，远程调用的是codeworld-cloud-system这个服务
![get-role-user-id](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-auth/get-role-user-id.png)
- 来到我们的这个codeworld-cloud-system这个服务
![get-role-user-id](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-system/get-role-user-id.png)
这样就请求到了我们的这个服务，并完成调用
- 那这样我们通过远程调用根据用户的id来获取到了用户角色信息
- 这一步是通过角色id获取菜单权限信息，也是通过远程调用openFeign
```java
 // 根据角色id查询角色权限
FCResponse<List<Menu>> menuFcResponse = this.menuClient.getMenuByRoleId(roleFcResponse.getData().getRoleId());
```
- 这样一步一步的下去我们就来到了
```java
// 菜单不为空
            if (!CollectionUtils.isEmpty(menus)) {
                menus.stream().filter(Objects::nonNull).forEach(menu -> {
                    // 按钮权限
                    if (StringUtils.equalsIgnoreCase("button", menu.getType())) {
                        // 添加到按钮权限
                        ButtonVO buttonVO = new ButtonVO();
                        BeanUtil.copyProperties(menu, buttonVO);
                        buttonVOS.add(buttonVO);
                    }
                    // 菜单权限
                    if (StringUtils.equalsIgnoreCase("menu", menu.getType())) {
                        // 添加到菜单权限
                        MenuVO menuVO = new MenuVO();
                        BeanUtil.copyProperties(menu, menuVO);
                        menuVOS.add(menuVO);
                    }
                });
            }
```
- 相信这上面的代码就很一目了然的吧，通过一个循环来判断权限的类型，若是menu，就保存到menuVOS，若是button，就保存到buttonVOS
- 最终吧所有的数据信息保存到systemLoginInfoResponse里面，这样就完成了用户的授权
- 其实授权很简单，就是将用户拥有的角色，权限查询出来保存到信息里面

那么到了这里就用户授权到此结束了
### 那么就会有疑问？
不是我们系统分两种用户登录吗？一个商户，一个系统管理，那么我们是怎么给它区分的呢？
那么就要来到我们的商户注册功能看见里面的serviceImpl方法
![merchant-register](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-merchant/merchant-register.png)
![merchant-register-serviceImpl](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-merchant/merchant-register-serviceimpl.png)
我们这里是默认写死了，后面会进行优化
那这样我们就默认将商户设置为商户角色信息

### 实践操作
#### 首先同样要启动我们的那些模块(nacos、gateway、auth、system、merchant)
#### 首先来测试我们系统管理员授权
这里如果我们直接拿上一次获得的token，可能已经过期了，需要我们重新登录
这里我们需要设置两个token信息,一个是在header里面，一个是我们请求的token
![get-system-login-info-user](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-auth/get-system-login-info-user.png)
最终会返回如下结果，可以看到roles：admin
![get-system-login-info-user-success](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-auth/get-system-login-info-user-success.png)
#### 首先来测试我们商户授权
![get-system-login-info-merchant](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-auth/get-system-login-info-merchant.png)
最终会返回如下结果，可以看到roles：merchant

好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！

### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)
