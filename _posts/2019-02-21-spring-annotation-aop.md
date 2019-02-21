---
layout:     post
title:      Spring Annotation(四)
subtitle:   Srping AOP
date:       2019-02-21
author:     WCS
header-img: img/2019-02-21-spring-annotation-aop.jpg
catalog: true
tags:
    - Spring
    - AOP
    - Notes
---

# 前言
这是一篇Spirng AOP相关的文章，包括Spring AOP的使用及原理解析。

## 一、AOP：【动态代理】
>AOP【动态代理】:  
指在程序运行期间动态的将某段代码切入到指定方法指定位置进行运行的编程方式。

#### 1、导入aop模块
* Spring AOP：(spring-aspects)，Maven引入相关的依赖。

#### 2、定义一个业务逻辑类
* 在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）。

```
public class MathCalculator {

	public int div(int i,int j){
		System.out.println("MathCalculator...div...");
		return i/j;	
	}

}
```

#### 3、定义一个日志切面类
* 切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；
   * 通知方法：
      * 前置通知(@Before)：logStart：在目标方法(div)运行之前运行
      * 后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
      * 返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
      * 异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
      * 环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()），在多数据源的时候有用到过，到时候再写一篇相关的文章。

#### 4、给切面类的目标方法标注何时何地运行（通知注解）

#### 5、告诉Spring哪个类是切面类

```
//@Aspect,告诉Spring这是一个切面类
@Aspect
public class LogAspects {

    //抽取公共的切入点表达式
    //1、本类引用
    //2、其他的切面引用
    @Pointcut("execution(public int com.annotation.aop.MathCalculator.*(..))")
    public void pointCut(){};

    //@Before在目标方法之前切入；切入点表达式（指定在哪个方法切入）
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println(""+joinPoint.getSignature().getName()+"运行。。。@Before:参数列表是：{"+Arrays.asList(args)+"}");
    }

    @After("com.annotation.aop.LogAspects.pointCut()")
    public void logEnd(JoinPoint joinPoint){
        System.out.println(""+joinPoint.getSignature().getName()+"结束。。。@After");
    }

    //JoinPoint一定要出现在参数表的第一位
    @AfterReturning(value="pointCut()",returning="result")
    public void logReturn(JoinPoint joinPoint,Object result){
        System.out.println(""+joinPoint.getSignature().getName()+"正常返回。。。@AfterReturning:运行结果：{"+result+"}");
    }

    @AfterThrowing(value="pointCut()",throwing="exception")
    public void logException(JoinPoint joinPoint,Exception exception){
        System.out.println(""+joinPoint.getSignature().getName()+"异常。。。异常信息：{"+exception+"}");
    }

}
```

#### 6、将切面类和业务逻辑类（目标方法所在类）都加入到容器中

#### 7、给配置类中加 @EnableAspectJAutoProxy【开启基于注解的aop模式】

```
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {

    //业务逻辑类加入容器中
    @Bean
    public MathCalculator calculator(){
        return new MathCalculator();
    }

    //切面类加入到容器中
    @Bean
    public LogAspects logAspects(){
        return new LogAspects();
    }
}
```

#### 8、测试

```
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class);

        //不要自己创建对象
//		MathCalculator mathCalculator = new MathCalculator();

        MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);
        mathCalculator.div(1, 1);

        applicationContext.close();
    }
```  
测试结果:
```
div运行。。。@Before:参数列表是：{[1, 1]}
MathCalculator...div...
div结束。。。@After
div正常返回。。。@AfterReturning:运行结果：{1}
```  
如果将除数改为0:
```
div运行。。。@Before:参数列表是：{[1, 0]}
MathCalculator...div...
div结束。。。@After
div异常。。。异常信息：{java.lang.ArithmeticException: / by zero}
```

## 二、主要的三步

* 将业务逻辑组件和切面类都加入到容器中；告诉Spring哪个是切面类（@Aspect）
* 在切面类上的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式）
* 开启基于注解的aop模式；@EnableAspectJAutoProxy

## 三、AOP原理

* @EnableAspectJAutoProxy是什么？
   * @Import(AspectJAutoProxyRegistrar.class)：给容器中导入AspectJAutoProxyRegistrar
   * 利用AspectJAutoProxyRegistrar自定义给容器中注册bean；BeanDefinetion
   * internalAutoProxyCreator=AnnotationAwareAspectJAutoProxyCreator
   * 给容器中注册一个AnnotationAwareAspectJAutoProxyCreator

* 继承结构：AnnotationAwareAspectJAutoProxyCreator->AspectJAwareAdvisorAutoProxyCreator->AbstractAdvisorAutoProxyCreator->AbstractAutoProxyCreator implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
   * 关注后置处理器（在bean初始化完成前后做事情）、自动装配BeanFactory
   * AbstractAutoProxyCreator.setBeanFactory();
   * AbstractAutoProxyCreator.postProcessBeforeInstantiation() 后置处理器的逻辑;
   * AbstractAdvisorAutoProxyCreator.setBeanFactory()->initBeanFactory();
   * AnnotationAwareAspectJAutoProxyCreator.initBeanFactory();

## 四、AOP流程

* 传入配置类，创建ioc容器
* 注册配置类，调用refresh（）刷新容器；
* registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；
   * 先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor
   * 给容器中加别的BeanPostProcessor
   * 优先注册实现了PriorityOrdered接口的BeanPostProcessor；
   * 再给容器中注册实现了Ordered接口的BeanPostProcessor；
   * 注册没实现优先级接口的BeanPostProcessor；
   * 注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】
      * 创建Bean的实例
      * populateBean；给bean的各种属性赋值
      * initializeBean：初始化bean；
         * invokeAwareMethods()：处理Aware接口的方法回调
         * applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization()
         * invokeInitMethods()；执行自定义的初始化方法
         * applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization()
      * BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder
   * 把BeanPostProcessor注册到BeanFactory中:beanFactory.addBeanPostProcessor(postProcessor);
  
 以上是创建和注册AnnotationAwareAspectJAutoProxyCreator的过程  
  
* finishBeanFactoryInitialization(beanFactory);完成BeanFactory初始化工作；创建剩下的单实例bean;(AnnotationAwareAspectJAutoProxyCreator => InstantiationAwareBeanPostProcessor)
   * 遍历获取容器中所有的Bean，依次创建对象getBean(beanName);getBean->doGetBean()->getSingleton()->
   * 创建bean
      * 【AnnotationAwareAspectJAutoProxyCreator在所有bean创建之前会有一个拦截，InstantiationAwareBeanPostProcessor，会调用postProcessBeforeInstantiation()】
      * 先从缓存中获取当前bean，如果能获取到，说明bean是之前被创建过的，直接使用，否则再创建；只要创建好的Bean都会被缓存起来
      * createBean（）;创建bean；
         * AnnotationAwareAspectJAutoProxyCreator 会在任何bean创建之前先尝试返回bean的实例
         * 【BeanPostProcessor是在Bean对象创建完成初始化前后调用的】
         * 【InstantiationAwareBeanPostProcessor是在创建Bean实例之前先尝试用后置处理器返回对象的】
         * resolveBeforeInstantiation(beanName, mbdToUse);解析BeforeInstantiation。希望后置处理器在此能返回一个代理对象；如果能返回代理对象就使用，如果不能就继续；
            * 后置处理器先尝试返回对象；bean = applyBeanPostProcessorsBeforeInstantiation()：拿到所有后置处理器，如果是InstantiationAwareBeanPostProcessor;就执行postProcessBeforeInstantiation;if (bean != null) {bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);}
         * doCreateBean(beanName, mbdToUse, args);真正的去创建一个bean实例；和之前一样；
  
* AnnotationAwareAspectJAutoProxyCreator【InstantiationAwareBeanPostProcessor】	的作用：
   * 每一个bean创建之前，调用postProcessBeforeInstantiation()；
      * 判断当前bean是否在advisedBeans中（保存了所有需要增强bean）
      * 判断当前bean是否是基础类型的Advice、Pointcut、Advisor、AopInfrastructureBean，或者是否是切面（@Aspect）
      * 是否需要跳过
         * 获取候选的增强器（切面里面的通知方法）【List<Advisor> candidateAdvisors】;每一个封装的通知方法的增强器是 InstantiationModelAwarePointcutAdvisor；判断每一个增强器是否是 AspectJPointcutAdvisor 类型的；返回true
         * 永远返回false
   * 创建对象;postProcessAfterInitialization；return wrapIfNecessary(bean, beanName, cacheKey);//包装如果需要的情况下;
      * 获取当前bean的所有增强器（通知方法）  Object[]  specificInterceptors
         * 找到候选的所有的增强器（找哪些通知方法是需要切入当前bean方法的）
         * 获取到能在bean使用的增强器。
         * 给增强器排序
      * 保存当前bean在advisedBeans中；
      * 如果当前bean需要增强，创建当前bean的代理对象；
         * 获取所有增强器（通知方法）
         * 保存到proxyFactory
         * 创建代理对象：Spring自动决定:JdkDynamicAopProxy(config);jdk动态代理；ObjenesisCglibAopProxy(config);cglib的动态代理；
      * 给容器中返回当前组件使用cglib增强了的代理对象；
      * 以后容器中获取到的就是这个组件的代理对象，执行目标方法的时候，代理对象就会执行通知方法的流程；
   * 目标方法执行;容器中保存了组件的代理对象（cglib增强后的对象），这个对象里面保存了详细信息（比如增强器，目标对象，xxx）；
      * CglibAopProxy.intercept();拦截目标方法的执行
      * 根据ProxyFactory对象获取将要执行的目标方法拦截器链；
         * List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
         * List<Object> interceptorList保存所有拦截器 5个，一个默认的ExposeInvocationInterceptor 和 4个增强器；
         * 遍历所有的增强器，将其转为Interceptor；registry.getInterceptors(advisor);
         * 将增强器转为List<MethodInterceptor>；
            * 如果是MethodInterceptor，直接加入到集合中;
            * 如果不是，使用AdvisorAdapter将增强器转为MethodInterceptor；
            * 转换完成返回MethodInterceptor数组；
      * 如果没有拦截器链，直接执行目标方法;拦截器链（每一个通知方法又被包装为方法拦截器，利用MethodInterceptor机制）
      * 如果有拦截器链，把需要执行的目标对象，目标方法，拦截器链等信息传入创建一个 CglibMethodInvocation 对象，并调用 Object retVal =  mi.proceed();
      * 拦截器链的触发过程;
         * 如果没有拦截器执行执行目标方法，或者拦截器的索引和拦截器数组-1大小一样（执行到了最后一个拦截器）执行目标方法；
         * 链式获取每一个拦截器，拦截器执行invoke方法，每一个拦截器等待下一个拦截器执行完成返回以后再来执行；拦截器链的机制，保证通知方法与目标方法的执行顺序；


## 五、总结

* @EnableAspectJAutoProxy 开启AOP功能
* @EnableAspectJAutoProxy 会给容器中注册一个组件 AnnotationAwareAspectJAutoProxyCreator
* AnnotationAwareAspectJAutoProxyCreator是一个后置处理器；
* 容器的创建流程：
   * registerBeanPostProcessors()注册后置处理器；创建AnnotationAwareAspectJAutoProxyCreator对象
   * finishBeanFactoryInitialization()初始化剩下的单实例bean
      * 创建业务逻辑组件和切面组件
      * AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程
      * 组件创建完之后，判断组件是否需要增强:是：切面的通知方法，包装成增强器（Advisor）;给业务逻辑组件创建一个代理对象（cglib）；
* 执行目标方法：
   * 代理对象执行目标方法
   * CglibAopProxy.intercept();
      * 得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor）
      * 利用拦截器的链式机制，依次进入每一个拦截器进行执行；
      * 效果：
         * 正常执行：前置通知-》目标方法-》后置通知-》返回通知
         * 出现异常：前置通知-》目标方法-》后置通知-》异常通知
* **原理分析技巧：看给容器中注册了什么组件，这个组件什么时候工作，这个组件的功能是什么？（eg:@EnableAspectJAutoProxy）**