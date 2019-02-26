---
layout:     post
title:      Spring Annotation(五)
subtitle:   Srping Summarized
date:       2019-02-25
author:     WCS
header-img: img/2019-02-25-spring-annotation-summarized.jpg
catalog: true
tags:
    - Spring
    - Notes
---

# 前言
这是一篇Spirng容器整个创建流程的总结文章。

## 一、容器刷新
>new AnnotationConfigApplicationContext().refresh();

## 二、刷新前的预处理
>prepareRefresh();  

1、初始化一些属性设置;子类自定义个性化的属性设置方法;  
`initPropertySources();`  
2、检验属性的合法等;  
`getEnvironment().validateRequiredProperties();`  
3、保存容器中的一些早期的事件;  
`earlyApplicationEvents= new LinkedHashSet<ApplicationEvent>();`

## 三、告诉子类刷新其内部的bean factory
>obtainFreshBeanFactory();  // Tell the subclass to refresh the internal bean factory.

1、**创建**刷新BeanFactory  
`refreshBeanFactory();`  

* (1)、创建了`this.beanFactory = new DefaultListableBeanFactory();  `
* (2)、给context容器设置唯一id;  

2、返回刚才GenericApplicationContext创建的BeanFactory对象;`getBeanFactory();`  

3、将创建的BeanFactory **DefaultListableBeanFactory**返回;

## 四、准备bean factory以便在当前容器中使用
>prepareBeanFactory(beanFactory);  

1、设置BeanFactory的类加载器、支持表达式解析器...  
```
beanFactory.setBeanClassLoader(getClassLoader());
beanFactory.setBeanExpressionResolver(...);
```  

2、添加部分BeanPostProcessor **ApplicationContextAwareProcessor**  
`beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));`  

3、设置忽略的自动装配的接口  
```
beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
```  

4、注册可以解析的自动装配；我们能直接在任何组件中自动注入  
```
beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
beanFactory.registerResolvableDependency(ResourceLoader.class, this);
beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
beanFactory.registerResolvableDependency(ApplicationContext.class, this);
```  

eg:  
```
@Component
public class Dog implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Dog(){
        System.out.println("dog constructor...");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        // TODO Auto-generated method stub
        this.applicationContext = applicationContext;
    }

}
```  

5、添加BeanPostProcessor **ApplicationListenerDetector**  
```
beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));
```  

6、添加编译时的AspectJ  
```
// Detect a LoadTimeWeaver and prepare for weaving, if found.
if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
   beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
   // Set a temporary ClassLoader for type matching.
   beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
}
```  

7、给BeanFactory中注册一些能用的组件  
```
//		environment【ConfigurableEnvironment】
if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
   beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
}
//		systemProperties【Map<String, Object>】
if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
   beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
}
//		systemEnvironment【Map<String, Object>】
if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
   beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
}
```  

## 五、BeanFactory准备完成后进行的后置处理工作
>postProcessBeanFactory(beanFactory);  

```
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {}
```  
子类通过重写这个方法来在BeanFactory创建并预准备完成以后做进一步的设置。  

********
**以上是BeanFactory的创建及预准备工作！**  

## 六、执行BeanFactoryPostProcessor的方法
>invokeBeanFactoryPostProcessors(beanFactory);
>>BeanFactoryPostProcessor：BeanFactory的后置处理器。在BeanFactory标准初始化之后执行的；
>>>两个接口：BeanFactoryPostProcessor、BeanDefinitionRegistryPostProcessor  

1、先执行BeanDefinitionRegistryPostProcessor  
* (1)、获取所有的BeanDefinitionRegistryPostProcessor  
* (2)、先执行实现了PriorityOrdered优先级接口的BeanDefinitionRegistryPostProcessor,postProcessor.postProcessBeanDefinitionRegistry(registry)  
* (3)、再执行实现了Ordered顺序接口的BeanDefinitionRegistryPostProcessor,postProcessor.postProcessBeanDefinitionRegistry(registry)  
* (4)、最后执行没有实现任何优先级或者是顺序接口的BeanDefinitionRegistryPostProcessor,postProcessor.postProcessBeanDefinitionRegistry(registry)  

2、再执行BeanFactoryPostProcessor的方法  
* (1)、获取所有的BeanFactoryPostProcessor  
* (2)、先执行实现了PriorityOrdered优先级接口的BeanFactoryPostProcessor,postProcessor.postProcessBeanFactory()  
* (3)、再执行实现了Ordered顺序接口的BeanFactoryPostProcessor,postProcessor.postProcessBeanFactory()  
* (4)、最后执行没有实现任何优先级或者是顺序接口的BeanFactoryPostProcessor,postProcessor.postProcessBeanFactory()  

## 七、注册BeanPostProcessor(Bean的后置处理器)
>registerBeanPostProcessors(beanFactory);
>>不同接口类型的BeanPostProcessor;在Bean创建前后的执行时机是不一样的
>>>BeanPostProcessor、DestructionAwareBeanPostProcessor、InstantiationAwareBeanPostProcessor、SmartInstantiationAwareBeanPostProcessor、MergedBeanDefinitionPostProcessor【internalPostProcessors】  

1、获取所有的 BeanPostProcessor;后置处理器都默认可以通过PriorityOrdered、Ordered接口来指定优先级  

2、先注册PriorityOrdered优先级接口的BeanPostProcessor,把每一个BeanPostProcessor添加到BeanFactory中,`beanFactory.addBeanPostProcessor(postProcessor);`  

3、再注册Ordered接口的  

4、最后注册没有实现任何优先级接口的  

5、最终注册MergedBeanDefinitionPostProcessor  

6、注册一个ApplicationListenerDetector；来在Bean创建完成后检查是否是ApplicationListener，如果是则`applicationContext.addApplicationListener((ApplicationListener<?>) bean);`  

## 八、初始化MessageSource组件
>initMessageSource();
>>做国际化功能；消息绑定，消息解析  

1、获取BeanFactory  

2、看容器中是否有id为messageSource的，类型是MessageSource的组件  
* (1)有，赋值给messageSource  
* (2)没有，自己创建一个DelegatingMessageSource  
* (3)MessageSource：取出国际化配置文件中的某个key的值，能按照区域信息获取  

3、把创建好的MessageSource注册在容器中，以后获取国际化配置文件的值的时候，可以自动注入MessageSource  
```
beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);	
MessageSource.getMessage(String code, Object[] args, String defaultMessage, Locale locale);
```  

## 九、初始化事件派发器
>initApplicationEventMulticaster();  

1、获取BeanFactory  

2、从BeanFactory中获取applicationEventMulticaster的ApplicationEventMulticaster  

3、如果上一步没有配置；创建一个SimpleApplicationEventMulticaster  

4、将创建的ApplicationEventMulticaster添加到BeanFactory中，以后其他组件直接自动注入  

## 十、在子容器中初始化其他特殊的bean
>onRefresh();// Initialize other special beans in specific context subclasses.
>>子类重写这个方法，在容器刷新的时候可以自定义逻辑  

## 十一、检查容器中ApplicationListener并注册
>registerListeners();// Check for listener beans and register them.  
1、从容器中拿到所有的ApplicationListener  
```
// Register statically specified listeners first.
for (ApplicationListener<?> listener : getApplicationListeners()) {
   getApplicationEventMulticaster().addApplicationListener(listener);
}
```  

2、将每个监听器添加到事件派发器中  
```
// Do not initialize FactoryBeans here: We need to leave all regular beans
// uninitialized to let post-processors apply to them!
String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
for (String listenerBeanName : listenerBeanNames) {
   getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
}
```  

3、派发之前步骤产生的事件  
```
// Publish early application events now that we finally have a multicaster...
Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
this.earlyApplicationEvents = null;
if (earlyEventsToProcess != null) {
   for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
      getApplicationEventMulticaster().multicastEvent(earlyEvent);
   }
}
```  

## 十二、初始化所有剩下的单实例bean
>finishBeanFactoryInitialization(beanFactory);// Instantiate all remaining (non-lazy-init) singletons.  

1、初始化后剩下的单实例bean  
>beanFactory.preInstantiateSingletons();  

(1)、获取容器中的所有Bean，依次进行初始化和创建对象  

(2)、获取Bean的定义信息；RootBeanDefinition  

(3)、Bean不是抽象的，是单实例的，不是懒加载  
①判断是否是FactoryBean；是否是实现FactoryBean接口的Bean  

②不是工厂Bean。利用getBean(beanName);创建对象  

* A、getBean(beanName)； ioc.getBean();  
* B、doGetBean(name, null, null, false);  
* C、先获取缓存中保存的单实例Bean。如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来），从`private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);`获取的  
* D、缓存中获取不到，开始Bean的创建对象流程`doGetBean()`  
* E、标记当前bean已经被创建`if (!typeCheckOnly) {markBeanAsCreated(beanName);}`  
* F、获取Bean的定义信息  
* G、**获取当前Bean依赖的其他Bean;如果有按照getBean()把依赖的Bean先创建出来**  
`String[] dependsOn = mbd.getDependsOn();`  

* H、启动单实例Bean的创建流程  

   * a、createBean(beanName, mbd, args);  
   * b、Object bean = resolveBeforeInstantiation(beanName, mbdToUse);让BeanPostProcessor先拦截返回代理对象；**InstantiationAwareBeanPostProcessor**：提前执行  
先触发：postProcessBeforeInstantiation();  
如果有返回值：触发postProcessAfterInitialization();  
   * c、如果前面的InstantiationAwareBeanPostProcessor没有返回代理对象；调用d
   * d、`Object beanInstance = doCreateBean(beanName, mbdToUse, args);`创建Bean  

      * (A)、**创建Bean实例**`createBeanInstance(beanName, mbd, args);`  
利用工厂方法或者对象的构造器创建出Bean实例。
      * (B)、`applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);`  
调用MergedBeanDefinitionPostProcessor的`postProcessMergedBeanDefinition(mbd, beanType, beanName);`  

      * (C)、**Bean属性赋值**`populateBean(beanName, mbd, instanceWrapper);`  

         * (a)赋值之前：  
拿到InstantiationAwareBeanPostProcessor后置处理器`postProcessAfterInstantiation();`  
拿到InstantiationAwareBeanPostProcessor后置处理器`postProcessPropertyValues();`  
         * (b)应用Bean属性的值；为属性利用setter方法等进行赋值  
`applyPropertyValues(beanName, mbd, bw, pvs);`  

      * (D)、**Bean初始化**`initializeBean(beanName, exposedObject, mbd);`  

         * (a)、**执行Aware接口方法**`invokeAwareMethods(beanName, bean)`;执行xxxAware接口的方法  
`BeanNameAware\BeanClassLoaderAware\BeanFactoryAware`  
         * (b)、**执行后置处理器初始化之前**`applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);`  
`BeanPostProcessor.postProcessBeforeInitialization();`  
         * (c)、**执行初始化方法**`invokeInitMethods(beanName, wrappedBean, mbd);`  
是否是InitializingBean接口的实现；执行接口规定的初始化；是否自定义初始化方法；  
         * (d)、**执行后置处理器初始化之后**`applyBeanPostProcessorsAfterInitialization`  
`BeanPostProcessor.postProcessAfterInitialization();`  

      * (E)、注册Bean的销毁方法

   * e、将创建的Bean添加到缓存中singletonObjects  

(4)、所有Bean都利用getBean创建完成以后，检查所有的Bean是否是SmartInitializingSingleton接口的，如果是，就执行afterSingletonsInstantiated()  

*********
ioc容器就是这些Map；很多的Map里面保存了单实例Bean，环境信息。。。；  

## 十三、IOC容器就创建完成
>finishRefresh();
>>完成BeanFactory的初始化创建工作  

1、初始化和生命周期有关的后置处理器`initLifecycleProcessor();`  
LifecycleProcessor,默认从容器中找是否有lifecycleProcessor的组件**LifecycleProcessor**；如果没有`new DefaultLifecycleProcessor();`加入到容器；  
写一个LifecycleProcessor的实现类，可以在BeanFactory
```
void onRefresh();
void onClose();
```  

2、拿到前面定义的生命周期处理器(BeanFactory);回调onRefresh()  
`getLifecycleProcessor().onRefresh();`  

3、发布容器刷新完成事件  
`publishEvent(new ContextRefreshedEvent(this));`  

4、liveBeansView.registerApplicationContext(this);  

## 十四、总结
1、Spring容器在启动的时候，先会保存所有注册进来的Bean的定义信息  
(1)、xml注册bean；<bean>   
(2)、注解注册Bean；@Service、@Component、@Bean、xxx  

2、Spring容器会合适的时机创建这些Bean  
(1)、用到这个bean的时候；利用getBean创建bean；创建好以后保存在容器中  
(2)、统一创建剩下所有的bean的时候；`finishBeanFactoryInitialization();`  

3、后置处理器**BeanPostProcessor**  
>每一个bean创建完成，都会使用各种后置处理器进行处理；来增强bean的功能
>>AutowiredAnnotationBeanPostProcessor:处理自动注入  
AnnotationAwareAspectJAutoProxyCreator:来做AOP功能；  
xxx....  
增强的功能注解：  
AsyncAnnotationBeanPostProcessor  

4、事件驱动模型  
ApplicationListener事件监听  
ApplicationEventMulticaster事件派发  