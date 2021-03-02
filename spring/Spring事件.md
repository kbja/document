## 									Spring事件

#### 一、基础知识

​		事件监听器ApplicationListener继承自java.util.EventListener，而事件主体ApplicationEvent继承自java.util.EventObject。使用观察者模式（发布订阅），将所有的EventListener注册到AbstractApplicationEventMulticaster，等到事件发送时，通过事件类型找到相应的监听器，发送事件。



#### 二、消息监听器

实现消息监听器有两种方式，第一种是接口方式，继承ApplicationListener接口，在类上标注@Component，在接口方法onApplicationEvent(E event)中处理消息；第二种注解方式，在方法上标注@EventListener，方法参数可以接受0到1个参数。

1. 接口方式实现

   和其他的Bean的解析方式类似，在invokeBeanFactoryPostProcessors会扫描bean，并注册到BeanDefinitionRegistry中，之后再根据定义实例化相应的bean。

2. 注解方式实现

   在SpringFramework实例化所有bean后，即DefaultListableBeanFactory#preInstantiateSingletons()最后，会执行所有SmartInitializingSingleton#afterSingletonsInstantiated()。EventListenerMethodProcessor是SmartInitializingSingleton的实现类，在回调方法afterSingletonsInstantiated中，会对所有bean进行遍历，筛选出来包含@EventListener的方法，对每一个方法创建一个ApplicationListener，并将这个ApplicationListener注册。



#### 三、消息发布

​		通过ApplicationEventPublisher进行消息的发布，而AbstractApplicationContext实现了这个接口，所以它所有的子类都可以实现消息的发布。顺带一提，ApplicationEventPublisherAware接口中传入的ApplicationEventPublisher对象，就是ApplicationContext。

​		AbstractApplicationContext是通过SimpleApplicationEventMulticaster实现消息的发布的。所有的监听器都注册在这个类中，通过传入Event，根据Event类型去查询出已注册的监听此类事件的监听器，将事件发布。



#### 四、异步事件、事务事件

​		异步事件通过增加@Async注解进行异步，事务事件通过增加@TransactionalEventListener注解在事务不同状态时收到消息。