## CodeWorld-Cloud-Shop XXL-JOB动态创建任务详解篇2---奶妈级教学
### 前言
我们在上一节[CodeWorld-Cloud-Shop XXL-JOB动态创建任务详解篇1---奶妈级教学](/technology/xxl-job-customize-two.md)讲解了XXL-JOB我们在创建任务的缺陷
那么这一节我们呢将继续实现怎么来动态创建任务

### 修改xxl-job-admin的接口
我们知道了，我们在调用接口前呢，就需要登录才能调用，不过后来我们发现可以加上一个 `PermissionLimit`并设置limit为false
那么这样就不用去登录就可以调用接口

将 `JobInfoController`添加如下接口
```java
// 自定义方法
	/**
	 * 添加任务
	 * @param jobInfo
	 * @return
	 */
	@RequestMapping("/addJob")
	@ResponseBody
	@PermissionLimit(limit = false)
	public ReturnT<String> addJob(@RequestBody XxlJobInfo jobInfo) {
		return xxlJobService.add(jobInfo);
	}
```
```text
我们在上面可以看见大致和原来的添加接口一致，这里我们加上了
@PermissionLimit(limit = false) // 可以用登录直接放行
@RequestBody // 接收json类型的参数
```
这里我们只需要添加这一个接口就可以啦，里面实现的逻辑就和原来的一样，不做任何的修改
我们只需要提供需要的参数就可以啦
![xxl-job-add-job](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/xxl-job/xxl-job-add-job.png)

### 创建一个新的项目
我们这里实现的一个需求就是
我们创建一个User用户，在创建时间的基础上加上1分钟，在这个时间自动添加
例如：2020-01-01 12:00:00这个时间创建 那么我们应该在2020-01-01 12:00:01这个时间添加
#### POM文件
```java
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.xuxueli</groupId>
            <artifactId>xxl-job-core</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.0.6</version>
        </dependency>
```
#### application.yml文件
```java
server:
  port: 10001

xxl:
  job:
    accessToken:
    admin:
      addresses: http://localhost:9999/xxl-job-admin
    executor:
      address:
      appname: xxl-job-test
      ip:
      logpath: /data/applogs/xxl-job/jobhandler
      logretentiondays: 30
      port: 10010
```
这些配置我相信你再熟悉不过了吧，我们在第一节就看见过啦
那么既然配置文件有了，那么我们也要创建一个配置类
#### XxlJobConfig
>XxlJobConfig

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
}
```
#### 基本的工具类
>DateUtils 日期工具类

```java
/**
 * 日期工具类
 *
 * @author Lenovo
 */
public class DateUtils {

    /**
     * 日期格式yyyy-MM-dd
     */
    public static final String pattern_date = "yyyy-MM-dd";

    /**
     * 日期时间格式yyyy-MM-dd HH:mm:ss
     */
    public static final String pattern_time = "yyyy-MM-dd HH:mm:ss";

    /**
     * 描述：日期格式化
     *
     * @param date 日期
     * @return 输出格式为 yyyy-MM-dd 的字串
     */
    public static String formatDate(Date date) {

        return formatDate(date, pattern_time);

    }

    /**
     * 描述：日期格式化
     *
     * @param date    日期
     * @param pattern 格式化类型
     * @return
     */
    public static String formatDate(Date date, String pattern) {

        SimpleDateFormat dateFormat = new SimpleDateFormat(pattern);

        return dateFormat.format(date);
    }

    /***
     * convert Date to cron ,eg.  "0 07 10 15 1 ? 2016"
     * @param date  : 时间点
     * @return
     */
    public static String getCron(Date date) {
        String dateFormat = "ss mm HH dd MM ? yyyy";
        return formatDate(date, dateFormat);
    }
}
```
>JsonUtils 工具类

```java
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
#### 创建Response响应信息
>ApiResponse

```java
@Data
public class FcResponse<T> {

    private Integer code;

    private String msg;

    private T data;

    public FcResponse(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```
#### 创建一个domain
>XxlJobInfo

```java
@Data
public class XxlJobInfo {

    private int id;				// 主键ID

    private int jobGroup;		// 执行器主键ID
    private String jobDesc;

    private Date addTime;
    private Date updateTime;

    private String author;		// 负责人
    private String alarmEmail;	// 报警邮件

    private String scheduleType;			// 调度类型
    private String scheduleConf;			// 调度配置，值含义取决于调度类型
    private String misfireStrategy;			// 调度过期策略

    private String executorRouteStrategy;	// 执行器路由策略
    private String executorHandler;		    // 执行器，任务Handler名称
    private String executorParam;		    // 执行器，任务参数
    private String executorBlockStrategy;	// 阻塞处理策略
    private int executorTimeout;     		// 任务执行超时时间，单位秒
    private int executorFailRetryCount;		// 失败重试次数

    private String glueType;		// GLUE类型	#com.xxl.job.core.glue.GlueTypeEnum
    private String glueSource;		// GLUE源代码
    private String glueRemark;		// GLUE备注
    private Date glueUpdatetime;	// GLUE更新时间

    private String childJobId;		// 子任务ID，多个逗号分隔

    private int triggerStatus;		// 调度状态：0-停止，1-运行
    private long triggerLastTime;	// 上次调度时间
    private long triggerNextTime;	// 下次调度时间
}
```
那么这个就是我们需要传入的参数
#### XxlUtil工具类
>XxlUtil

```java
@Component
public class XxlUtil {

    /**
     * 地址
     */
    private final String adminAddress = "http://localhost:9999/xxl-job-admin";

    @Autowired(required = false)
    private final RestTemplate restTemplate = new RestTemplate();

    // 请求Url
    private static final String ADD_INFO_URL = "/jobinfo/addJob";
    private static final String UPDATE_INFO_URL = "/jobinfo/updateJob";
    private static final String REMOVE_INFO_URL = "/jobinfo/removeJob";
    private static final String GET_GROUP_ID = "/jobgroup/loadByAppName";

    /**
     * 添加任务
     * @param xxlJobInfo
     * @param appName
     * @return
     */
    public String addJob(XxlJobInfo xxlJobInfo, String appName){
        Map<String, Object> params = new HashMap<>();
        params.put("appName",appName);
        String json = JsonUtils.serialize(params);
        String result = doPost(adminAddress + GET_GROUP_ID, json);
        JSONObject jsonObject = JSON.parseObject(result);
        Map<String, Object> map  = (Map<String, Object>) jsonObject.get("content");
        Integer groupId = (Integer) map.get("id");
        xxlJobInfo.setJobGroup(groupId);
        String json2 = JSONObject.toJSONString(xxlJobInfo);
        return doPost(adminAddress + ADD_INFO_URL, json2);
    }

    /**
     * 远程调用
     * @param url
     * @param json
     */
    private String doPost(String url, String json) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<String> entity = new HttpEntity<>(json,headers);
        ResponseEntity<String> responseEntity = restTemplate.postForEntity(url, entity, String.class);
        return responseEntity.getBody();
    }
}
```
这里对代码进行讲解下
```text
 /**
     * 远程调用
     * @param url
     * @param json
     */
    private String doPost(String url, String json) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<String> entity = new HttpEntity<>(json,headers);
        ResponseEntity<String> responseEntity = restTemplate.postForEntity(url, entity, String.class);
        return responseEntity.getBody();

// 这个就是我们进行远程调用的方法，我们一个是请求的url，第二个就是参数
然后调用返回主体信息
```
```text
       Map<String, Object> params = new HashMap<>();
        params.put("appName",appName);
        String json = JsonUtils.serialize(params);
        String result = doPost(adminAddress + GET_GROUP_ID, json);
        JSONObject jsonObject = JSON.parseObject(result);
        Map<String, Object> map  = (Map<String, Object>) jsonObject.get("content");
        Integer groupId = (Integer) map.get("id");
        xxlJobInfo.setJobGroup(groupId);
        String json2 = JSONObject.toJSONString(xxlJobInfo);
        return doPost(adminAddress + ADD_INFO_URL, json2);

其中有一个appName，何为appName--执行器的名称，也就是我们的任务是在哪一个执行器下执行的
首先呢我们需要通过执行器的名称来查询执行器的id，对应数据库中的xxl_job_group中的id字段
我们在xxl_job_info这个表中也可以清楚的看见有个job_group这个字段
这样我们将其设置到xxlJobInfo中，然后进行远程调用我们的添加接口，这样就实现了远程调用
```
在上面我们提到根据执行器名称来查询执行器id，那么这个接口在哪里呢，其实和任务添加的一样，我们也需要去开发一个新的接口
#### 修改xxl-job-admin中的JobGroupController
添加如下接口
```java
@RequestMapping("/loadByAppName")
	@ResponseBody
	@PermissionLimit(limit = false)
	public ReturnT<XxlJobGroup> loadByAppName(@RequestBody Map<String,Object> map){
		XxlJobGroup jobGroup = xxlJobGroupDao.loadByAppName(map);
		return jobGroup!=null?new ReturnT<XxlJobGroup>(jobGroup):new ReturnT<XxlJobGroup>(ReturnT.FAIL_CODE, null);
	}
```
#### 继续创建dao下的方法
```java
XxlJobGroup loadByAppName(Map<String, Object> map);
```
#### 创建查询语句
```java
<select id="loadByAppName" parameterType="hashmap" resultMap="XxlJobGroup">
		SELECT <include refid="Base_Column_List" />
		FROM xxl_job_group AS t
		WHERE t.app_name = #{appName}
	</select>
```
这样我们的执行器id就查询出来了

好了，既然我们的远程调用接口写好了，我们将来写本地的接口

#### 首先创建UserController
>UserController

```java
@RestController
@RequestMapping("codeworld-xxl")
public class XxlController {

    @Autowired(required = false)
    private XxlService xxlService;

    /**
     * 添加任务
     * @return
     */
    @PostMapping("add-job")
    public FcResponse<Void> addJob(){
        return this.xxlService.addJob();
    }
}
```
#### 接着UserService
>UserService

```java
public interface XxlService {
    /**
     * 添加任务
     * @return
     */
    FcResponse<Void> addJob();
}
```
#### 再接着UserServiceImpl
>UserServiceImpl

```java
@Service
@Slf4j
public class XxlServiceImpl implements XxlService {

    @Autowired(required = false)
    private XxlUtil xxlUtil;

    /**
     * 添加任务
     *
     * @return
     */
    @Override
    public FcResponse<Void> addJob() {
        User user = new User();
        user.setId(1000L);
        user.setUserName("code");
        user.setCreateTime(new Date());
        XxlJobInfo xxlJobInfo = new XxlJobInfo();
        xxlJobInfo.setJobDesc("测试用户定时添加");
        xxlJobInfo.setAuthor("admin");
        xxlJobInfo.setScheduleType("CRON");
        Date startTime = new Date(user.getCreateTime().getTime() + 60000);
        xxlJobInfo.setScheduleConf(DateUtils.getCron(startTime));
        xxlJobInfo.setGlueType("BEAN");
        xxlJobInfo.setExecutorHandler("addUser");
        xxlJobInfo.setExecutorParam(JsonUtils.serialize(user));
        xxlJobInfo.setExecutorRouteStrategy("FIRST");
        xxlJobInfo.setMisfireStrategy("DO_NOTHING");
        xxlJobInfo.setExecutorBlockStrategy("SERIAL_EXECUTION");
        xxlJobInfo.setTriggerStatus(1);
        xxlUtil.addJob(xxlJobInfo, "xxl-job-test");
        log.info("任务已添加，将在{}开始执行任务",DateUtil.date(startTime));
        return new FcResponse<>(1,"任务添加成功",null);
    }
}
```
这里呢我们就直接指定创建用户了，重点不是用户而是我们的XxlJobInfo
```text
        xxlJobInfo.setJobDesc("测试用户定时添加"); // 描述信息
        xxlJobInfo.setAuthor("admin"); // 执行人
        xxlJobInfo.setScheduleType("CRON"); // 执行类型 采用CRON表达式
        Date startTime = new Date(user.getCreateTime().getTime() + 60000); // 在创建时间上加上1分钟
        xxlJobInfo.setScheduleConf(DateUtils.getCron(startTime)); // 设置cron
        xxlJobInfo.setGlueType("BEAN"); // 设置GLUE类型为BEAN模式
        xxlJobInfo.setExecutorHandler("addUser"); // 设置任务Handler名称
        xxlJobInfo.setExecutorParam(JsonUtils.serialize(user)); // 设置参数
        xxlJobInfo.setExecutorRouteStrategy("FIRST"); // 执行器路由策略
        xxlJobInfo.setMisfireStrategy("DO_NOTHING"); // 调度过期策略
        xxlJobInfo.setExecutorBlockStrategy("SERIAL_EXECUTION"); // 阻塞处理策略
        xxlJobInfo.setTriggerStatus(1); // 调度状态：0-停止，1-运行
        xxlUtil.addJob(xxlJobInfo, "xxl-job-test"); // 调用工具类方法，xxl-job-test为执行器名称
```
那么到这里我们接口写完了
那么我们应该怎么执行任务呢
#### UserTask
>UserTask

```java
@Component
@Slf4j
public class UserTask {

    @XxlJob(value = "addUser")
    public ReturnT<String> addUser(String param){
        log.info("开始执行任务,在指定时间添加用户，收到参数{}，当前时间{}",param, DateUtil.date(new Date()));
        return ReturnT.SUCCESS;
    }
}
```
这个就是我们执行任务的方法，可以看见我们在这个@XxxlJob(value="addUser")和参数里的xxlJobInfo.setExecutorHandler("addUser"); 是一一对应的

好了到这里我么的全部详解就说明完了

不过说了这么多，到底行不行呢？
### 实践
#### 首先先启动xxl-job-admin这个项目
![xxl-job-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/xxl-job/xxl-job-start.png)
#### 启动本地项目
![xxl-job-customize](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/xxl-job/xxl-job-customize.png)
看见这个就是xxl加载成功啦
#### 我们需要去创建一个执行器，名称是xxl-job-test
这个就不用上图片了吧，去[CodeWorld-Cloud-Shop XXL-JOB入门详解篇---奶妈级教学](/technology/xxl-job-get-started.md)
#### 调用接口:http://localhost:10001/codeworld-xxl/add-job
![add-job-customize](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/xxl-job/add-job-customize.png)
出现这个说明接口调用成功啦
再来看我们的控制台
![add-task-success](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/xxl-job/add-task-success.png)
说明我们的任务已经添加成功啦
然后等着1分钟后看看会不会执行任务
![add-task-result](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/xxl-job/add-task-result.png)

哇哦，看来执行成功了，那么我们就这样成功了。
那么轮播图的问题就解决了，动态创建任务实现实现了轮播图的上线和下线
[代码地址](https://github.com/javaenigneer/SpringBoot-demo)

好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！

### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
