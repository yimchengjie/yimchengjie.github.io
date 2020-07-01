---
title: HttpClient技术入门
categories:
  - 爬虫
tags:
  - 爬虫
  - Java爬虫
  - HttpClient
toc: true
date: 2019-12-30 20:05:53
---
## HttpClient技术入门

网络爬虫通常都是使用Http协议访问互联网,所以HttpClient这个同样是Http协议的客户端技术就被运用到了爬虫技术中
只需要在maven引入HttpClient就可以使用

### 简单示例

```java
public static void main(String[] args) {
    // 1.创建HttpClient对象(Default默认设置的)
    CloseableHttpClient httpClient = HttpClients.createDefault();
    // 2.创建请求对象,需要Url参数
    HttpGet httpGet = new HttpGet("https://www.baidu.com");
    // 3.发起请求,接收响应
    CloseableHttpResponse httpResponse = null;
    try {
        httpResponse = httpClient.execute(httpGet);
    } catch (IOException e) {
        e.printStackTrace();
    }
    // 4.解析请求,获取数据
    // 4.1 判断状态码
    if (httpResponse.getStatusLine().getStatusCode() == 200){
        // 4.2 获取主体数据
        HttpEntity httpEntity = httpResponse.getEntity();
        try {
            // 4.3 数据转码
            String context = EntityUtils.toString(httpEntity,"utf-8");
            System.out.println(context);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 使用步骤

#### 1. 构建HTTP请求

```java
// 方法一
HttpGet httpGet = new HttpGet("https://www.baidu.com");

// 方法二
URI uri = new URIBuilder().setScheme("http")
                          .setHost("https://www.baidu.com")
                          .setPath("/s")
                          .setParameter("ie","utf-8")
                          .setParameter("f","8")
                          .setParameter("rsv_bp","1")
                          .setParameter("tn","80035161_2_dg")
                          .build();
HttpGet httpGet = new HttpGet(uri);
```

#### 2. 添加消息头

```java
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,HttpStatus.SC_OK,"OK");
response.addHeader("Set-Cookie","c1=a; path=/; domain=localhost");
response.addHeader("Set-Cookie","c2=b; path=\"/\", c3=c; domain=\"localhost\"");
Header h1 = response.getFirstHeader("Set-Cookie");
Header h2 = response.getLastHeader("Set-Cookie");
Header[] hs = response.getHeaders("Set-Cookie");
```

#### 3. 生成HTTP实体

```java
// 1. String类型的实体
StringEntity myEntity = new StringEntity("important message",ContentType.create("text/plain","UTF-8"));

// 2. File类型实体
File file = new File("onefile.txt");
FileEntity entity = new FileEntity(file,contentType.create("text/plain","UTF-8"));

// 3. 表单实体
List<NameValuePair> formparam = new ArrayList<NameValuePair>();
formparam.add(new BasicNameValuePair("param1","value1"));
formparam.add(new BasicNameValuePair("param2","value2"));
UrlEncodeFormEntity entity = new UrlEncodedFormEntity(formparam,Consts.UTF_8);
```

#### 4. 配置请求信息

```java
// 配置请求信息
RequestConfig requestConfig = RequestConfig.custom()
        .setConnectTimeout(1000)   // 创建连接的最长时间
        .setConnectionRequestTimeout(500)    // 设置获取连接的最长时间
        .setSocketTimeout(10*1000)    // 设置数据传输的最长时间
        .build();
// 添加配置对象
httpGet.setConfig(requestConfig);
```

### 连接池的应用

```java
public static void main(String[] args) {
    // 1. 创建连接池管理器
    PoolingHttpClientConnectionManager poolingHttpClientConnectionManager = new PoolingHttpClientConnectionManager();
    // 2. 使用连接池管理器发起请求
    doGet(poolingHttpClientConnectionManager,"https://www.baidu.com/s","wt","httpclient");
}

private static String doGet(PoolingHttpClientConnectionManager poolingHttpClientConnectionManager,String url,String...param) {
    // 从连接池中获取HttpClient
    CloseableHttpClient httpClient = HttpClients.custom().setConnectionManager(poolingHttpClientConnectionManager).build();

    // 2.1 创建URI对象(带参的URL)
    URIBuilder uriBuilder;
    HttpGet httpGet = null;
    try {
        uriBuilder = new URIBuilder(url);
        for (int i=0;(i+1)<param.length;i+=2)
            uriBuilder.setParameter(param[i],param[i+1]);
        httpGet = new HttpGet(uriBuilder.build());
    } catch (URISyntaxException e) {
        e.printStackTrace();
    }
    // 3.发起请求,接收响应
    CloseableHttpResponse httpResponse = null;
    try {
        httpResponse = httpClient.execute(httpGet);
        // 4.解析请求,获取数据
        // 4.1 判断状态码
        if (httpResponse.getStatusLine().getStatusCode() == 200){
            // 4.2 获取主体数据
            HttpEntity httpEntity = httpResponse.getEntity();
            // 4.3 数据转码
            String context = EntityUtils.toString(httpEntity,"utf-8");
            System.out.println(context);
            return context;
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally{
        // 关闭资源
        try {
            httpResponse.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        /* 交给连接池管理, 不用关闭
        try {
            httpClient.close();
        } catch (IOException e) {
            e.printStackTrace();
        }*/
    }
    return null;
}
```
