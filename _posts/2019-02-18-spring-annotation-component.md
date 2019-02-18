---
layout:     post
title:      Spring Annotation(一)
subtitle:   Srping Component Registry
date:       2019-02-18
author:     WCS
header-img: img/spring-annotation01.jpg
catalog: true
tags:
    - Spring
    - Annotation
---

# Spring注解之组件注册  

## 一、传统的XML注册bean

```
    <!--扫描com.annotation下的组件-->
    <context:component-scan base-package="com.annotation"/>
    <bean id="person" class="com.annotation.bean.Person" scope="prototype">
        <property name="age" value="13"></property>
        <property name="name" value="wcs"></property>
    </bean>
```  

## 二、Spring注解之Configuration

```
/**
 * Configuration声明这是一个配置类
 */
@Configuration
public class MainConfig {

    /**
     * 默认id是方法名,也可以通过value设置,类型为返回值类型
     */
    @Bean(value = "ConfigPerson")
    public Person person() {
        return new Person("wcs_by_annotation", 12);
    }
}
```  

## 三、Spring注解之ComponentScans

```
@Configuration
@ComponentScans(
    value = {
//扫描com.annotation包
@ComponentScan(basePackages = "com.annotation", includeFilters = {
//1.按照注解扫描：扫描@Controller注解的类
//@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class}),
//2.按照给定类型：只扫描BookService这个类
//@ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {BookService.class}),
//3.自定义扫描策略
@ComponentScan.Filter(type = FilterType.CUSTOM, classes = {MyTypeFilter.class}),
//使用includeFilters时,要设置useDefaultFilters = false,其默认是true
useDefaultFilters = false)
    }
)
public class MainConfig {
    @Bean(value = "personAnnotation")
    public Person person() {
        return new Person("wcs_by_annotation", 12);
    }
}
```  

自定义的扫描文件要实现TypeFilter

```
public class MyTypeFilter implements TypeFilter {
    /**
     * metadataReader：读取到的当前正在扫描的类的信息
     * metadataReaderFactory:可以获取到其他任何类信息的
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) {
        //获取当前类注解的信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取当前正在扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //获取当前类资源（类的路径）
        Resource resource = metadataReader.getResource();
        String className = classMetadata.getClassName();
        System.out.println("--->" + className);
//                --->com.annotation.bean.Person
//                --->com.annotation.config.MyTypeFilter
//                --->com.annotation.controller.BookController
//                --->com.annotation.dao.BookDao
//                --->com.annotation.service.BookService
//                --->com.annotation.Test
        if (className.contains("er")) {
            //上面com.annotation包下扫描到的都包含er所以都会在容器中
            return true;
        }
        return false;
    }
```

## 四、Spring注解之Scope

