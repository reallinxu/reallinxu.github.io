---
title: "使用Feign发送http请求以及源码分析"
id: 20200115
categories:
  - spring cloud
date: 2020-01-15 16:28:47
tags: spring cloud
---

## 使用Feign发送http请求以及源码分析

### 简介
Feign是一个http请求调用的轻量级框架，可以用Java接口注解的方式调用Http请求，而不用像Java中通过封装HTTP请求报文的方式直接调用。 

### 注解(Annotation)
注解 | 作用域 |  用法  
-|-|-
@RequestLine | 方法 | 定义HttpMethod和UriTemplate请求。 {expression}使用大括号括起来的值使用其相应的带@Param注释参数进行解析。 |
@Param | 参数 | 作用于参数上，定义一个模板变量，其值将用于Expression按名称解析相应的模板。 |
@Headers | 方法，类型 | 定义一个HeaderTemplate; 的变体UriTemplate。使用带@Param注释的值来解析相应的Expressions。在上使用时Type，该模板将应用于每个请求。当在上使用时Method，模板将仅应用于带注释的方法。 |
@QueryMap | 参数 | 定义一个Map名称-值对或POJO，以扩展为查询字符串。 |
@HeaderMap | 参数 | 定义一个Map名称-值对，以扩展为Http Headers |
@Body | 方法 | 定义Template，类似于UriTemplate和HeaderTemplate，使用@Param注释值来解决相应的Expressions。 |

> 如果需要将请求定向到其他主机，则需要在创建Feign客户端时提供的主机，或者要为每个请求提供目标主机，请包含一个java.net.URI参数，Feign将使用该值作为请求目标。
> @RequestLine("POST /repos/{owner}/{repo}/issues")
void createIssue(URI host, Issue issue, @Param("owner") String owner, @Param("repo") String repo);

### Demo
#### ready
idea创建一个springboot项目(此处本人项目端口设置为8089，可自行设置)，勾选openfeign如下图所示：
![new project](/imgs/feign/new-project.jpg)

#### test
创建controller作为服务接口  

```java
package com.reallinxu.cloud.controller;

import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;

/**
 * @author linxu
 */
@RestController
public class TestController {

    /**
     * demo1 -- @RequestLine @Param 简单使用
     *
     * @param param1
     * @return
     */
    @GetMapping("/demo1/{param1}")
    public String demo1(@PathVariable String param1) {
        return "Demo1: hello" + param1;
    }

    /**
     * demo2 -- @Headers使用
     *
     * @param param1
     * @return
     */
    @GetMapping("/demo2/{param1}")
    public String demo2(HttpServletRequest request, @PathVariable String param1) {
        System.out.println(request.getHeader("Accept"));
        return "Demo2: hello" + param1;
    }

    /**
     * demo3 -- @QueryMap使用 map
     *
     * @param param1
     * @return
     */
    @GetMapping("/demo3")
    public String demo3(@RequestParam String param1, @RequestParam String param2) {
        return "Demo3: param1-" + param1 + " param2-" + param2;
    }

    /**
     * demo4 -- @QueryMap使用 pojo
     *
     * @param param1
     * @return
     */
    @GetMapping("/demo4")
    public String demo4(@RequestParam String param1, @RequestParam String param2) {
        return "Demo4: param1-" + param1 + " param2-" + param2;
    }

    /**
     * demo5 -- @HeaderMap使用
     *
     * @param request
     * @return
     */
    @GetMapping("/demo5")
    public String demo5(HttpServletRequest request) {
        System.out.println(request.getHeader("Accept"));
        System.out.println(request.getHeader("accept-encoding"));
        return "Demo5: hello ";
    }

    /**
     * demo6 -- @Body使用
     *
     * @param param
     * @return
     */
    @PostMapping("/demo6")
    public String demo6(@RequestBody String param) {
        return "Demo6: param- " + param;
    }

}
```

创建接口，用来作为feign客户端
```java
package com.reallinxu.cloud.feign;


import com.reallinxu.cloud.entity.Demo4;
import feign.*;

import java.net.URI;
import java.util.Map;

/**
 * feign接口
 *
 * @author linxu
 */
public interface TestInterface {

    /**
     * demo1 -- 基本用法
     *
     * @param param1
     * @return
     */
    @RequestLine("GET /demo1/{param1}")
    String demo1(@Param("param1") String param1);

    /**
     * demo2 -- @Headers使用
     *
     * @param param1
     * @param param2
     * @return
     */
    @RequestLine("GET /demo2/{param1}")
    @Headers("Accept: {param2}")
    String demo2(@Param("param1") String param1, @Param("param2") String param2);

    /**
     * demo3 -- @QueryMap使用 map
     *
     * @param paramMap
     * @return
     */
    @RequestLine("GET /demo3")
    String demo3(@QueryMap Map paramMap);

    /**
     * demo4 -- @QueryMap使用 pojo
     *
     * @param demo4
     * @return
     */
    @RequestLine("GET /demo4")
    String demo4(@QueryMap Demo4 demo4);

    /**
     * demo5 -- @HeaderMap使用 pojo
     *
     * @param demo5
     * @return
     */
    @RequestLine("GET /demo5")
    String demo5(@HeaderMap Map demo5);

    /**
     * demo6
     */
    @RequestLine("POST /demo6")
    @Headers("Content-Type: application/json")
    @Body("%7B\"param1\": \"{param1}\", \"param2\": \"{param2}\"%7D")
    String demo6(@Param("param1") String param1, @Param("param2") String param2);

    /**
     * demoN -- 自定义host
     *
     * @param param1
     * @return
     */
    @RequestLine("GET /demo1/{param1}")
    String demoN(URI host, @Param("param1") String param1);

}

```
创建一个Demo4用到的pojo
```java
package com.reallinxu.cloud.entity;

public class Demo4 {
    private String param1;

    private String param2;

    public String getParam1() {
        return param1;
    }

    public void setParam1(String param1) {
        this.param1 = param1;
    }

    public String getParam2() {
        return param2;
    }

    public void setParam2(String param2) {
        this.param2 = param2;
    }
}

```

创建测试案例进行测试
```java
package com.reallinxu.cloud.test;

import com.reallinxu.cloud.entity.Demo4;
import com.reallinxu.cloud.feign.TestInterface;
import feign.Feign;

import java.net.URI;
import java.net.URISyntaxException;
import java.util.HashMap;
import java.util.Map;

/**
 * 测试类
 * @author linxu
 */
public class Test {
    public static void main(String[] args) throws URISyntaxException {
        TestInterface testInterface = Feign.builder()
                        .target(TestInterface.class, "http://localhost:8089");

        //demo1
        System.out.println(testInterface.demo1("demo1 - 1"));

        //demo2
        System.out.println(testInterface.demo2("demo2 - 1", "text/html"));

        //demo3
        Map demo3Map = new HashMap<String,String>();
        demo3Map.put("param1","heihei");
        demo3Map.put("param2","haha");
        System.out.println(testInterface.demo3(demo3Map));

        //demo4
        Demo4 demo4 = new Demo4();
        demo4.setParam1("heihei");
        demo4.setParam2("haha");
        System.out.println(testInterface.demo4(demo4));

        //demo5
        Map demo5 = new HashMap<String,String>();
        demo5.put("Accept","text/html");
        demo5.put("accept-encoding","gzip");
        System.out.println(testInterface.demo5(demo5));

        //demo6
        System.out.println(testInterface.demo6("heihei","haha"));

        //自定义host，此处会访问8081端口，连接拒绝
        URI a = new URI("http://localhost:8081");
        System.out.println(testInterface.demoN(a,"demo2 - 2"));
    }
}

```
启动springboot项目，运行Test查看执行结果。

### 源码分析
#### 定义feign客户端接口
	TestInterface testInterface = Feign.builder().target(TestInterface.class, "http://localhost:8089");
feign.builder()时可以设置一些参数，未设置为默认值，参数对应如下：
```java
//控制日志级别
//NONE：不记录。
//BASIC：只记录请求方法和URL以及响应状态代码和执行时间。
//HEADERS：记录基本信息以及请求和响应头。
//FULL：记录请求和响应的头、正文和元数据。
this.logLevel = Level.NONE; 
//定义在接口上有效的注释和值。通过registerClassAnnotation对@RequestLine、@param等注解的定义及对应处理
this.contract = new Default();
//Http请求客户端，https调用时需要设置参数
this.client = new feign.Client.Default((SSLSocketFactory)null, (HostnameVerifier)null);
//重试机制，设置重试期间与重试次数
this.retryer = new feign.Retryer.Default();
//可以设置自己的记录日志方式
this.logger = new NoOpLogger();
//定义编码器，请求参数为对象时调用，默认支持json与String
this.encoder = new feign.codec.Encoder.Default();
//定义解码器，响应为对象时调用，将响应转为对应对象
this.decoder = new feign.codec.Decoder.Default();
//如果@QueryMap使用的类型是对象会通过编码转为Map
this.queryMapEncoder = new feign.QueryMapEncoder.Default();
//可以根据状态实现抛出适合项目场景的类型异常
this.errorDecoder = new feign.codec.ErrorDecoder.Default();
//控制客户端请求的设置，设置连接时间，读取时间，是否支持3xx重定向
this.options = new Options();
//控制反射方法的调用
this.invocationHandlerFactory = new feign.InvocationHandlerFactory.Default();
//解码完成后自动关闭响应
this.closeAfterDecode = true;
//apache许可证不允许使用文件啥啥玩意的，不管了
this.propagationPolicy = ExceptionPropagationPolicy.NONE;
```
接下来我们看下target方法，原理主要是根据接口生成一个Proxy代理类，主要方法是build().newInstance(target)
![new project](/imgs/feign/feign-core1.jpg)
特别注意框中的几个参数，我们debug进去看下，各自是什么东西
![new project](/imgs/feign/feign-core2.jpg)
可以看出nameToHandler中的key为接口中的方法，value为初始化Feign时设置的参数，注意target中有传入的host地址
![new project](/imgs/feign/feign-core3.jpg)
methodToHandler也很简单直白，明显就是为了以后的反射
![new project](/imgs/feign/feign-core4.jpg)
``` java
default void defaultMethod(){
    System.out.println("我是默认方法");
}
```
我们将TestInterface接口中加入上方默认方法再debug可以看到defaultMehthodHandlers用于存放接口的默认方法,默认方法需要另外绑定到代理对象上。

#### 调用feign客户端接口
代理类就已经生成完成，接下来我们看调用。
![new project](/imgs/feign/feign-core5.jpg)
除了object中自带的一些方法其他的都走反射
![new project](/imgs/feign/feign-core6.jpg)
这里可以看到创建了RestTemplate用来发起请求，并使用创建时设置的配置
![new project](/imgs/feign/feign-core7.jpg)
创建RestTemplate时会将表达式进行转换为正确的连接
![new project](/imgs/feign/feign-core8.jpg)
通过client发送请求，整个请求完成。

### end
Feign的Http调用到此处就完结了，Feign微服务中还可以通过service-id进行调用，内部使用ribbon进行负载均衡，等后续有兴趣再继续更新，over~





