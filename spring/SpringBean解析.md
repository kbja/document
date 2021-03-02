## 										SpringBean解析

### 一、入口

SpringBean解析的入口是AbstractApplicationContext.refresh()调用invokeBeanFactoryPostProcessors()，该方法是调用所有BeanFactoryPostProcessors，允许对所有Bean的定义进行修改。



#### 二、解析步骤

##### 第一部分：SpringApplication（SpringBoot）

在main函数中，执行了SpringApplication.run(source.class, args)，其中的source在SpringBoot创建ApplicationContext时，被注册进了BeanDefinitionRegistry。而main函数所在类会包含@SpringBootApplication/@ComponentScan注解，两个注解中会包含需要扫描的包名，之后SpringFramework就通过这个包名，解析所有Bean的定义。



##### 第二部分：AbstractApplicationContext(SpringFramework)

在AbstractApplicationContext中的onRefresh中，有invokeBeanFactoryPostProcessors方法，内部会调用PostProcessorRegistrationDelegate，执行所有的BeanFactoryPostProcessors相关回调。



##### 第三部分：ConfigurationClassPostProcessor

BeanDefinitionRegistryPostProcessor是继承自BeanFactoryPostProcessors的接口，可以更好的对注册/修改bean。而ConfigurationClassPostProcessor实现了BeanDefinitionRegistryPostProcessor接口，在接口方法对所有注册的@Configuration/@Component/@Bean进行注册，在步骤1中，source被注册了，同时main函数所在类被标注了@SpringBootApplication(其内部被@Component注解了)，所以之后会对开发者所有的bean进行扫描。

在被@Configuration标注的类中，会有@ComponentScan/@Import/@ImportSelector

/@ImportBeanDefinitionRegistrar等注解，这些注解都是为了导入相关的类。

ConfigurationClassPostProcessor会对这些注解进行判断，根据配置并导入所有的类。

**SpringFramework中的@Enable注解的原理就是这种方式。**