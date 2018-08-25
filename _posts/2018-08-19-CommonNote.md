---
layout:     post
title:      慕课网课程之大众点评笔记
subtitle:   Learning from common
date:       2018-08-19
author:     WCS
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - Notes
---
# 这是一篇从慕课网学习大众点评开发的笔记

> 课程的链接为[IT段子手详解MyBatis遇到Spring 秒学Java SSM开发大众点评](https://coding.imooc.com/class/105.html)

## 1.entity

字段long 别用基本类型，用Long封装类更好，数据库取出来为null，更方便用。

## 2.根据数据库字段创建实体类

这种方法感觉用idea自动创建更为方便。在这只是贴上如何在数据库查询某表的字段。  
`SELECT COLUMN_name FROM information_schema.COLUMNS WHERE table_schema='comment' AND table_name='ad'`  
通过以上代码即可查询到数据库comment中ad表中的字段，以前从来不知道数据库还可以这样查，于是便记下来。

## 3.dao层查询

`int add();`  
返回int是为了查看影响的条数，后面有可能用得到。

## 4.service层

`public boolean add(){}`
返回值为boolean可以用来判断是否新增成功。

## 5.文件上传

利用MultipartFile实现文件上传

## 6.数据库中字段weight权重

这个权重可以用在前端取数据的时候，数据根据这个个权重来显示在页面的顺序。

## 7.restful风格有上传文件时的PUT/DELETE请求

有上传文件，multipartResolver是配在springmvc.xml中，文件上传，PUT请求时，由于过滤器hiddenHttpMethodFilter不能过滤表单为enctype ="multipart/form-data，所以无法将表单中`<input="hidden" name="_method" value="PUT">`解析PUT请求。所以在过滤器hiddenHttpMethodFilter之前就需要进行有文件上传的FORM表单进行解析。

```filter
<filter>
    <filter-name>MultipartFilter</filter-name>
        <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class>
    <init-param>
    <param-name>multipartResolverBeanName</param-name>
    <param-value>multipartResolver</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>MultipartFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

但是，通过跟踪源码，spring中获取上下文为null,beanName=multipartResolver能取到，解决方法为将原来放springmvc.xml的文件上传配置放到，spring.xml配置中，其获取的上下文才不为null，才能够正常拦截。

```filter
<context-param>
    <!-- 配置spring资源 -->
    <param-name>contextConfigLocation</param-name>
    <!-- 配置文件文件路径 -->
    <param-value>classpath:spring-*.xml</param-value>
</context-param>
```

## 8.mybatis中引用静态常量的方法

`d_city.type='${@org.imooc.constant.DicTypeConst@CITY}'`//CITY是常量字段,(方法也行)。  
为什么要这样引用呢？当然是为了让程序高度的解耦。

## 9.BeanUtils的用法

`BeanUtils.copyProperties(teacher,teacherForm)`  
可以省去大量的get/set：  
`teacher.setName(teacherForm.getName());`

## 10.表单验证

可以用jQuery的插件Validation

## 11.大众点评中统计订单已售数量

**商户表**冗余已售数量字段，**订单表**。SpringTask定时执行更新已售数量字段，但是为了避免每次都coun(*)所有数据,当数据过多时太慢，增加一个 **task表**，第一次统计已售数量时，将这次成功执行的时间插入 **task表**中，则下次统计数量时添加一个条件， **商户表**中的时间需大于上次成功执行的时间(新增的订单)，然后将获取的数量与上一次的数量相加。

## 12.总结

以上就是快速看视频学习完后的一个课程总结，学到的东西还是不少的，一些自己以前从来没用过的用法，以及一些写代码的思路等等，都很值得学习。这篇笔记也是匆忙的用MarkDown赶出来的，3个月前学习了下MarkDown语法，但一直都没怎么使用，导致现在写博客的时候好多语法都忘记了。这篇博客的格式也是匆忙的排了下版，有点乱。下次一定多花点时间，把版面弄整齐！