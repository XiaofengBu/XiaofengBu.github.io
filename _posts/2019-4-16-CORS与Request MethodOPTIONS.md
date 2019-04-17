---
layout: post
title:  "CORS与Request Method:OPTIONS"
categories: SpringBoot
date:  2019-4-16 19:55:40
tags:  SpringBoot  
author: Xiaofeng
---

* content
{:toc}


一直注意到在前端的NetWork中的http请求对同一个地址总是发送两次,其中第一次的Request Method为OPTIONS,第二次才是正式的http请求,因此查询相关资料,发现和CORS跨域相关。

### http请求动词
首先http请求的动词并不是固定不变的,随着标准的改变会有所增加,目前有八种动词:
1. GET 向服务器请求一个资源,因为本身不对资源修改或更新所以是幂等的(幂等是指,无论多少次请求在没有人为修改服务器端数据的前提下,请求的结果都是相同的)
2. POST 向服务器提交数据 会导致资源增加,所以自然不是幂等
3. PUT 向指定资源位置上上传其最新内容,多次调用永远都会保持最新的,即后一次的会覆盖前一次的资源,所以是幂等的
4. HEAD 请求一个资源的部分信息而不是全部 幂等
5. DELETE 删除一个资源幂等
6. TRACE 
7. OPTIONS 客户端决定使用其他方式检索和处理服务端的文档
8. CONNECT 当客户端想要确定一个明确的连接到远程主机的时候使用

### CORS跨域
CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。
它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。
对于用户和开发者,CORS跨域都是无感的,只需要后端实现CORS接口,就可以像AJAX一样使用CORS进行跨域访问。
浏览器将CORS请求分为两类：简单请求（simple request）和非简单请求（not-so-simple request）。
简单请求满足条件:
1. 请求方法是以下三种方法之一:
 - HEAD
 - GET
 - POST
2. HTTP的头信息不超出以下几种字段:
 - Accept
 - Accept-Language
 - Content-Language
 - Last-Event-ID
 - Content-Type:只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。

Origin字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。

如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。都以Access-Control- 开头。

虽然使用的动词是GET,但是在Content-Type中的Content-Type: application/json因此不能算作简单请求

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

在页面域名与接口域名不一致的情况下，就出现了每次请求前先发送一个options请求的问题。

OPTIONS请求头信息中，除了Origin字段，还至少会多两个特殊字段：

（1）Access-Control-Request-Method

该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法。

（2）Access-Control-Request-Headers

该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段。



### CORS在SpringBoot中的配置
在Springboot中配置CORS的代码如下:
```
@Configuration
public class CorsConfiguration extends WebMvcConfigurerAdapter {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") //可跨域的路径
                .allowedOrigins("*") //允许发起请求的域名
                .allowedHeaders("*") //允许的请求头
                .allowedMethods("GET", "POST", "DELETE", "PUT")//允许的请求动词 如GET POST
                .maxAge(3600);
    }

}
```

### 和JSONP的比较
JSONP技术只支持GET方式的请求,CORS更加强大
但JSONP对老旧浏览器的支持更好




