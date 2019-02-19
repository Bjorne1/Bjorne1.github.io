---
layout:     post
title:      Spring Annotation(二)
subtitle:   Srping Life Cycle
date:       2019-02-19
author:     WCS
header-img: img/2019-02-19-spring-annotation-life-cycle.jpg
catalog: true
tags:
    - Spring
    - LifeCycle
    - Notes
---

# 前言
Spring bean的生命周期。

## 一、Spring bean的生命周期

* bean创建---初始化----销毁的过程  
* 容器管理bean的生命周期；
* 我们可以自定义初始化和销毁方法
   * 容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法
* 构造（对象创建）
   * 单实例：在容器启动的时候创建对象
   * 多实例：在每次获取的时候创建对象
* 销毁：
   * 单实例：容器关闭的时候
   * 多实例：容器不会管理这个bean；容器不会调用销毁方法；

## 二、指定初始化和销毁方法

```
@ComponentScan("com.annotation.bean")
@Configuration
public class MainConfigOfLifeCycle {

    //@Scope("prototype")
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }

}
```  

```
public class Car {

    public Car() {
        System.out.println("Car constructor");
    }

    public void init() {
        System.out.println("Car is initialized");
    }

    public void destroy() {
        System.out.println("Car is destroy");
    }
}

```  
打印结果：  
```
Car constructor
Car is initialized
Car is destroy
```  
* 打开`//@Scope("prototype")`注解打印，控制台则没有输出，因为多实例只有在每次获取的时候才创建对象。

## 三、实现InitializingBean,DisposableBean接口

```
public class Cat implements InitializingBean,DisposableBean {
	
	public Cat(){
		System.out.println("cat constructor...");
	}
    //销毁方法
	@Override
	public void destroy() throws Exception {
		System.out.println("cat...destroy...");
	}
    //初始化方法
	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("cat...afterPropertiesSet...");
	}

}
```  
打印结果：  
```
cat constructor...
cat...afterPropertiesSet...
cat...destroy...
```

## 四、使用JSR250

* 使用@PostConstruct和@PreDestroy java规范中的两个注解

```
public class Dog {

    public Dog(){
        System.out.println("dog constructor...");
    }

    //对象创建并赋值之后调用
    @PostConstruct
    public void init(){
        System.out.println("dog....@PostConstruct...");
    }

    //容器移除对象之前
    @PreDestroy
    public void detory(){
        System.out.println("dog....@PreDestroy...");
    }

}
```  
打印结果：  
```
dog constructor...
dog....@PostConstruct...
dog....@PreDestroy...
```

## 五、初始化前后BeanPostProcessor
需要注意的是，BeanPostProcessor是在bean初始化前后分别执行的postProcessBeforeInitialization和postProcessAfterInitialization。

```
/**
 * 后置处理器：初始化前后进行处理工作
 * 将后置处理器加入到容器中
 */
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..." + beanName + "=>" + bean);
        return bean;
    }

}
```  
使用后置处理器后，bean初始化执行顺序打印结果如下：  
```
Car constructor
postProcessBeforeInitialization...car=>com.annotation.bean.Car@4a22f9e2
Car is initialized
postProcessAfterInitialization...car=>com.annotation.bean.Car@4a22f9e2
```  

* 原理分析：在bean初始化initializeBean方法中，执行顺序如下：  

```
//先调用这个方法给bean进行属性赋值，再执行下面的代码
populateBean(beanName, mbd, instanceWrapper);
```  

```
protected Object initializeBean(...){

    ...

    /**
    * 遍历得到容器中所有的BeanPostProcessor；挨个执行beforeInitialization，
    * 一但返回null，跳出for循环
    * 不会执行后面的BeanPostProcessor.postProcessorsBeforeInitialization
    */
    applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);

    ...

    invokeInitMethods(beanName, wrappedBean, mbd);执行自定义初始化

    ...

    applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);

    ...
}
```  

* 经过一步步debug，将整理出一张简易的流程图，然后到最后才发现debug的是ApplicationContextAwareProcessor的底层。。但不影响，跟MyBeanPostProcessor都是差不多的。  

![lifeCycle](/img/BeanLifeCycle.png)  

> Spring底层对 BeanPostProcessor 的使用:  
> bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async,xxx BeanPostProcessor

## 六、总结
通过debug查看BeanPostProcessor底层代码，了解了bean从创建，到初始化，再到销毁的整个过程。现在对bean对象的生命周期有了更深的认识，在bean对象初始化的时候，通过BeanPostProcessor可以完成好多事情。
