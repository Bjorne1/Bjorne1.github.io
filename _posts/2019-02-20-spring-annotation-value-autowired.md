---
layout:     post
title:      Spring Annotation(三)
subtitle:   Srping 属性赋值与自动装配
date:       2019-02-20
author:     WCS
header-img: img/2019-02-20-spring-annotation-value-autowired.jpg
catalog: true
tags:
    - Spring
    - Notes
---

# 前言
Srping属性赋值与自动装配相关内容。

## 一、Spring属性赋值

#### 1、@Value

```
@Configuration
public class MainConfigOfPropertyValues {
    @Bean
    public Person person() {
        return new Person();
    }
}
```  

```
public class Person {
    //使用@Value赋值；
    //1、基本数值
    //2、可以写SpEL； #{}

    @Value("张三")
    private String name;

    //@Value("#{20-2}")
    private int age;

    ....
}
```

#### 2、@PropertySource

```
/**
 * 使用@PropertySource读取外部配置文件中的k/v保存到运行的环境变量中;加载完外部的配置文件以后使用${}取出配置文件的值.
 */
@PropertySource(value = {"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {
    @Bean
    public Person person() {
        return new Person();
    }
}
```  

```
public class Person {
    //使用@Value赋值；
    //3、可以写${}；取出配置文件【properties】中的值（在运行环境变量里面的值）

    @Value("张三")
    private String name;

    //@Value("#{20-2}")
    private int age;

    //person.properties的内容为person.nickName=张小山
    @Value("${person.nickName}")
    private String nickName;

    ....
}
```

## 二、Spring自动装配

#### 1、@Autowired、@Qualifier、@Primary

>Spring自动装配:  
Spring利用依赖注入（DI），完成对IOC容器中中各个组件的依赖关系赋值。
  
* **@Autowired**：自动注入
   * 默认优先按照类型去容器中找对应的组件:**applicationContext.getBean(BookDao.class)**,找到就赋值；
   * 如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找**applicationContext.getBean("bookDao")**；
   * **@Qualifier("bookDao")**：使用 **@Qualifier**指定需要装配的组件的id，而不是使用属性名；
   * **@Primary**：让Spring进行自动装配的时候，默认使用首选的bean；也可以继续使用 **@Qualifier**指定需要装配的bean的名字；
   * 自动装配默认一定要将属性赋值好，如果 **@Autowired BookDao bookDao**，但是在容器中没有找到bookDao就会报错;可以使用 **@Autowired(required=false)**。

```
@Configuration
@ComponentScan({"com.annotation.service", "com.annotation.dao", "com.annotation.controller"})
public class MainConifgOfAutowired {

    @Primary
    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setLabel("2");
        return bookDao;
    }
}
```  

```
@Service
public class BookService {

    @Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService [bookDao=" + bookDao + "]";
    }

}
```  

```
//名字默认是类名首字母小写，也可以通过value指定
@Repository //(value="bookDao3")
public class BookDao {

    private String label = "1";

    public String getLabel() {
        return label;
    }

    public void setLabel(String label) {
        this.label = label;
    }

    @Override
    public String toString() {
        return "BookDao [label=" + label + "]";
    }
}
```  

测试demo:  
```
@Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConifgOfAutowired.class);

        BookService bookService = applicationContext.getBean(BookService.class);
        System.out.println(bookService);

        BookDao bean = applicationContext.getBean(BookDao.class);
        System.out.println(bean);
    }
```

打印结果：  
```
BookService [bookDao=BookDao [label=1]]
BookDao [label=2]
```  
* 因为**BookService**中带有注解 **@Qualifier("bookDao")**,指定组件id为"bookDao"，所以注入的是label = "1"的BookDao
* 而直接获取**BookDao.class**时，MainConifgOfAutowired配置类中注入的BookDao带有注解 **@Primary**，所以注入的是label = "2"的BookDao

#### 2、@Resource、@Inject

* Spring还支持使用 **@Resource(JSR250)** 和 **@Inject(JSR330)** *[java规范的注解]*；
* **@Resource**: 可以和 **@Autowired**一样实现自动装配功能；默认是按照组件名称进行装配的；没有能支持 **@Primary**功能没有支持 **@Autowired（reqiured=false）**；
* **@Inject**: 需要导入javax.inject的包，和**Autowired**的功能一样。没有**required=false**的功能；
* **@Autowired**:Spring是定义的； **@Resource、@Inject**都是java规范。

#### 3、AutowiredAnnotationBeanPostProcessor
* **Autowired,Resource,Inject**都是通过AutowiredAnnotation**BeanPostProcessor**解析完成自动装配功能的。

#### 4、@Autowired标注在其他位置
* *[标注在方法位置]*：@Bean+方法参数；参数从容器中获取;默认不写 **@Autowired**效果是一样的；都能自动装配；

```
	/**
	 * @Bean标注的方法创建对象的时候，方法参数的值从容器中获取
	 * @param car
	 * @return
	 */
	@Bean
	public Color color(@Autowired Car car){
		Color color = new Color();
		color.setCar(car);
		return color;
	}
```  

```
	//标注在方法，Spring容器创建当前对象，就会调用方法，完成赋值；
	//方法使用的参数，自定义类型的值从ioc容器中获取
    @Autowired
	public void setCar(Car car) {
		this.car = car;
	}
```

* *[标在构造器上]*：如果组件只有一个有参构造器，这个有参构造器的 **@Autowired**可以省略，参数位置的组件还是可以自动从容器中获取；
```
@Component
public class Boss {
	private Car car;
	
	//构造器要用的组件，都是从容器中获取
    @Autowired
	public Boss(Car car){
		this.car = car;
		System.out.println("Boss...有参构造器");
	}
}
```  

* 放在参数位置。
> 以上感觉除了@Autowired用的多，@Primary配置多数据源的时候会用到，其他的到目前为止，我都没怎么用啊。

#### 5、Aware
* 自定义组件想要使用Spring容器底层的一些组件（ApplicationContext，BeanFactory，xxx）；
* 自定义组件实现xxxAware；在创建对象的时候，会调用接口规定的方法注入相关组件；
* 把Spring底层一些组件注入到自定义的Bean中；
* **xxxAware**：功能都是用**xxxProcessor**来处理的； **ApplicationContextAware**==》**ApplicationContextAwareProcessor**；底层原理与前一篇文章[Spring Annotation(二)](http://wenchangsheng.com/2019/02/19/spring-annotation-life-cycle/ "Spring Annotation(二)")中分析的是一样的。

```
@Component
public class Red implements ApplicationContextAware, BeanNameAware, EmbeddedValueResolverAware {

    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("传入的ioc：" + applicationContext);
        this.applicationContext = applicationContext;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("当前bean的名字：" + name);
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        String resolveStringValue = resolver.resolveStringValue("你好 ${os.name} 我是 #{20*18}");
        System.out.println("解析的字符串：" + resolveStringValue);
    }

}
```

## 三、不同的环境下的Profile

* **@Profile**注解是Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；
* **@Profile**指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件；
   * 加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中。默认是default环境；
   * 写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效；
   * 没有标注环境标识的bean在，任何环境下都是加载的；

```
@PropertySource("classpath:/dbconfig.properties")
@Configuration
public class MainConfigOfProfile implements EmbeddedValueResolverAware{
    //1、通过@Value从配置文件中获取值
    @Value("${db.user}")
    private String user;

    private StringValueResolver valueResolver;

    private String  driverClass;

    @Bean
    public Yellow yellow(){
        return new Yellow();
    }

    @Profile("test")
    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}")String pwd) throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/gene_app_debug");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}")String pwd) throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/gene_app");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}")String pwd) throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    //2、通过EmbeddedValueResolverAware获取配置文件中的值
    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.valueResolver = resolver;
        driverClass = valueResolver.resolveStringValue("${db.driverClass}");
    }

}
```  

测试：
```
    //1、使用命令行动态参数: 在虚拟机参数位置加载 -Dspring.profiles.active=test
    //2、代码的方式激活某种环境；
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext();
        //1、创建一个applicationContext
        //2、设置需要激活的环境
        applicationContext.getEnvironment().setActiveProfiles("test");
        //3、注册主配置类
        applicationContext.register(MainConfigOfProfile.class);
        //4、启动刷新容器
        applicationContext.refresh();

        String[] namesForType = applicationContext.getBeanNamesForType(DataSource.class);
        for (String string : namesForType) {
            System.out.println(string);
        }

        Yellow bean = applicationContext.getBean(Yellow.class);
        System.out.println(bean);
        applicationContext.close();
    }
```  
dbconfig.properties
```
db.user=root
db.password=root
db.driverClass=com.mysql.jdbc.Driver
```

## 四、总结
通过这几天整理学习笔记下来，原先对于BeanPostProcesser这个后置处理器还不理解，那时候只是单纯看着视频DEBUG走了一遍，并简单用代码测试了一下，简单的内容还好，BeanPostProcesser这种复杂的当时听原理的时候，听完感觉就跟没听一样，那时候也是视频快进2倍看的。现在隔了一段时间，来重新整理这个笔记，一步步将原来的都过一遍，测试了一遍，尤其是对于BeanPostProcesser原理的DEBUG，通过自己一步步DEBUG并画图整理，让我对原先学习的内容也有了更深的认识。