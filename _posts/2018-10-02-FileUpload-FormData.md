---
layout: post
title:      FileUpload-FormData
subtitle:   My Personal Plug-in Components 
date:       2018-10-01
author:     WCS
header-img: img/2018-10-02-FileUpload.jpg
catalog: true
tags:
    - JAVA
---

# 前言

上个礼拜公司要做头像上传的功能，于是用到了`FormData`，关于`FormData`上传文件的使用在这里  
就不详细的写了，重点记录下上传文件中踩的坑！  

## 一、`AJAX`文件上传

`AJAX`中的参数设置  

```
processData : false,  //必须false才会避开jQuery对 formdata 的默认处理   
contentType : false,  //必须false才会自动加上正确的Content-Type 
```

## 二、`node`作为中间层

公司最初是用`node`服务器作为中间层，来转发各种请求到`java`后台的。当测试通过node服务器来  
获取后台`mysql`中的`BLOB`格式的图片时，一直获取不到，而通过`tomcat`服务器则可以获取到， 
找了一天的资料才最终解决：  

将`encoding`设置为`null`后，`request`会直接返回`Buffer`类型的，即：  
```
var options = { 
    url: url,
    encoding: null
};
```  

最终通过`node`服务器获取到了图片。  

但是通过`node`服务上传文件到后台又出问题了，因为后台除了要获取文件外，还需要其他参数，  
但是后台却一直获取不到文件以及参数。通过一系列的尝试，将`ajax FormData`传过来的文件  
用fs模块去读取文件信息，然后再引入`FormData`模块，然后`FormData`再`append`一次文件以及  
其他参数。最终后台可以获取到文件数据了，但是其他参数却一直取不到。然后朱老师一看要读取文   件，可能会引起单点阻塞，于是放弃了`node`服务器作为动态资源的中间层,转而用起了`nginx`。  

## 三、`nginx`反向代理

之前从来没用过，配置过`nginx`。打开朱老师发过来的`nginx`文件，自己找到`conf`文件打开看。  

看到下面一段代码，就大概知道`nginx`做了什么了。
```
http {
    include       mime.types;
    default_type  application/octet-stream;
	client_max_body_size    2m;

    server {
        listen       80;
        server_name  localhost;

        location /v1 {
            proxy_pass http://127.0.0.1:8080/appManager/v1/;
        }
        location /upload {
            proxy_pass http://127.0.0.1:8080/appManager/upload/;
        }
        location / {
            proxy_pass http://127.0.0.1:3000/;
        }
    }
}
```
a.`nginx`监听`80`这个端口号,之前访问`node`服务器`http://127.0.0.1:3000/`要换成访问  
`http://127.0.0.1:80/`了。  

b.`nginx`作为中间层，会将你的访问代理到对应的配置`location`上。  

c.以上为对`nginx`只存在知道为反向代理服务器的认知上，作的一些猜想。有待后面修正补充！  

## 四、`request`大坑

中间的服务器在换成`nginx`之后，文件能正常上传。但是，却提示“没有访问权限”，`debug`发现`AOP`  
权限拦截那块`request.getParameter("token")`获取到的`token`为空，所以导致“没有访问权限”。  
但是把权限设置为不需要权限后，在`controller`中能够通过`request.getParameter("token")`  获取到`token`参数，其`AOP`却获取不到！  

于是百度发现，别人也有这种情况，但是是在监听器中取不到，他们给出的最简单的方法是把参数放在  `url`后面，这样确实能够获取到，但是却违反了`RESTFUL`协议啊。  

另一种方法则是:  

a.首先在`spring`配置文件中配置`MultipartHttpServletRequest`  

```
<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver"
		p:defaultEncoding="UTF-8">
		<property name="maxUploadSize">
			<value>104857600</value>
		</property>
		<property name="maxInMemorySize">
			<value>4096</value>
		</property>
</bean>

```

b.在拦截器中注入`MultipartHttpServletRequest`  

```
// 用于创建MultipartHttpServletRequest
private MultipartResolver multipartResolver = null;
	
@Override
public void init(FilterConfig arg0) throws ServletException {
// 注入bean
	multipartResolver = ((MultipartResolver)ApplicationContextUtil.getContext().getBean("multipartResolver", MultipartResolver.class));
}

```

c.在你的最顶层的拦截器中把你的`ServletRequest`替换成`MultipartHttpServletRequest`  

```
	private ServletRequest getRequest(ServletRequest req){
		String enctype = req.getContentType();
		if (StringUtils.isNotBlank(enctype) && enctype.contains("multipart/form-data"))
			// 返回 MultipartHttpServletRequest 用于获取 multipart/form-data 方式提交的请求中 上传的参数
			return multipartResolver.resolveMultipart((HttpServletRequest) req);
		else 
			return req;
	}

```

第二种方式我还没有尝试，因为朱老师亲自上阵`debug`。朱老师不亏是大神啊，通过他的分析，   `controller`中能取到参数，`AOP`中取不到，难道两个地方的`request`不是同一个？然后`debug`  一看，果真不是同一个，然后朱老师代码一敲，换成了同一个request，然后每个controller都要加  
上`HttpServletRequest request`参数，并放在第一个(提高性能)。  

换之后的代码如下:  

```
	public Object around(ProceedingJoinPoint pjp) throws Throwable {
		MethodSignature ms = (MethodSignature) pjp.getSignature();
		Method method = pjp.getTarget().getClass().getMethod(ms.getName(), ms.getParameterTypes());
		Permission permission = method.getAnnotation(Permission.class);
		HttpServletRequest request = null;
		Object[] args = pjp.getArgs();
		for (int i = 0; i < args.length; i++) {
			if (args[i] instanceof HttpServletRequest) {// 参数包含HttpServletRequest
				request = (HttpServletRequest) args[i];
				break;
			}
		}
        ...
    }
```

以前从来没这样取过`request`，目前还看不懂，只知道跟反射有关系。  

# 五、总结

虽然发现了`controller`与`AOP`中`request`不是同一个对象这个大坑，但是真正让我学习到的还是  朱老师解决问题的方式。目前我开发中遇到问题，有具体的报错，我都是直接百度解决，百度解决不了  经常用英文`Google`能找到答案，偶尔能自己通过debug找到问题所在，但还是很少习惯去debug。  

当发现参数在`AOP`取不到，在`controller`能取到，我的解决思路一直都固定在通过其他方式在`AOP`  中取参数，却从来没有想到过，`AOP`与`controller`中的`request`竟然不是同一个对象！  这个在后面的好好研究研究。