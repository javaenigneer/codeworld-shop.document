## CodeWorld-Cloud-Shop 订单过期处理详解---奶妈级教学 {docsify-ignore}
>今天我们来讲一讲订单失效的处理方法
>我们在创建订单时，都会设置一个时间限制，我们该如何去处理订单失效的问题，怎么去自动关闭订单

### 前言
在学习之前相信大家对Redis有一定的了解了吧，并且也安装好了Redis环境
如果这些还没有安装好，那就去看看吧
[Redis的安装和使用](/environmental-installation/environmental-installation-redis.md)
```text
我们的需求是在创建订单的时候，用户可能不会去支付订单，那么我们应该怎么去自动关闭这个订单呢

相信可能会有人想到的是定时任务，我们将订单创建后加入到定时任务中，并设置时间，如果在这个时间还没有进行支付，那么我们就去关闭这个订单
如何使用这个定时任务呢，这个就需要你自己去实践了
```
- [CodeWorld-Cloud-Shop XXL-JOB入门详解篇---奶妈级教学](/technology/xxl-job-get-started.md)
- [CodeWorld-Cloud-Shop XXL-JOB动态创建任务详解篇1---奶妈级教学](/technology/xxl-job-customize.md)
- [CodeWorld-Cloud-Shop XXL-JOB动态创建任务详解篇2---奶妈级教学](/technology/xxl-job-customize-two.md)

那么我们这里要将的是Redis的过去监听，我们将创建的订单保存到redis中，并设置有效时间，如果有效时间已过，那么将此订单作为失效订单

### 直接来到codeworld-cloud-order模块吧
其他的我们就不用再多说了吧，要启动那些，这就我们都是小菜一碟了 
#### OrderController
既然是订单操作，那么我们就应该来到订单接口
直接看到这个订单接口
```java
 @PostMapping("create-order")
    @ApiOperation("创建订单")
    public FCResponse<String> createOrder(@RequestBody @Valid OrderAddRequest orderAddRequest){
        return this.orderService.createOrder(orderAddRequest);
    }
```
#### OrderServiceImpl
```java
        log.info("生成订单，订单编号：{}，用户id：{}", order.getId(), memberInfo.getMemberId());
        // 将订单信息保存到redis缓存中,时间为10分钟，10分钟过后将其置为失效订单
        // 将订单id作为Key，订单信息作为value
        Map<String, Object> map = new HashMap<>(2);
        map.put("orderId", order.getId());
        map.put("totalPay", order.getTotalPay());
        // 将map进行序列化
        String json = JsonUtils.serialize(map);
        assert json != null;
        this.stringRedisTemplate.opsForValue().set(String.valueOf(order.getId()), json, 60 * 10, TimeUnit.SECONDS);
        log.info("订单加入Redis缓存，订单编号：{}，用户id：{}，将在10分钟后过期", order.getId(), memberInfo.getMemberId());
```
这里就是将订单id和总共的钱保存到redis中，订单id作为key，不过这里设计不合理
那么我们怎么在监听这个呢

#### RedisMessageListener
```java
@Component
@Slf4j
public class RedisMessageListener extends KeyExpirationEventMessageListener {


    @Autowired(required = false)
    private StringRedisTemplate stringRedisTemplate;

    @Autowired(required = false)
    private OrderService orderService;

    public RedisMessageListener(RedisMessageListenerContainer listenerContainer){
        super(listenerContainer);
        log.info("Redis过期监听已启动");
    }

    @Override
    public void onMessage(Message message, byte[] pattern) {
        // 获取Key
        String orderId = (String) this.stringRedisTemplate.getValueSerializer().deserialize(message.getBody());
        // 收到过期异步通知
        log.info("收到过期异步通知，Key：{}", orderId);
        // 将失效订单关闭
        assert orderId != null;
        this.orderService.autoCloseOrder(Long.parseLong(orderId));
    }

}
```
直接看到了这个类 `RedisMessageListener` 在重写的方法 `onMessage`中看见获取key的方法
```java
this.stringRedisTemplate.getValueSerializer().deserialize(message.getBody());
```
我们这里就将这个key获取出来，这个key就是我们的订单id，我们拿到这个订单id去自动关闭这个失效订单

#### 自动关闭失效订单
```java
/**
     * 自动关闭失效订单
     *
     * @param orderId
     */
    @Override
    @Transactional
    public void autoCloseOrder(Long orderId) {

        if (ObjectUtil.isEmpty(orderId) || orderId <= 0) {
            log.error("订单号无效：{}", orderId);
        }
        // 根据订单号查询订单是否存在,且属于未付款订单
        OrderStatus orderStatus = this.orderStatusMapper.getOrderStatusByOrderId(orderId);
        // 订单不存在
        if (ObjectUtil.isEmpty(orderStatus)) {
            log.error("订单不存在：{}", orderId);
        }
        // 订单状态不正确,只有状态为1才能自动取消订单
        if (orderStatus.getOrderStatus() != 1) {
            log.error("订单状态错误，{}", orderId);
        }
        // 执行修改订单状态
        orderStatus.setCloseTime(new Date());
        orderStatus.setOrderId(orderId);
        orderStatus.setOrderStatus(7);
        this.orderStatusMapper.updateOrderStatus(orderStatus);
        log.error("订单已关闭，订单号：{}", orderId);
    }
```
这里也不难理解吧，我们要判断订单的是否存在和订单状态是否为未支付状态，否则不能将其关闭

```text
我们在这里不是主要讲怎么关闭订单，而是学习redis的过期监听的使用，怎么灵活运用到我们的项目中来
```
好了，本次的技术解析就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！！
### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)
