### CodeWorld-Cloud-Shop 短信服务之发送短信详解篇---奶妈级教学 {docsify-ignore}
>我们今天来讲我们在注册或登录时我可以通过发送短信的功能来达到我们想要的目的

### 首先来到我们的codeworld-cloud-sms模块
#### POM文件
这里一些基本的pom文件就不用说了吧
这里我们需要加入redis的使用,你难道还不会基本的redis吗？那就快去学习吧[Redis的安装和使用](../environmental-installation/environmental-installation-redis.md)
```java
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <version>2.3.4.RELEASE</version>
        </dependency>
```
我们这里加入Redis的目的是什么呢？
我们将我们获取到的验证码保存到Redis缓存中，这样做的好处是什么呢？
- redis缓存运行效率高
- redis可以通过expire来设定过期策略，比较适用于验证码场景
- 考虑到分布式数据个负载均衡数据要一致，这种共有的不用持久化的数据最好找一个缓存服务器存储
- 等等等等。。。。。。

那么话不多少直接上代码
#### application.yml，加入以下配置
```java
redis:
    host: 192.168.2.4
    database: 4
```
#### controller
```java
@RestController
@RequestMapping("codeworld-sms")
@Api(tags = "短信接口管理")
public class SmsController {

    @Autowired(required = false)
    private SmsService smsService;

    @PostMapping("send-sms")
    @ApiOperation("发送短信")
    public FCResponse<String> sendSms(@RequestParam("phone") String phone){
        return this.smsService.sendSms(phone);
    }
}
```
#### service 
```java
 /**
     * 发送短信
     * @param phone
     * @return
     */
    FCResponse<String> sendSms(String phone);
```
#### serviceImpl
```java
@Service
@Slf4j
public class SmsServiceImpl implements SmsService {

    @Autowired(required = false)
    private StringRedisTemplate stringRedisTemplate;

    private final String PHONE_CODE = "PHONE_CODE:";

    /**
     * 发送短信
     *
     * @param phone
     * @return
     */
    public FCResponse<String> sendSms(String phone) {
        if (!StringUtil.checkPhone(phone)){
            return FCResponse.dataResponse(HttpFcStatus.PARAMSERROR.getCode(), HttpMsg.sms.SMS_PHONE_ERROR.getMsg());
        }
        // 默认验证码
        String code = "888888";
        // 将验证码保存到Redis中,两分钟失效
        this.stringRedisTemplate.opsForValue().set(PHONE_CODE + phone, code, 60 * 2, TimeUnit.SECONDS);
        log.info("短信发送成功，手机号为：{}",phone);
        return FCResponse.dataResponse(HttpFcStatus.DATASUCCESSGET.getCode(),HttpMsg.sms.SMS_PHONE_SUCCESS.getMsg(),code);
    }
}
```
```text
这里就有点尴尬了，并没有真正实现短信发送。短信发送功能要money，哎
不过我们这样就将就是吧,也不影响我们使用，同样可以达到我们想要的功能，哈哈哈哈哈
1.首先我们去定义一个redis的缓存Key---》PHONE_CODE
2.判断手机号是否是一个合法的手机号
3.然后就将默认的验证码保存到redis缓存中
4.再返回给前端
这也太low吧，难受，难受
不过后面肯定会优化的
```
### 实践
#### 首先启动我们一系列的服务
- 启动nacos服务注册中心
- 启动codeworld-cloud-gateway 网关模块
- 启动codeworld-cloud-sms 短信模块服务
![server-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-sms/server-start.png)
#### 打开我们的PostMan工具
输入接口：http://localhost:8888/codeworld-sms/send-sms
添加参数，若输入一个格式错误的手机号或者不输入手机号
![send-message-error](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-sms/send-message-error.png)
然后输入一个正确的手机号

这样我们就在Redis的可视化工具中查看到了缓存信息，并设置了有效时间为2分钟

那么简单的短信发送功能就完成啦

好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！

### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)




