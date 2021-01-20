## CodeWorld-Cloud-Shop 上传服务图片上传详解篇---奶妈级教学 {docsify-ignore}
>今天我们来讲解我们使用到的上传服务，我们怎么实现将图片上传并保存下来

我们这里使用到的是腾讯云的COS对象储存，当然还有其他的比如七牛云，阿里云啊
还有可以使用分布式文件系统FastDFS，等等等等。。。。。。。。。。

### 首先我们要先去腾讯对象储存上创建一个储存桶
![bucket](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/04/18931583305584139.png)
![](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/04/77441583305917395.png)
![](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/03/04/41161583306037146.png)
创建这个桶的步骤就不用详解了
我们主要的核心是代码，怎么实现的上传
### Pom文件
首先要引入我们的pom依赖
```java
 <!-- 腾讯云 -->
        <dependency>
            <groupId>com.qcloud</groupId>
            <artifactId>cos_api</artifactId>
            <version>5.6.8</version>
        </dependency>
```
### 核心代码
```java
/**
 * @author Lenovo
 * 图片上传到腾讯云
 */
public class TencentCOS {

    /**
     * 图片访问地址
     */
    private static final String IMAGE_URL = "";

    /**
     * 存储桶的名称
     */
    private static final String BUCKET_NAME = "";

    /**
     * 密钥ID
     */
    private static final String SECRET_ID = "";

    /**
     * 密钥Key
     */
    private static final String SECRET_KEY = "";

    // 1 初始化用户身份信息(secretId, secretKey，可在腾讯云后台中的API密钥管理中查看！
    private static COSCredentials credentials = new BasicCOSCredentials(SECRET_ID, SECRET_KEY);

    // 2 设置bucket的区域, COS地域的简称请参照
    private static ClientConfig clientConfig = new ClientConfig(new Region("ap-chengdu"));
    
    /**
     * 上传图片
     *
     * @param file
     * @return
     */
    public static String uploadImage(File file) {
    
     // 生成Cos客户端
        COSClient cosClient = new COSClient(credentials, clientConfig);
        
       SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd");

        String fileName = "";

        try {

            fileName = file.getName();

            String substring = fileName.substring(fileName.lastIndexOf("."));

            Random random = new Random();

            // 指定要上传到COS上的路径
            fileName = dateFormat.format(new Date()) + "/" + random.nextInt(10000) + System.currentTimeMillis() + substring;

            PutObjectRequest putObjectRequest = new PutObjectRequest(BUCKET_NAME, fileName, file);

            PutObjectResult putObjectResult = cosClient.putObject(putObjectRequest);

        } catch (Exception e) {

            e.printStackTrace();

            return null;

        } finally {

            cosClient.shutdown();
        }

        // 返回图片访问地址
        return IMAGE_URL + fileName;
    }  
    
    /**
     * 从腾讯云上全部查询数据
     *
     * @param bucketName
     * @return
     */
    public static ObjectListing listObjects(String bucketName) {

        // 生成Cos客户端
        COSClient cosClient = new COSClient(credentials, clientConfig);

        // 获取bucket下成员（设delimiter）
        ListObjectsRequest listObjectsRequest = new ListObjectsRequest();

        listObjectsRequest.setBucketName(bucketName);

        // 设置list的prefix，表示list出来的文件key都是以这个prefix开始
        listObjectsRequest.setPrefix("/");

        listObjectsRequest.setDelimiter("");

        listObjectsRequest.setMaxKeys(100);

        ObjectListing objectListing = cosClient.listObjects(listObjectsRequest);

        String nextMarker = objectListing.getNextMarker();

        boolean truncated = objectListing.isTruncated();

        List<COSObjectSummary> objectSummaries = objectListing.getObjectSummaries();

        return objectListing;
    }

    /**
     * 删除图片，通过Key
     * @param bucketName
     * @param key
     * @return
     */
    public static boolean delete(String bucketName, String key) {

        boolean flag = false;

        try {

            // 执行删除
            // 生成Cos客户端
            COSClient cosClient = new COSClient(credentials, clientConfig);

            cosClient.deleteObject(BUCKET_NAME, key);

            return flag = true;

        } catch (Exception e) {

            e.printStackTrace();

            return flag;
        }
    }
}   
```
代码中详细的备注信息已经标明

### 实践
那么我们结合我们的项目来实现一个图片的上传
#### 启动我们的一系列动作
- 启动我们的nacos服务注册中心
- 启动codeworld-cloud-gateway网关模块
- 启动codeworld-cloud-auth认证模块
- 启动codeworld-cloud-system 系统模块
- 启动codeworld-cloud-goods 商品模块
- 启动codeworld-cloud-upload 上传模块

启动成功后
![server-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-upload/server-start.png)

#### 启动web端界面
![code-web-start](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-upload/code-web-start.png)
#### 然后登陆上我们的界面，这里使用系统管理员登陆，点击添加分类上传图片按钮
![add-category](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-upload/add-category.png)
#### 上传成功
![upload-image-success](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-upload/upload-image-success.png)

这样我们的图片就上传成功了
我们来到我们的腾讯对象储存桶里查看
![bucket-success](https://codeworld-cloud-shop-1300450814.cos.ap-chengdu.myqcloud.com/codeworld-cloud-upload/bucket-success.png)

那么我们的图片就上传成功了

好了，本次的技术分享就到这里了？如果觉得不错的话，点亮一下小星星[codeworld-cloud-shop](https://github.com/javaenigneer/codeworld-cloud-shop-api)
只看不点，不是好孩子哦！！

### 欢迎加入QQ群(964285437)
![QQ群](https://fcblog-1300450814.cos.ap-chengdu.myqcloud.com/2020/hexoblog/temp_qrcode_share_964285437.png)
