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


## 四、AOP流程


## 五、总结