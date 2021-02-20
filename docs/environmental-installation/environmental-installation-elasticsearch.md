## ElasticSearch的安装和使用 
### ElasticSearch安装（我们也是安装在Linux下的）
#### 安装Java环境（这个我相信你已经安装好了）
可以参考
[安装Java环境](https://www.cnblogs.com/renxixao/p/11469754.html)

#### 安装ElasticSearch-6.2.4
##### 安装并解压文件
```java
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.4.tar.gz
tar -zxvf elasticsearch-6.2.4.tar.gz -C /usr/local/
sh /usr/local/elasticsearch-6.2.4/bin/elasticsearch
```
启动后发现报错了

![启动ElasticSearch报错](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/elasticsearch/%E6%8A%A5%E9%94%99.png)

**从5.0开始 elasticsearch 安全级别提高了 不允许采用root帐号启动 所以我们要添加一个用户用来启动 elasticsearch**首先我们先把防火墙关闭
[防火墙使用](../firewall/firewall-use.md)

##### 执行以下命令

```java
useradd es // 创建es用户
chown -R es:es /usr/local/elasticsearch-6.2.4///把目录权限赋予给es用户
su es//切换至es用户
vi /usr/local/elasticsearch-6.2.4/config/elasticsearch.yml
```
##### 修改配置文件

把 host改为本机地址
![配置文件](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/elasticsearch/ElasticSearch.png)

**记得把前面注释#删掉 再执行 sh /usr/local/elasticsearch-6.2.4/bin/elasticsearch**
不过这样执行后一样会报错，那么就按照以下执行吧

##### 修改报错信息
**注意：以下操作都要切换到root下执行**

- max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
```java
// 修改/etc/security/limits.conf文件 在文件末尾添加如下
su root
vi /etc/security/limits.conf
// 添加以下内容
 * hard    nofile           65536
 * soft    nofile           65536
```
- max number of threads [3818] for user [es] is too low, increase to at least [4096]
```java
vi /etc/security/limits.d/20-nproc.conf
// 修改以下内容
 *            soft          nproc     4096
 *            hard          nproc     4096
root          soft             nproc     unlimited
```
- max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```java
vi  /etc/sysctl.conf
// 添加以下内容
vm.max_map_count = 2621441
// 执行命令立即生效
sudo sysctl -p /etc/sysctl.conf//立即生效
```
以上三个是常见的三个错误 其余的请自行百度

```
ulimit -a
```

![查看](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/elasticsearch/ulimit-a.png)

```text
发现当前最大线程数还是为3818  别慌 重启下虚拟机 重启后才能生效
```

![线程数](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/elasticsearch/ulimit-a%E7%BB%93%E6%9E%9C.png)

```java
// 接着切换到es用户启动
su es
sh/usr/local/elasticsearch-6.2.4/bin/elasticsearch -d     //加-d就是启动后台进程
ps -ef|grep elasticsearch
验证下服务是否正常运行 curl  http://192.168.88.133:9200
```
![结果](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/elasticsearch/curl.png)
##### 浏览器请求

```text
http://192.168.88.133:9200/
```

![结果](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/elasticsearch/result.png)

### 安装Kibana（可视化界面）

#### 下载kibana
`wget https://artifacts.elastic.co/downloads/kibana/kibana-6.2.4-linux-x86_64.tar.gz`

#### 解压kibana
`tar -zxvf kibana-6.2.4-linux-x86_64.tar.gz`

#### 进入到kibana
`cd kibana-6.2.4`

#### 修改配置文件
`vim config/kibana.yml`

```java
//放开注释,将默认配置改成如下：
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://192.168.88.133:9200"(虚拟机主机地址)
kibana.index: ".kibana"
```
#### 启动kibaana
```java
bin/kibana
```
![kibana](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/kibana/kibana.png)
### ElasticSearch的简单使用（根据商品id查询商品）
#### 引入Pom文件
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```
#### 修改application.yml文件
```java
  data:
    elasticsearch:
      cluster-name: elasticsearch
      cluster-nodes: 你的虚拟机主机地址(192.168.2.7):9300
```
#### controller层
```java
@PostMapping("get-product-id")
    @ApiOperation("根据id查询商品")
    @PassToken
    public FCResponse<ProductResponse> getProductById(@RequestParam("productId") Long productId){
        return this.goodsService.getProductById(productId);
    }
```
#### service层
```java
 /**
     * 根据商品id查询商品
     * @param productId
     * @return
     */
    FCResponse<ProductResponse> getProductById(Long productId);
```
#### serviceImpl层
```java
/**
     * 根据商品id查询商品
     *
     * @param productId
     * @return
     */
    @Override
    public FCResponse<ProductResponse> getProductById(Long productId) {
        if (ObjectUtil.isEmpty(productId) || productId <= 0) {
            return FCResponse.dataResponse(HttpFcStatus.PARAMSERROR.getCode(), HttpMsg.product.PRODUCT_ID_ERROR.getMsg());
        }
        // 自定义查询构建器
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        // 添加查询条件
        queryBuilder.withQuery(QueryBuilders.termQuery("productId", productId));
        // 获取结果集
        Page<SearchItem> search = this.searchRepository.search(queryBuilder.build());
        if (CollectionUtils.isEmpty(search.getContent())) {
            return FCResponse.dataResponse(HttpFcStatus.DATAEMPTY.getCode(), HttpMsg.product.PRODUCT_DATA_EMPTY.getMsg());
        }
        SearchItem searchItem = search.getContent().get(0);
        ProductResponse productResponse = this.buildProductResponse(searchItem);
        return FCResponse.dataResponse(HttpFcStatus.DATASUCCESSGET.getCode(), HttpMsg.product.PRODUCT_GET_SUCCESS.getMsg(), productResponse);
    }
```
#### 构建商品信息
```java
/**
     * 从索引库获取商品转化为productResponse
     *
     * @param searchItem
     * @return
     */
    private ProductResponse buildProductResponse(SearchItem searchItem) {
        ProductResponse productResponse = new ProductResponse();
        productResponse.setId(searchItem.getProductId());
        productResponse.setCategoryId(searchItem.getCategoryId());
        // 根据分类id查询分类信息
        FCResponse<Category> categoryResponse = this.categoryClient.getCategoryById(searchItem.getCategoryId());
        if (ObjectUtil.isEmpty(categoryResponse.getData())) {
            return productResponse;
        }
        Category category = categoryResponse.getData();
        productResponse.setCategoryName(category.getName());
        productResponse.setCreateTime(searchItem.getCreateTime());
        productResponse.setUpdateTime(searchItem.getUpdateTime());
        // 获取图片
        List<String> images = Arrays.asList(searchItem.getImages().split(","));
        productResponse.setImages(images);
        // 获取价格
        List<String> prices = Arrays.asList(searchItem.getPrices().split(","));
        productResponse.setPrice(Integer.parseInt(prices.get(0)));
        // 获取标题
        List<String> allTitle = Arrays.asList(searchItem.getAllTitle().split(","));
        productResponse.setTitle(allTitle.get(0));
        // 获取特殊参数
        List<String> specialParam = StringUtil.getFirstBlankString(searchItem.getAllTitle());
        productResponse.setSpecialParam(specialParam);
        productResponse.setSku(searchItem.getSku());
        productResponse.setMerchantName(searchItem.getMerchantName());
        productResponse.setMerchantNumber(searchItem.getMerchantNumber());
        return productResponse;
    }
```
以上就是ElasticSearch的安装和使用过程
好了，本次的技术分享就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！
### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
### 公众号
![公众号](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/qrcode_for_gh_e90987068371_258.jpg)



