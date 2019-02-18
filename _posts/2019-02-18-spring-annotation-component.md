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
    - Notes
---

# 前言
Spring组件注册相关的spring注解。

## 一、传统的XML注册bean

```
    <!--扫描com.annotation下的组件-->
    <context:component-scan base-package="com.annotation"/>
    <bean id="person" class="com.annotation.bean.Person" scope="prototype">
        <property name="age" value="13"></property>
        <property name="name" value="wcs"></property>
    </bean>
```  

## 二、@Configuration

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

## 三、@ComponentScans

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

## 四、@Scope

```
    /**
     * ConfigurableBeanFactory#SCOPE_PROTOTYPE
     *
     * @Scope:
     * 调整作用域 prototype：多实例的：ioc容器启动并不会去调用方法创建对象放在容器中。
     * 每次获取的时候才会调用方法创建对象；
     * singleton：单实例的（默认值）：ioc容器启动会调用方法创建对象放到ioc容器中，
     *              以后每次获取就是直接从容器（map.get()）中拿。
     * request：同一次请求创建一个实例
     * session：同一个session创建一个实例
     */
    @Scope("prototype")
    @Bean("person")
    public Person person() {
        System.out.println("给容器中添加Person....");
        return new Person("张三", 25);
    }
```

## 五、@Lazy

```
    /**
     * 懒加载：
     * 单实例bean：默认在容器启动的时候创建对象；
     * 懒加载：容器启动不创建对象。第一次使用(获取)Bean创建对象，并初始化；
     */
    @Lazy
    @Bean("person")
    public Person person() {
        System.out.println("给容器中添加Person....");
        return new Person("张三", 25);
    }
```

## 六、@Conditional