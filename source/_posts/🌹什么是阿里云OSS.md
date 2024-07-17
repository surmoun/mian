---
title: 【SpringBoot篇】阿里云OSS--文件存储
date: '2024-06-04 08:05'
tag: SpringBoot
categories: Java
cover: 'https://s21.ax1x.com/2024/07/16/pkI4pOx.png'
abbrlink: 2973527110
---



##  🌹什么是阿里云OSS

阿里云对象存储（Alibaba Cloud Object Storage Service，简称OSS）是阿里云提供的海量、安全、低成本、高可靠的云存储服务。用户可以通过简单的API接口在任何时间、任何地点上传和下载数据，适用于图像、音视频、文档、网站等各种类型的数据存储和分发场景。

## ⭐阿里云OSS的优点
**高可靠性**：OSS采用了多副本存储和容灾备份机制，确保数据的高可靠性和持久性。
**安全性**：支持多种安全策略，如访问控制、加密传输等，保障数据的安全性。
**弹性扩展**：支持按需存储和弹性扩展，用户可以根据实际需求灵活调整存储容量。
**低成本**：OSS的存储费用低廉，且支持按量付费模式，使用户能够根据实际使用量付费
## 🏳️‍🌈为什么要使用云服务OSS
解决本地存储无法直接访问，本地磁盘空间有限，本地磁盘损坏这3个问题
## 🎄使用步骤
![image-20240716215007220](https://s21.ax1x.com/2024/07/16/pkI2Fu8.png)


## 文件上传操作

------------

> 🎈在pom文件中添加下面的代码

```xml
    <properties>
        <aliyun.sdk.oss>3.10.2</aliyun.sdk.oss>
    </properties>
	
    <dependency>
         <groupId>com.aliyun.oss</groupId>
         <artifactId>aliyun-sdk-oss</artifactId>
         <version>${aliyun.sdk.oss}</version>
     </dependency>

```
> 🎈本地存储的话 在yml中写明存储地方
```yaml
ali:
  spring:
    localhost: D:\Program Files\Project\Test-Project\TEST01\alioosdemo\ali-service\src\main\resources\upload
```

> 🎈在Controller层进行上传操作
```java
@RestController
@RequestMapping("/api/common")

public class CommonController {
    @Value("${ali.spring.localhost}")
    private  String path;

    @Autowired
    private AliOssUtil aliOssUtil;
    /**
     * 文件上传
     */
    @PostMapping("upload")
    public String Upload(MultipartFile file) throws IOException {

        //获取原始名
        String originalFilename = file.getOriginalFilename();
        //截取 .png
        String substring = originalFilename.substring(originalFilename.lastIndexOf("."));
        //构造一个新名称  使用Hutool工具类
        String filename = IdUtil.simpleUUID() + substring;
	//新建File对象 进行存储
        File file1 = new File(filename);
     	try {
            file.transferTo(file1);
         } catch (IOException e) {
            throw new RuntimeException(e);
         }
         return  "上传成功";
    }


}
```
## 文件上传操作（OSS版）
> 🎈Oss存储 在yml中写明存储地方 **access-key-id**、 **access-key-secret**、**bucket-name**
```yaml
ali:
  alioos:
    endpoint: oss-cn-beijing.aliyuncs.com
    access-key-id: ***********
    access-key-secret: ***********
    bucket-name: ***********

```

> 使用@ConfigurationProperties注解读取 properties的数据
```java
/**
*
*@Component 表示将该类标识为Bean
*@ConfigurationProperties(prefix = "ali.alioos")用于绑定属性，其中prefix表示所绑定的属性的前缀。
**/
@Component
@ConfigurationProperties(prefix = "ali.alioos")
@Data
public class AliOssProprtyies {
    private String endpoint;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;
}
```
> 配置类，用于创建AliOssUtil对象
```java

@Configuration
public class OssConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public AliOssUtil aliOssUtil(AliOssProprtyies aliOssProprtyies){
        return  new AliOssUtil(aliOssProprtyies.getEndpoint(),
                    aliOssProprtyies.getAccessKeyId(),
                    aliOssProprtyies.getAccessKeySecret(),
                aliOssProprtyies.getBucketName());
    }
}

```
> 定义阿里云工具类
```java
@Data
@AllArgsConstructor
@Slf4j
public class AliOssUtil {

    private String endpoint;
    private String accessKeyId;
    private String accessKeySecret;
    private String bucketName;

    /**
     * 文件上传
     *
     * @param bytes
     * @param objectName
     * @return
     */
    public String upload(byte[] bytes, String objectName) {

        // 创建OSSClient实例。
        OSS ossClient = new OSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);

        try {
            // 创建PutObject请求。
            ossClient.putObject(bucketName, objectName, new ByteArrayInputStream(bytes));
        } catch (OSSException oe) {
            System.out.println("Caught an OSSException, which means your request made it to OSS, "
                    + "but was rejected with an error response for some reason.");
            System.out.println("Error Message:" + oe.getErrorMessage());
            System.out.println("Error Code:" + oe.getErrorCode());
            System.out.println("Request ID:" + oe.getRequestId());
            System.out.println("Host ID:" + oe.getHostId());
        } catch (ClientException ce) {
            System.out.println("Caught an ClientException, which means the client encountered "
                    + "a serious internal problem while trying to communicate with OSS, "
                    + "such as not being able to access the network.");
            System.out.println("Error Message:" + ce.getMessage());
        } finally {
            if (ossClient != null) {
                ossClient.shutdown();
            }
        }

        //文件访问路径规则 https://BucketName.Endpoint/ObjectName
        StringBuilder stringBuilder = new StringBuilder("https://");
        stringBuilder
                .append(bucketName)
                .append(".")
                .append(endpoint)
                .append("/")
                .append(objectName);

        log.info("文件上传到:{}", stringBuilder.toString());

        return stringBuilder.toString();
    }
}
```
> 自定义上传接口
```java

    @Autowired
    private AliOssUtil aliOssUtil;
    /**
     * 文件上传
     */

    @PostMapping("upload")
    public String Upload(MultipartFile file) throws IOException {

        //获取原始名
        String originalFilename = file.getOriginalFilename();
        //截取 .png
        String substring = originalFilename.substring(originalFilename.lastIndexOf("."));
        //构造一个新名称
        String filename = IdUtil.simpleUUID() + substring;


        aliOssUtil.upload(file.getBytes(), filename);

        return  "上传成功";
    }


```