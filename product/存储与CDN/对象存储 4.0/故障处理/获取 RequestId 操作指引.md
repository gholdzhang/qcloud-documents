## 简介

在对象存储（Cloud Object Storage，COS）服务中，每次发送请求时，COS 服务端都会为请求生成一个 ID，即 RequestId。本文主要介绍在不同场景下获取 RequestId 的方法。

## 在控制台通过浏览器获取

1. 登录 [对象存储控制台](https://console.cloud.tencent.com/cos5)，在左侧导航栏中单击【存储桶列表】，进入存储桶列表页。
2. 单击并进入想要访问的存储桶。
3. 按`F12`键，进入浏览器的开发者工具页面。
4. 单击开发者工具上方的【Network】。
![](https://main.qcloudimg.com/raw/0a201a890f54bfabc4267e9c86c89338.png)
5. 在需要下载的文件名右侧，单击【下载】，并在开发者工具页面中输入要下载的文件名进行过滤，选择文件，单击【Headers】，在 **Response Headers** 区域中获取 RequestId 信息。
![](https://main.qcloudimg.com/raw/1a97264c5238b43b52c941109f8946b7.png)

## 访问文件失败时获取

您可以在访问文件失败时，从页面展示返回的 XML 信息中获取 **RequestId 节点信息**。
![](https://main.qcloudimg.com/raw/e0d4149121fb0022640465ff690810e1.png)

也可以进行如下操作获取：
1. 按`F12`键，进入浏览器的开发者工具页面。
2. 单击页面上方的【Network】，选择 All 类型，便能在 Response Headers 中找到 RequestId 字段信息。
![](https://main.qcloudimg.com/raw/ac6902c6ac615a9ec2978a5999a49073.png)

## 通过 SDK 获取

由于 SDK 包含的接口太多，无法一一穷举所有接口示例，故所有的 SDK 均以**上传文件**为例，演示如何获取当前操作的 RequestId。

### 通过 .NET SDK 获取

```
try
{
 string bucket = "examplebucket-1250000000"; //存储桶，格式：BucketName-APPID
 string cosPath = "test.cs"; // 对象键
 byte[] data = System.Text.Encoding.Default.GetBytes("Hello COS"); // 二进制数据
 PutObjectRequest putObjectRequest = new PutObjectRequest(bucket, cosPath, data);
 
 PutObjectResult result = cosXml.PutObject(putObjectRequest);
 string requestId = result.responseHeaders.GetValueOrDefault("x-cos-request-id")[0];
 Console.WriteLine(requestId);
}
catch (COSXML.CosException.CosClientException clientEx)
{
 //请求失败
 Console.WriteLine("CosClientException: " + clientEx);
}
catch (COSXML.CosException.CosServerException serverEx)
{
 //请求失败
 Console.WriteLine("CosServerException: " + serverEx.GetInfo());
}
```

 

### 通过 Go SDK 获取

```
package main
 
import (
   "context"
   "net/http"
   "net/url"
   "os"
   "strings"
   "github.com/tencentyun/cos-go-sdk-v5"
)
 
func main() {
   // 将 examplebucket-1250000000 和 COS_REGION 修改为真实的信息
   u, _ := url.Parse("https://examplebucket-1250000000.cos.COS_REGION.myqcloud.com")
   b := &cos.BaseURL{BucketURL: u}
   c := cos.NewClient(b, &http.Client{
       Transport: &cos.AuthorizationTransport{
           SecretID:  "SECRETID",
           SecretKey: "SECRETKEY",
       },
   })
   // 对象键（Key）是对象在存储桶中的唯一标识。
   // 例如，在对象的访问域名 `examplebucket-1250000000.cos.COS_REGION.myqcloud.com/test.go` 中，对象键为 test.go
   name := "test.go"
   // 1.通过字符串上传对象
   f := strings.NewReader("Hello COS")
 
   _, err := c.Object.Put(context.Background(), name, f, nil)
   if err != nil {
       // error信息中直接包含RequestId字段
       panic(err)
   }
   requestId := response.Header.Get("X-Cos-Request-Id")
   fmt.Println(requestId)
}
```

### 通过 Java SDK 获取

```
// 1 初始化用户身份信息（secretId, secretKey）。
String secretId = "SECRETID";
String secretKey = "SECRETKEY";
COSCredentials cred = new BasicCOSCredentials(secretId, secretKey);
// 2 设置 bucket 的地域, COS 地域的简称请参照 https://cloud.tencent.com/document/product/436/6224
// clientConfig 中包含了设置 region, https(默认 http), 超时, 代理等 set 方法, 使用可参见源码或者常见问题 Java SDK 部分。
Region region = new Region("COS_REGION");
ClientConfig clientConfig = new ClientConfig(region);
// 这里建议设置使用 https 协议
clientConfig.setHttpProtocol(HttpProtocol.https);
// 3 生成 cos 客户端。
COSClient cosClient = new COSClient(cred, clientConfig);
// Bucket的命名格式为 BucketName-APPID ，此处填写的存储桶名称必须为此格式
String bucketName = "examplebucket-1250000000";
 
String content = "Hello COS";
String key = "test.java";
PutObjectResult putObjectResult = cosClient.putObject(bucketName, key, content);
String requestId = putObjectResult.getRequestId();
System.out.println(requestId);
```

 

### 通过 JavaScript SDK 获取

```
cos.putObject({
    Bucket: 'examplebucket-1250000000', /* 必须 */
    Region: 'COS_REGION',    /* 必须 */
    Key: 'test.js',              /* 必须 */
    StorageClass: 'STANDARD',
    Body: Buffer.from('Hello COS'),
    onProgress: function(progressData) {
        console.log(JSON.stringify(progressData));
    }
}, function(err, data) {
    var requestId = (err || data).headers['x-cos-request-id'];
    console.log(requestId );
});
```

 

### 通过 Node.js SDK 获取

```
var COS = require('cos-nodejs-sdk-v5');
var cos = new COS({
    SecretId: 'SECRETID',
    SecretKey: 'SECRETKEY'
});
 
cos.putObject({
    Bucket: 'examplebucket-1250000000', /* 必须 */
    Region: 'COS_REGION',    /* 必须 */
    Key: 'test.nodejs',              /* 必须 */
    StorageClass: 'STANDARD',
    Body: Buffer.from('Hello COS'),
    onProgress: function(progressData) {
        console.log(JSON.stringify(progressData));
    }
}, function(err, data) {
    var requestId = (err || data).headers['x-cos-request-id'];
    console.log(requestId );
});
```

 

### 通过 PHP SDK 获取

```
$secretId = "SECRETID"; //"云 API 密钥 SecretId";
$secretKey = "SECRETKEY"; //"云 API 密钥 SecretKey";
$region = "COS_REGION"; //设置一个默认的存储桶地域
$cosClient = new Qcloud\Cos\Client(
   array(
       'region' => $region,
       'schema' => 'https', //协议头部，默认为http
       'credentials'=> array(
           'secretId'  => $secretId ,
           'secretKey' => $secretKey)));
# 上传文件
## putObject(上传接口，最大支持上传5G文件)
### 上传内存中的字符串
try {
   $bucket = "examplebucket-1250000000"; //存储桶名称 格式：BucketName-APPID
   $key = "test.php"; //此处的 key 为对象键，对象键是对象在存储桶中的唯一标识
   $result = $cosClient->putObject(array(
       'Bucket' => $bucket,
       'Key' => $key,
       'Body' => 'Hello COS'));
   $requestId = $result['RequestId'];
   print_r($requestId);
} catch (\Exception $e) {
   echo "$e\n";
}
```


### 通过 Python SDK 获取

```
# -*- coding=utf-8
# appid 已在配置中移除,请在参数 Bucket 中带上 appid。Bucket 由 BucketName-APPID 组成
# 1. 设置用户配置, 包括 secretId，secretKey 以及 Region
from qcloud_cos import CosConfig
from qcloud_cos import CosS3Client
import sys
import logging
logging.basicConfig(level=logging.INFO, stream=sys.stdout)
secret_id = 'SECRETID'      # 替换为用户的 secretId
secret_key = 'SECRETKEY'      # 替换为用户的 secretKey
region = 'COS_REGION'     # 替换为用户的 Region
token = None                # 使用临时密钥需要传入 Token，默认为空，可不填
scheme = 'https'            # 指定使用 http/https 协议来访问 COS，默认为 https，可不填
config = CosConfig(Region=region, SecretId=secret_id, SecretKey=secret_key, Token=token, Scheme=scheme)
# 2. 获取客户端对象
client = CosS3Client(config)
# 字节流简单上传
response = client.put_object(
   Bucket='examplebucket-1250000000',     # 替换为用户的存储桶名称
   Body=b'Hello COS',
   Key='test.py',
   EnableMD5=False
)
request_id = response['X-Cos-Request-Id']
print(request_id)
```
