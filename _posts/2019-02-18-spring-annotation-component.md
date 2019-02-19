---
layout:     post
title:      Spring Annotation(一)
subtitle:   Srping Component Registry
date:       2019-02-18
author:     WCS
header-img: img/2019-02-18-spring-annotation-component.jpg
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

```
@Configuration
public class MainConfig2 {

    /**
     * @Conditional({Condition}) ： 按照一定的条件进行判断，满足条件给容器中注册bean
     * <p>
     * 如果系统是windows，给容器中注册("bill")
     * 如果是linux系统，给容器中注册("linus")
     */
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates", 62);
    }

    @Conditional(LinuxCondition.class)
    @Bean("linus")
    public Person person02() {
        return new Person("linus", 48);
    }
}
```  

```
//判断是否windows系统
public class WindowsCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if (property.contains("Windows")) {
            return true;
        }
        return false;
    }

}
```

## 七、@Import

```
@Configuration
//@Import导入组件，id默认是组件的全类名
//导入Color和Red对象
@Import({Color.class, Red.class})
public class MainConfig2 {
    ...
}
```

## 八、@MyImportSelector

```
//导入Color和Red对象,MyImportSelector导入了Blue和Yellow对象
@Import({Color.class, Red.class, MyImportSelector.class})
public class MainConfig2 {
    ...
}
```  

```
/**
 * 自定义逻辑返回需要导入的组件
 */
public class MyImportSelector implements ImportSelector {

    /**
     * 返回值，就是到导入到容器中的组件全类名
     * AnnotationMetadata:当前标注@Import注解的类的所有注解信息
     */
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //importingClassMetadata
        //方法不要返回null值
        return new String[]{"com.annotation.bean.Blue", "com.annotation.bean.Yellow"};
    }

}
```

## 九、@MyImportBeanDefinitionRegistrar

```
@Configuration
//导入Color和Red对象,MyImportSelector导入了Blue和Yellow对象
//MyImportBeanDefinitionRegistrar导入了RainBow对象
@Import({Color.class, Red.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
public class MainConfig2 {
    ...
}
```

```
/**
 * 手动注册bean到容器中
 */
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    /**
     * AnnotationMetadata：当前类的注解信息
     * BeanDefinitionRegistry:BeanDefinition注册类；
     * 把所有需要添加到容器中的bean；调用
     * BeanDefinitionRegistry.registerBeanDefinition手工注册进来
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean definition = registry.containsBeanDefinition("com.annotation.bean.Red");
        boolean definition2 = registry.containsBeanDefinition("com.annotation.bean.Blue");
        if (definition && definition2) {
            //指定Bean定义信息；（Bean的类型，Bean。。。）
            RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
            //注册一个Bean，指定bean名
            registry.registerBeanDefinition("rainBow", beanDefinition);
        }
    }

}
```

## 十、FactoryBean注册组件

```
@Configuration
public class MainConfig2 {

    @Bean
    public ColorFactoryBean colorFactoryBean() {
        return new ColorFactoryBean();
    }

}
```  

```
/**
 * 创建一个Spring定义的FactoryBean
 */
public class ColorFactoryBean implements FactoryBean<Color> {

    /**
     * 返回一个Color对象，这个对象会添加到容器中
     */
    @Override
    public Color getObject() {
        System.out.println("ColorFactoryBean...getObject...");
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }


    /**
     * true：这个bean是单实例，在容器中保存一份
     * false：多实例，每次获取都会创建一个新的bean；
     */
    @Override
    public boolean isSingleton() {
        return false;
    }

}
```

## 十一、总结

#### 给容器中注册组件方法

* 包扫描+组件标注注解（**@Controller**/**@Service**/**@Repository**/**@Component**）*[自己写的类]*  
* **@Bean***[导入的第三方包里面的组件]*  
* **@Import***[快速给容器中导入一个组件]*  
   * **@Import**(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名  
   * **ImportSelector**:返回需要导入的组件的全类名数组；  
   * **ImportBeanDefinitionRegistrar**:手动注册bean到容器中  
* 使用Spring提供的 **FactoryBean**（工厂Bean）;  
   * 默认获取到的是工厂bean调用getObject创建的对象  
   * 要获取工厂Bean本身，我们需要给id前面加一个&  