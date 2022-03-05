# 								MyBatis 

## 一、整体流程

#### 1、初始化阶段（基于Spring）

1. 在DataSourceConfig中通过连接池获得DataSource，并生成SqlSessionFactoryBean(生产SqlSessionFactory的工厂Bean)和DataSourceTransactionManager(Spring的事务管理器)；之后通过SqlSessionFactory创建SQLSessionTemplate(代替DefaultSQLSession，是SqlSession的代理对象。先通过SqlSession执行数据库操作，如果没有问题就提交事务，否则进行回滚)。
2. MapperScannerConfigurer继承自BeanDefinitionRegistryPostProcessor，,在Spring启动时，会根据包名扫描（@ComponantScan,@MapperScan），从而得到BeanDefinition。将所属的BeanDefinition交给MapperScannerConfigurer处理，它对每一个BeanDefinition设置相关属性。
3. 在Spring自动装配Mapper时，会通过类型，从步骤二中的candidates中进行选择，从而实现自动装配

#### **2**、执行阶段

1. 类中的接口类型对象，为代理对象。调用接口方法调用的是代理对象的方法--MapperProxy.invoke，如果缓存methodCache（ConcurrentHashMap）中存在该方法，直接返回，否则新建一个并放入缓存。最后根据MapperMethod中sqlType（insert、update...）的不同，执行SqlSession的不同方法。
2. MappedStatement在MapperMethod初始化时，通过ClassName.MethodName从Configuration中读出，并把StatementId存下来。所以重载方法是无效的，因为ClassName.MethodName在一个类中只能找到一个。
3. 在DefaultSqlSession中通过MappedStatement的StatementId查找出Configuration中储存的从xml中解析得到的sql。
4. 在BaseExecutor中，执行query、update(包括insert、update、delete)时，传入MappedStatement和相应方法的参数。如果是查询操作，会构造出相应的CacheKey，并在CachingExecutor中的二级缓存中查找相应结果，如果没有查到，执行实现类的具体方法查询；如果是update，则直接进行查询。
5. 通过StatementHandler执行jdbc的相应方法，并将结果交由resultHandler处理，最后将结果返回。



## 二、缓存

### 1、一级缓存

每个BaseExecutor的实现类中都有一个一级缓存，但是对于Spring来说，每次数据库查询，都是新建了一个SqlSession，而SqlSession和Executor是一对一的关系，所以一级缓存没有用

### 2、二级缓存

​		二级缓存通过CacheExecutor实现。CacheExecutor是抽象类BaseExecutor的实现类，基于装饰者模式实现。代理的对象是BaseExecutor的另外三个实现类，ReuseExecutor、SimpleExecutor、BatchExecutor。

​		二级缓存的初始化在创建DefaultSQLSession中执行，如果程序开启了二级缓存，就会在包装Executor。

​		二级缓存的读取是建立在MappedStatement的基础上的，CacheKey也是通过MappedStatement算出的。他是Mapper级别的，如果该Mapper执行了其中的update方法，缓存就会被清空。

