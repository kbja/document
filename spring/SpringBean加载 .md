## 																		SpringBean加载

## 一、入口

在AbstractApplicationContext的refresh方法中的最后调用finishBeanFactoryInitialization()，spring已经将所有bean的定义、各种Processor加载，只差将bean进行实例化。此时执行beanFactory.preInstantiateSingletons()，beanFactory是默认实现DefaultListableBeanFactory，既通过BeanDefinitionRegistry接口，实现了bean的注册功能；也通过ListableBeanFactory接口实现了bean的创建功能。



## 二、加载步骤

具体的实现位于AbstractBeanFactory的doGetBean

#### 第一部分 检查

1. **将beanName转化成合法的形式**

* 如果是FactoryBean(生产Bean的工厂)，则前缀为&，去掉前缀。

* 之后将所有的别名转换成Bean的真实名称，存在有别名的情况

2. **如果是正在初始化的单例，直接从缓存中返回**
3. **检查原型模式是否存在循环依赖**

* 所有当前线程创建的原型类型的bean都会被存到一个ThreadLocal中
* 每次创建bean时都会从ThreadLocal中读取，有可能是一个String，有可能是一个Set<String> 检查当前创建的bean是否等于String或存在于Set。true则表示发生了循环依赖。
* ThreadLocal的写入位于AbstractBeanFactory的doGetBean方法最后，将当前正在创建的beanName写入；创建成功之后，会删除。
* 对于原型模式，不管是构造依赖，还是设值依赖，Spring都无法解决，会报BeanCreationException。但是对于单例模式，是可以支持设值依赖的。**在单例对象创建后，提前暴露单例对象，以实现设值。暴露的单例对象存在singletonsCurrentlyInCreation中。具体见AbstractBeanFactory.getSingleton()**

4. 查看ParentFactory是否存在Bean定义

<font color="red">需跟进，到底ParentFactory是个啥</font>



#### 第二部分 

5. **将定义合并**

* Spring读取Bean的定义是GenericBeanDefinition，其中的父类信息只有parentName，这里需要将增量信息合并成全量信息，向上遍历父类，将所有父类的信息合并到RootBeanDefinition。
* <font color="red">GenericBeanDefinition has the advantage that it allows to dynamically define</font> 所以开始时使用GenericBeanDefinition，只有在创建bean时才使用RootBeanDefinition

6. 检查BeanDefinition中dependsOn是否存在循环依赖

* depend-on用来表示一个Bean的实例化依靠另一个Bean先实例化。如果在一个bean A上定义了depend-on B那么就表示：A 实例化前先实例化 B。

#### 第三部分

7. 创建Bean

* 如果Bean是单例，则通过一个Set记录当前创建的单例Bean对象，如果对象已经存在，则说明存在循环依赖。在创建前检查，在创建后删除。
* 如果是Bean是原型，则执行第一部分中的循环依赖检查。
* 实际执行AbstractAutowireCapableBeanFactory.doCreateBean()执行Bean的创建。
  * 在实际创建中，又会分几种形式
  * 通过工厂方法构建
  * 通过有参构造函数构建
  * 通过无参构造函数构建
* 在创建后，会执行AbstractAutowireCapableBeanFactory.populateBean()对Bean的属性进行填充
* 将bean的类型进行转化并返回。

## 三、创建bean的三级缓存

### 1. 三级缓存分别是什么

**第一级** 实例化好的bean    singletonObjects

**第二级** 因循环依赖提前暴露的bean  earlySingletonObjects

**第三极** 提前暴露的ObjectFactory。通过getObject方法得到bean的代理对象（预定义的BeanPostProcessor）。 singletonFactories

 
### 2. 没有第三级缓存行不行

**不行**
正常逻辑下，spring是在bean初始化完成后才创建代理。
当出现循环依赖时，需要提前对bean进行注入，否则会导致注入的对象不是代理对象。此时需要通过第三级缓存创建代理对象，

