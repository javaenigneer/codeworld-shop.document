## CodeWorld-Cloud-Shop XXL-JOB动态创建任务详解篇1---奶妈级教学
### 前言
我们在[CodeWorld-Cloud-Shop XXL-JOB入门详解篇](../technology/xxl-job-get-started.md)介绍了XXL-JOB的基本使用
如果你还没有进行入门的同学，就先来一段奶妈级教学吧，然后再进行下一阶段
### 问题探索
我们虽然在入门级教学中演示了XXL-JOB任务调度的基本使用，我们也体会到了XXL-JOB的优越性以及页面操作的简单性
这样我们可以在界面直接对任务进行启动，停止等等操作
这让我很是舒服，但是可怕的事情来了

我们有这样一个需求，就拿项目中的来说

在商城首页呢，我们有一个轮播图，那么这个轮播图是随时进行切换的，我们随时可能在一个指定的时间更新这个轮播图信息
那么问题来了，当我们在添加轮播图时会给他指定一个上线和下线的时间，可能会有很多的轮播图添加，那我们每添加一个轮播图就去XXL-JOB的
可视化界面添加一个任务吗？那这样我们的手还不得断？还有加入我要在2021-01-01:12:05:35这个时间上线轮播图，那么我们在去手写CRON表达
式来执行任务吗？那我觉得你是真的有耐性，还附带了一个下线时间，上线和下线的时间都不一样。

那么我们应该怎么解决呢？

### 问题解决
首先呢，遇事不要慌，消灭恐惧的做好办法就是面对恐惧，奥利给

首先我们来看
![add-task](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/2021/01/%E4%BB%BB%E5%8A%A1%E6%B7%BB%E5%8A%A0%E6%88%90%E5%8A%9F%E7%95%8C%E9%9D%A2.png)
我们在这里可以清楚的看见，这很明显就是一个表单嘛，那么既然是表单，那就好办了
有表单就有接口，那么来到我们xxl-job-admin的接口看看呢
![xxl-job-interface](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/xxl-job/xxl-job-interface.png)
嚯哟，原来一切的接口都在这里啊
还好没放弃，哭了哭了
那么我们一路追踪下去来到 添加任务这个接口的实现类方法
我们可以看见传入的参数就是这个XxlJobInfo
```java
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
```
这里面有很多的参数信息，都很清楚明了
那么我们想想，再想想
接口有，参数信息我们也知道，那么我们不就可以直接调用他这个接口，传入想要的参数信息不就ok吗？
那么我们就不用再去可视化界面进行操作，那不就可以了吗？

意思好像是这个意思，那么哦们应该怎么去操作呢？

还有XXL-JOB这个是在另外一个项目上，而我们的却在另外一个项目上，那么我们怎么才能去调用这个接口呢？

那么就是用到远程调用了，这里我们使用的是RestTemplate这个来实现调用
你可以考虑使用openFeign来实现[CodeWorld-Cloud-Shop openFeign技术---奶妈级教学]

### 问题重重
那这样的还有一个问题，首先我们要在界面上操作是吧，那么要进入界面的第一步是什么呢？
登录。。对就是登录
要登录上去了才会使用到我们这些接口，如果没有登录那么就会被拦截掉
![xxl-job-permission](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/xxl-job/xxl-job-permission.png)
我们这里很清楚的看见，这里设置了对接口使用了拦截器
那么这里面有一个很清楚的注解 `PermissionLimit`
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PermissionLimit {
	
	/**
	 * 登录拦截 (默认拦截)
	 */
	boolean limit() default true;

	/**
	 * 要求管理员权限
	 *
	 * @return
	 */
	boolean adminuser() default false;

}
```
有一个limit的属性，默认的是true，就是说如果设置limit为true的话就需要被拦截，需要登录才能调用
那么我们想到如果设置为false，是不是就可以不用登录调用了
是的，我们这里呢，就是这样做的
那么我们就需要重新写一个接口，然后在这个接口上加上这个注解并设置为false

那么到这里我们的问题就基本解决了
流程呢就是 
```text
1.在xxl-job-admin项目的原有基础上开发新的一个接口，加上 `PermissionLimit`并设置为false
2.使用RestTemplate进行远程调用，实现任务的添加
```
那么我们将在下一节继续讲解[CodeWorld-Cloud-Shop XXL-JOB动态创建任务详解篇2---奶妈级教学](../technology/xxl-job-customize-two.md)
好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！！
### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)

