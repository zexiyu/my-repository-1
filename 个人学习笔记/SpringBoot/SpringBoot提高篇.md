

分布式和RPC



微服务：将一个大的应用模块拆分成多个微小的模块，每个模块间可以进行远程调用

分布式：有微服务必然产生分布式，即：将各个微小模块分配到不同的机器上

## 分布式理论

### 什么是分布式系统

在《分布式系统原理与范型》一书中有如下定义：“分布式系统是若干独立计算机的集合，这些计算机对 于用户来说就像单个相关系统”； 

分布式系统是由一组通过网络进行通信、为了完成共同的任务而协调工作的计算机节点组成的系统。分 布式系统的出现是为了用廉价的、普通的机器完成单个计算机无法完成的计算、存储任务。其目的是**利** **用更多的机器，处理更多的数据**。

分布式系统（distributed system）是建立在网络之上的软件系统。 首先需要明确的是，只有当单个节点的处理能力无法满足日益增长的计算、存储任务的时候，且硬件的 提升（加内存、加磁盘、使用更好的CPU）高昂到得不偿失的时候，应用程序也不能进一步优化的时候，我们才需要考虑分布式系统。因为，分布式系统要解决的问题本身就是和单机系统一样的，而由于 分布式系统多节点、通过网络通信的拓扑结构，会引入很多单机系统没有的问题，为了解决这些问题又会引入更多的机制、协议，带来更多的问题。



随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及 流动计算架构势在必行，急需**一个治理系统**确保架构有条不紊的演进。 

在Dubbo的官网文档有这样一张图:

![1605750943683](/Users/liqunzhang/Library/Application Support/typora-user-images/1605750943683.png)



**单一应用架构**(集中式架构)

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简 化增删改查工作量的数据访问框架(ORM)是关键。

**优点**：适用于小型网站，小型管理系统，将所有功能都部署到一个功能里，简单易用。 

**缺点：** 

1. 性能扩展比较难 

2. 协同开发问题 

3. 不利于升级维护

![1605751690538](/Users/liqunzhang/Library/Application Support/typora-user-images/1605751690538.png)

**垂直应用架构** 

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提 

升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键

**优点**：通过切分业务来实现各个模块独立部署，降低了维护和部署的难度，团队各司其职更易管理，性能扩展 

也更方便，更有针对性。

**缺点**： 公用模块无法重复利用，开发性的浪费 

![1605751699491](/Users/liqunzhang/Library/Application Support/typora-user-images/1605751699491.png)



**分布式服务架构**

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定 

的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的**分布式服** 

**务框架**(RPC)是关键。 

![1605751776894](/Users/liqunzhang/Library/Application Support/typora-user-images/1605751776894.png)



**流动计算架构** 

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问 

压力实时管理集群容量，提高集群利用率。此时，用于**提高机器利用率的资源调度和治理中心**(SOA)[ 

Service Oriented Architecture]是关键。

![1605751815565](/Users/liqunzhang/Library/Application Support/typora-user-images/1605751815565.png)

### 什么是RPC

RPC【Remote Procedure Call】是指远程过程调用，是一种进程间通信方式，他是一种技术的思想，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同。也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，**需要通过网络来表达调用的语义和传达调用的数据**。为什么要用RPC呢？就是无法在一个进程内，甚至一个计算机内通过本地调用的方式完成的需求，比如不同的系统间的通讯，甚至不同的组织间的通讯，由于计算能力需要横向扩展，需要在多台机器组成的**集群**上部署应用。**RPC就是要像调用本地的函数一样去调远程函数（方法）**； 

简单来说，就是一台机器远程调用另一台机器的函数或方法

# Dubbo

是阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的 RPC 实现服务的输出和输入功能，可以和 Spring框架无缝集成。

Dubbo是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

 Apache Dubbo is a high-performance, java based open source RPC framework 



# Jedis

1. 新建项目
2. 导入Jedis依赖

```xml
 <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.3.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.75</version>
        </dependency>

```

3. 测试联通代码

```java
//测试联通代码
public class TestPing {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        System.out.println(jedis.ping());
    }
}
```

4.Jedis整合事务

```java
public class TestMulti {
    public static void main(String[] args) {
        //创建客户端连接服务器
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        System.out.println(jedis.ping());
        jedis.flushAll();
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("username","Larry");
        jsonObject.put("password","zaxscd");

        //开启事务
        Transaction multi = jedis.multi();
        String result = jsonObject.toJSONString();
        try {
            multi.set("json",result);
            multi.set("json1",result);
            multi.exec();
        } catch (Exception e) {
            e.printStackTrace();
            multi.discard();   //如果出现异常，放弃事务
        } finally {
            System.out.println(jedis.get("json"));
            System.out.println(jedis.get("json1"));
            jedis.close();
        }
    }
}
```

正常运行结果如下：

![1606119462777](/Users/liqunzhang/Library/Application Support/typora-user-images/1606119462777.png)

如果在代码中事务执行前加一段异常：

![1606119554366](/Users/liqunzhang/Library/Application Support/typora-user-images/1606119554366.png)

运行结果如下：

![1606119443264](/Users/liqunzhang/Library/Application Support/typora-user-images/1606119443264.png)

发现写入对列的命令没有生效，事务进行了回滚



# SpringBoot缓存



## JSR107（了解）

JSR107整合的难度较大，后期整合更多都会使用缓存抽象或整合Redis的方式

### JSR107规范的核心概念

Java Caching定义了5个核心接口，分别是**CachingProvider**, **CacheManager**, **Cache**, **Entry**和 **Expiry**。 

- **CachingProvider（缓存提供者）**定义了创建、配置、获取、管理和控制多个**CacheManager**。一个应用可以在运行期访问多个CachingProvider。 

- **CacheManager（缓存管理）**定义了创建、配置、获取、管理和控制多个唯一命名的**Cache**，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。 

- **Cache**是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。 

- **Entry**是一个存储在Cache中的key-value对。 

- **Expiry（缓存有效期）** 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

![1606207623448](/Users/liqunzhang/Library/Application Support/typora-user-images/1606207623448.png)



```java
public interface CacheManager extends Closeable {
    CachingProvider getCachingProvider();
    URI getURI();
    ClassLoader getClassLoader();
    Properties getProperties();
    <K, V, C extends Configuration<K, V>> Cache<K, V> createCache(String var1, C var2) throws IllegalArgumentException;
    <K, V> Cache<K, V> getCache(String var1, Class<K> var2, Class<V> var3);
    <K, V> Cache<K, V> getCache(String var1);
    Iterable<String> getCacheNames();
    void destroyCache(String var1);
    void enableManagement(String var1, boolean var2);
    void enableStatistics(String var1, boolean var2);
    void close();
    boolean isClosed();
    <T> T unwrap(Class<T> var1);
}

```

CacheManager接口中提供了多种抽象方法：获取Cache，获取Cache的名字，销毁Cache等,获取了Cache后又可以对Cache进行各种增删改查操作，这些操作都在Cache接口中定义着

```java
public interface Cache<K, V> extends Iterable<Cache.Entry<K, V>>, Closeable {
    V get(K var1);
    Map<K, V> getAll(Set<? extends K> var1);
    boolean containsKey(K var1);
    void loadAll(Set<? extends K> var1, boolean var2, CompletionListener var3);
    void put(K var1, V var2);
    void putAll(Map<? extends K, ? extends V> var1);
    boolean putIfAbsent(K var1, V var2);
    boolean remove(K var1);
    boolean remove(K var1, V var2);
    V getAndRemove(K var1);
    boolean replace(K var1, V var2, V var3);
    boolean replace(K var1, V var2);
    V getAndReplace(K var1, V var2);
    void removeAll(Set<? extends K> var1);
    void removeAll();
    void clear();
    Iterator<Cache.Entry<K, V>> iterator();
    public interface Entry<K, V> {
        K getKey();
        V getValue();
        <T> T unwrap(Class<T> var1);
    }
}
```



## Spring的缓存抽象



### 介绍

Spring从3.1开始定义了Cache和CacheManager接口来统一不同的缓存技术；并支持使用JCache（JSR-107）注解简化我们开发； 

- Cache接口为缓存的组件规范定义，包含缓存的各种操作集合； 
- Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等； 

- 每次调用需要缓存功能的方法时，Spring会先检查指定参数的指定的目标方法是否已经被调用过，如果没有就调用方法并缓存结果后返回给用户，下次调用直接从缓存中获取。

   如果有被调用过，就直接走缓存，从缓存中获取方法调用后的结果

- 使用Spring缓存抽象时我们需要关注以下两点； 
  1. 确定方法需要被缓存以及他们的缓存策略
  2. 从缓存中读取之前缓存存储的数据



### 几个重要概念

![1606209682460](/Users/liqunzhang/Library/Application Support/typora-user-images/1606209682460.png)

### 一、搭建基本环境

![1606210608924](/Users/liqunzhang/Library/Application Support/typora-user-images/1606210608924.png)


1. 导入数据库文件 创建出department和employee表

2. 创建javaBean封装数据

3. 整合MyBatis操作数据库

   - 配置数据源信
   - 使用注解版的MyBatis；
   - @MapperScan指定需要扫描的mapper接口所在的包

4. 编写配置文件

   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/spring_cache?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
   spring.datasource.username=root
   spring.datasource.password=zaxscd
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   
   #开启驼峰命名规则
   mybatis.configuration.map-underscore-to-camel-case=true
   #开启log日志
   logging.level.com.zlq.mapper=debug
   ```

5. 编写mapper接口()使用注解的方式)

   

   ```java
   @Mapper
   public interface EmployeeMapper {
       @Select("select * from employee where id = #{id}")
       Employee getEmpById(Integer id);
   
       @Update("update employee set lastName = #{lastName},email = #{email},gender = #{gender},d_id = {did} where id = #{id} ")
       void updateEmp(Employee employee);
   
       @Delete("delete from employee where id = #{id}")
       void deleteEmpById(Integer id);
   
       @Insert("insert into employee(lastName,email,gender,d_id) values(#{lastName},#{email},#{gender},#{dId})")
       void insertEmployee(Employee employee);
   
       @Select("select * from employee where lastName = #{lastName}")
       Employee getEmpByLastName(String lastName);
   }
   ```

   ```java
   @Mapper
   public interface DepartmentMapper {
       @Select("SELECT * FROM department WHERE id = #{id}")
       Department getDeptById(Integer id);
   }
   ```

6. 测试环境

   ```java
      @Autowired
       EmployeeMapper employeeMapper;
       @Test
       void contextLoads() {
           System.out.println(employeeMapper.getEmpById(1));
       }
   }
   ```

   

   


7. 编写Service类

   ```java
   @Service
   @Cacheable
   public class EmployeeService {
       @Autowired
       EmployeeMapper employeeMapper;
   
       public Employee getEmpById(Integer id){
           System.out.println("查询" + id + "号员工");
           return employeeMapper.getEmpById(id);
       }
   
       public Employee updateEmp(Employee employee){
           System.out.println("更改：" + employee);
           employeeMapper.updateEmp(employee);
           return employee;
       }
   
       public void deleteEmpById(Integer id){
           System.out.println("删除的员工Id：" + id);
           employeeMapper.deleteEmpById(id);
       }
   
       public Employee getEmpByLastName(String lastName){
           return employeeMapper.getEmpByLastName(lastName);
       }
   }
   ```

   

### 二、快速体验缓存

缓存注解的介绍

- @EnableCaching  --标注在主启动类上，表示开启缓存功能
- @Cacheable   --根据方法的请求参数对其结果进行缓存
- @CacheEvict    --清空缓存
- @CachePut     --既保证方法被调用，又希望结果被缓存

**@Cachable的几个属性**

  几个属性：

![1606225731771](/Users/liqunzhang/Library/Application Support/typora-user-images/1606225731771.png)

- cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个指定的缓存中，可以以以数组的方式指定多个缓存；

- key：缓存数据使用的键key；可以为空，可以用它来指定。为空时：**默认是使用方法参数的值**

  ​	 编写SpEL； #id;参数id的值   #a0  #p0  #root.args[0]
  ​	 getEmp[2]

- keyGenerator：key的生成器；可以自己指定key的生成器的组件id
       注：key/keyGenerator：二选一使用;

- cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器

- condition：指定符合条件的情况下才缓存；
   condition = "#id>0"
   condition = "#a0>1"：第一个参数的值》1的时候才进行缓存

- unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
   unless = "#result == null"
  unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
- sync：是否使用异步模式

SPEL表达式列举

![1606225452292](/Users/liqunzhang/Library/Application Support/typora-user-images/1606225452292.png)



1、在主启动类上开启基于注解的缓存 @EnableCaching

在主启动类上加入@EnableCaching注解

作用：开启基于注解的缓存

2、在service类的方法上加入@Cachable注解

作用：将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
CacheManager管理多个Cache组件，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一的名字；

3、测试：第一次查询这届从mysql中取，控制台也显示有sql语句

![1606227600310](/Users/liqunzhang/Library/Application Support/typora-user-images/1606227600310.png)

第二次进行同样的查询控制台无反应，但是页面正常显示，说明走了缓存

### 三、缓存工作原理

在SpringBoot想要探究原理那就肯定要找到自动配置类来进行探究，我们探究的是Cache原理，那肯定要想到CacheAutoConfiguration自动配置类了

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({CacheManager.class})
@ConditionalOnBean({CacheAspectSupport.class})
@ConditionalOnMissingBean(
    value = {CacheManager.class},
    name = {"cacheResolver"}
)
@EnableConfigurationProperties({CacheProperties.class})
@AutoConfigureAfter({CouchbaseDataAutoConfiguration.class, HazelcastAutoConfiguration.class, HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class})
@Import({CacheAutoConfiguration.CacheConfigurationImportSelector.class, CacheAutoConfiguration.CacheManagerEntityManagerFactoryDependsOnPostProcessor.class}){
    
}
```

我们看到@EnableConfigurationProperties({CacheProperties.class}：使CacheProperties类中的@ConfigurationProperties注解生效

![1606230280581](/Users/liqunzhang/Library/Application Support/typora-user-images/1606230280581.png)

里面的prefix使我们在配置文件中可以以spring.cache为开头全局配置里面的属性

![1606230573796](/Users/liqunzhang/Library/Application Support/typora-user-images/1606230573796.png)

 既然是配置，肯定是定义了之后才能配置的，没定义，怎么可能配置，spring又不是神。那，定义了那些种类的缓存技术呢？我们看一下CacheType类 

 ![缓存枚举类 .png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy83MDQxNjc1LThmYmZlZTgxM2YwOGIwNzIucG5n?x-oss-process=image/format,png) 

 定义的，就这10种：GENERIC，JCACHE，EHCACHE，HAZELCAST，INFINISPAN，COUCHBASE，REDIS，CAFFEINE，SIMPLE，NONE。



在CacheAutpConfiguration类中有个方法selectImports

```java

       public String[] selectImports(AnnotationMetadata importingClassMetadata) {
            CacheType[] types = CacheType.values();
            String[] imports = new String[types.length];

            for(int i = 0; i < types.length; ++i) {
                imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
            }
            return imports;
        }
```

在该方法中打断点debug一下发现向容器中导入了这么多组件

![1606228959589](/Users/liqunzhang/Library/Application Support/typora-user-images/1606228959589.png)

这些组件都是各种cache的自动配置类

每个类的类头如下：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnBean({Cache.class})
@ConditionalOnMissingBean({CacheManager.class})
@Conditional({CacheCondition.class})
```

SimpleCacheConfiguration配置类默认生效,其他都是没生效的；



为什么SimpleCacheConfiguration配置类默认生效了？



SimpleCacheConfiguration的源码如下：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnMissingBean({CacheManager.class})
@Conditional({CacheCondition.class})
class SimpleCacheConfiguration {
    SimpleCacheConfiguration() {
    }

    @Bean
    ConcurrentMapCacheManager cacheManager(CacheProperties cacheProperties, CacheManagerCustomizers cacheManagerCustomizers) {
        ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
        List<String> cacheNames = cacheProperties.getCacheNames();
        if (!cacheNames.isEmpty()) {
            cacheManager.setCacheNames(cacheNames);
        }

        return (ConcurrentMapCacheManager)cacheManagerCustomizers.customize(cacheManager);
    }
}
```

里面导入了ConcurrentMapCacheManager这个组件

在ConcurrentMapCacheManager类中有getCache方法

![1606233003158](/Users/liqunzhang/Library/Application Support/typora-user-images/1606233003158.png)

![1606233202289](/Users/liqunzhang/Library/Application Support/typora-user-images/1606233202289.png)

我们可以看到，最终创建了ConcurrentMapCache类型的组件，点进这个组件

```java

public class ConcurrentMapCache extends AbstractValueAdaptingCache {
	private final String name;
	private final ConcurrentMap<Object, Object> store;

	private final SerializationDelegate serialization;
	public final boolean isStoreByValue() {
		return (this.serialization != null);
	}

	public final String getName() {	}
	public final ConcurrentMap<Object, Object> getNativeCache() {
		return this.store;
	}
	protected Object lookup(Object key) {	}
	public <T> T get(Object key, Callable<T> valueLoader) {}
	public void put(Object key, @Nullable Object value) {}
	public ValueWrapper putIfAbsent(Object key, @Nullable Object value) {}
	public void evict(Object key) {	}
	public boolean evictIfPresent(Object key) {}
	public void clear() {	}
	public boolean invalidate() {}
	protected Object toStoreValue(@Nullable Object userValue) {	}
	protected Object fromStoreValue(@Nullable Object storeValue) {}

```

发现里面有获取缓存名字，清空缓存等方法，而获取缓存是存放在ConcurrentMap<Object, Object>这个map集合中的，我们知道Map是以键值对的形式存在的，这里的键就是缓存中的key，缓存中的key又有着指定的生成策略！

运行流程：

1. 我们在Service类进行查询等等操作通过@Cachable添加缓存功能

![1606268216495](/Users/liqunzhang/Library/Application Support/typora-user-images/1606268216495.png)

2. 方法运行之前，先去查询Cache（缓存组件），按照缓存的名字指定获取；（CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建，通过this.cacheMap.put(name, cache);放在将缓存map集合中。

3. 通过调用lookUp方法去Cache中查找缓存的内容，使用一个key，默认就是方法的参数； key是按照某种策略生成的。

   默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key； SimpleKeyGenerator生成key的默认策略；如果没有参数；key=new SimpleKey()； 如果有一个参数：key=参数的值，如果有多个参数：key=new SimpleKey(params)；

   

   ![1606268683449](/Users/liqunzhang/Library/Application Support/typora-user-images/1606268683449.png)

![1606269459492](/Users/liqunzhang/Library/Application Support/typora-user-images/1606269459492.png)

![1606269667855](/Users/liqunzhang/Library/Application Support/typora-user-images/1606269667855.png)

![1606270391562](/Users/liqunzhang/Library/Application Support/typora-user-images/1606270391562.png)

4. 没有查到缓存就调用目标方法；

5. 将目标方法返回的结果，放进缓存中



 总的来说，@Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
 如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；



核心：

- 使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
- key使用keyGenerator生成的，默认是SimpleKeyGenerator



### @Cacheput

该注解意为：既保证方法被调用，又希望结果被缓存

我们要知道@Cachable和@Cachput不一样，@Cachable是在方法执行前调用缓存，查看key里面有没有指定数据，如果有走缓存，如果没有缓存那就先执行你的查询的方法，然后将方法查询的结果放在缓存中。

而@Cachput是：

1. 先调用目标方法，**目标方法一定是会被调用的**
2. 将目标方法的结果缓存进来



现在我们来更改一个员工数据

更改了1号员工

![1606273252648](/Users/liqunzhang/Library/Application Support/typora-user-images/1606273252648.png)

再来查询1号员工

![1606273285711](/Users/liqunzhang/Library/Application Support/typora-user-images/1606273285711.png)

查询的居然是更新前的，从控制台上也可以看到没有执行sql语句，走了之前的缓存



   详细测试步骤：

1.  查询1号员工；查到的结果会放在缓存中；key为1 （因为getEmpById()标注的 @Cacheable(value = {"employee"})，key为默认方法参数值） lastName:Larry

2. 查询结果放入缓存

3. 更新1号员工；【lastName:张立群；gender:0】，将方法的返回值也放进缓存了；key：传入的employee对象值：返回的employee对象

4. 查询1号员工，应该是更新后的员工；

   为什么是没更新前的？因为缓存中的1号员工没有更新。

   我们更改完了的员工信息是放在缓存了，但是他们之间的key不同

   我们看下自己写的两个service方法,因为key没指定，默认都是方法参数，第一个key为1，第二个key为employee了

   ```java
      @Cacheable(value = {"employee"})
       public Employee getEmpById(Integer id){
           System.out.println("查询" + id + "号员工");
           Employee employee = employeeMapper.getEmpById(id);
           return employee;
       }
   
       @CachePut(value = {"employee"})
       public Employee updateEmp(Employee employee){
           System.out.println("更改：" + employee);
           employeeMapper.updateEmp(employee);
           return employee;
       }
   ```

   

我们可以指定 key，方式有两种：

- key = "#employee.id":使用传入的参数的员工id； 
- key = "#result.id"：使用返回后的id @Cacheable的key是不能用#result 

即可将之前的缓存覆盖



重新测试，搞定



### @CacheEvict

@CacheEvict意为：缓存清除

参数说明：

- key：指定要清除的数据
- allEntries = true：指定清除这个缓存中所有的数据
- beforeInvocation = false：缓存的清除是否在方法之前执行，默认的缓存清除操作是在方法执行之后才执行，如果出现异常缓存就不会清除
- beforeInvocation = true：代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除



### @Caching

@Caching用来定义复杂的缓存规则

可以将@Cacheable、 @CachePut、@CacheEvict三个注解组合起来

```java
public @interface Caching {
    Cacheable[] cacheable() default {};
    CachePut[] put() default {};
    CacheEvict[] evict() default {};
}
```

比如我们重新写一个按名字查询的方法

```java
   @Caching(
         cacheable = {
             @Cacheable(value="employee",key = "#lastName")
         },
         put = {
             @CachePut(value="employee",value="employee",key = "#result.id"),
             @CachePut(value="employee",key = "#result.email")
         }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }
```

如此一来，我们通过名字查询员工信息就以id和email为键都保存了下来，以后我们通过id查还是通过email查都可以走缓存



### @CachConfig

如果我们觉得每次再缓存上定义缓存的名字太麻烦，可以使用@CacheConfig定义在类的上面，抽取缓存的公共配置：

![1606280456955](/Users/liqunzhang/Library/Application Support/typora-user-images/1606280456955.png)

```java
@CacheConfig(cacheNames="employee"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置
```



那么目标方法的缓存注解上就不用定义写缓存的名字了

![1606280578003](/Users/liqunzhang/Library/Application Support/typora-user-images/1606280578003.png)

## SpringBoot整合Redis

 * SpringBoot默认使用的是ConcurrentMapCacheManager==ConcurrentMapCache；将数据保存在	ConcurrentMap<Object, Object>中

 * 开发中使用的缓存中间件有：Redis、memcached、ehcache；

   我们这里使用Redis

   

整合redis作为缓存，Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。





### 基础使用

1. 新建SpringBoot项目
2. 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

3. 创建并编写application.yaml来配置Redis

```yaml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    jedis:
      pool:
        max-active: 8
        max-wait: -1ms
        max-idle: 500
        min-idle: 0
    lettuce:
      shutdown-timeout: 0
```



4. 测试代码

注意：使用redisTemplate操作对象时一定要把JavaBean类进行序列化（使JavaBean继承Serializable）

```java

    @Autowired
    RedisTemplate redisTemplate;

    @Autowired
    StringRedisTemplate stringRedisTemplate;

	
    @Test //测试操作String字符串类型数据
    void contextLoads() {
        redisTemplate.opsForValue().set("key1","value1");
        System.out.println(redisTemplate.opsForValue().get("key1"));
    }

    @Test //测试操作Object对象类型数据
    void contextLoads1(){
        Employee employee = employeeService.getEmpById(1);
        redisTemplate.opsForValue().set("employee",employee);
        Employee employeeValue = (Employee) redisTemplate.opsForValue().get("employee");
        System.out.println(employeeValue);
    }	

```



### 原理分析

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class}) //使RedisProperties类中用@ConfigurationProperties(prefix = "spring.redis")的注解生效
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    //向容器中注入RedisTemplate对象，有@ConditionalOnMissingBean注解意味着我们也可以自己手动配置RedisTemplate
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}

```

可见，在RedisAutoConfiguration类中给容器注入了两个template组件：RedisTemplate<Object, Object>和StringRedisTemplate

这两个template组件是为了操作redis的，我们想使用就直接从容其中取即可

StringRedisTemplate是用来专门操作K-V都是String字符串类型数据的

RedisTemplate<Object, Object>用来操作K-V都是对象类型的数据



RedisTemplate操作对象的序列化机制默认使用JDK序列化机制

![1606291921712](/Users/liqunzhang/Library/Application Support/typora-user-images/1606291921712.png)



### 自定义RedisTemplate

我们使用RedisTemplate虽然能操作Object对象类型数据，但是每次每次用来操作不同的Object对象类型不是很方便，需要强转等，我们可以自定义RedisTemplate，一来可以简化我们的操作，二来可以改变其序列化机制

```java
   @Bean
    public RedisTemplate<Object, Employee> empRedisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<Object, Employee> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        //设置使用Json序列化器
        template.setDefaultSerializer(new Jackson2JsonRedisSerializer<Employee>(Employee.class)); 
        return template;
    }
```

自定义好以后我们就可以从容其中取我们自己的RedisTemplate来进行操作。

```java
   
    @Autowired
    RedisTemplate<Object, Employee> employeeRedisTemplate;

    @Test
    void contextLoads2(){
        Employee employee = employeeService.getEmpById(2);
        employeeRedisTemplate.opsForValue().set("employee2",employee);
        System.out.println(employeeRedisTemplate.opsForValue().get("employee2"));
    }
```



### 自定义CacheManager

原理：CacheManager创建Cache 缓存组件来实际给缓存中存取数据
1）、我们引入redis的starter，容器中保存的就是 RedisCacheManager；
2）、RedisCacheManager 帮我们创建 RedisCache 来作为缓存组件；RedisCache通过RedisCacheManager操作redis缓存数据
3）、默认保存数据 k-v 都是Object；利用JDK序列化保存；如何保存为json
		1、引入了redis的starter，cacheManager变为 RedisCacheManager；
		2、默认创建的 RedisCacheManager 操作redis的时候使用的是 RedisTemplate<Object, Object>
        3、RedisTemplate<Object, Object> 是 默认使用JDK的序列化机制
4）、自定义CacheManager；

```java
 @Bean
    public RedisCacheManager deptCacheManager(RedisTemplate<Object, Department> deptRedisTemplate){
        RedisCacheManager cacheManager = new RedisCacheManager(deptRedisTemplate);
        //key多了一个前缀

        //使用前缀，默认会将CacheName作为key的前缀
        cacheManager.setUsePrefix(true);

        return cacheManager;
    }
```

注：如果存在多个CacheManager，则需要将某个缓存管理器作为默认的



# SpringBoot与消息

## 一、概述

1. 大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力

2. 消息服务中两个重要概念：

   消息代理（message broker）和目的地（destination）

   当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。

3. 消息队列主要有两种形式的目的地
   - 队列（queue）：点对点消息通信（point-to-point）
   - 主题（topic）：发布（publish）/订阅（subscribe）消息通信



4. 使用消息的作用
   - 异步通信
   - 应用解耦
   - 流量消峰

![image-20201130093454666](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130093454666.png)

![image-20201130093806499](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130093806499.png)

![image-20201130093955927](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130093955927.png)

4. 点对点式：

   - 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列

   - 消息只有唯一的发送者和接受者，但并不是说只能有一个接收者

5. 发布订阅式：

   发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息

6. JMS（Java Message Service）Java的一个API：JAVA消息服务：

   基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

7. AMQP（Advanced Message Queuing Protocol）：网络线程协议 

   - 高级消息队列协议，也是一个消息代理的规范，兼容JMS

   - RabbitMQ是AMQP的实现

**JMS和AMQP的对比**

![image-20201130094656824](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130094656824.png)

8. Spring支持

- spring-jms提供了对JMS的支持

- spring-rabbit提供了对AMQP的支持

- 需要ConnectionFactory的实现来连接消息代理

- 提供JmsTemplate、RabbitTemplate来发送消息

- @JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发布的消息

- @EnableJms、@EnableRabbit开启支持

  

9. SpringBoot的自动配置

- JmsAutoConfiguration

- RabbitAutoConfiguration



## 二、RabbitMQ

### RabbitMQ简介

RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。

### 核心概念：

- **Message**

  消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

- **Publisher**

  消息的生产者，也是一个向交换器发布消息的客户端应用程序。

- **Exchange**

  交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

  Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别

- **Queue**

  消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

- **Binding**

  绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

  Exchange 和Queue的绑定可以是多对多的关系。

- **Connection**

  网络连接，比如一个TCP连接。

- **Channel**

  信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

- **Consumer**

  消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

- **Virtual Host**

  虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

- **Broker**

  表示消息队列服务器实体

  ![image-20201130100616320](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130100616320.png)



### RabbitMQ的运行机制



AMQP 中的消息路由

- AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了

  **Exchange** 和 **Binding** 的角色。也是运行机制的核心，生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。不同的交换器和绑定规则决定消息能达到的不同的队列

![image-20201130100813136](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130100813136.png)

Exchange 类型

- **Exchange**分发消息时根据类型的不同分发策略有区别，目前共四种类型：

  **direct、fanout、topic、headers** 

  headers 匹配 AMQP 消息的 header消息头而不是路由键， headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：



​	Direct Exchange：只有当消息的路由键和绑定的key一致才能将消息发送到对应的队列（点对点） 

![image-20201130100946825](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130100946825.png)

​	Fanout Exchange ：不管绑定的key是什么都会把消息发送过去（广播模式），速度最快

![image-20201130101011919](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130101011919.png)

​	Topic Exchange：通过单词对消息进行匹配

![image-20201130101027542](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130101027542.png)

### RabbitMQ的安装配置及运行测试

下载并安装docker

在docker设置阿里云镜像，否则下载很慢

```properties
{
  "debug": true,
  "experimental": false,
  "registry-mirrors": [
    "https://gvyg7vv4.mirror.aliyuncs.com"
  ]
}
```



![image-20201130105802033](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130105802033.png)

- 下载docker

```bash
docker search rabbitmq:management
 
docker pull rabbitmq:management

#运行rabbitmq 
docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq 67a3629d4d71

```

![image-20201130111123592](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130111123592.png)

- 登录rabbitmq

  输入localhost:15672即进入rabbitmq操作界面

  账号密码默认都为guest

   ![image-20201130203303841](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130203303841.png)

- 创建Exchange交换器

  ![image-20201130203603361](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130203603361.png)

按照此步骤添加另外两个交换器

![image-20201130203755000](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130203755000.png)

- 创建队列

![image-20201130204202449](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130204202449.png)

这些队列要能工作必须和交换器绑定

- 绑定交换器

![image-20201130204441178](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130204441178.png)

依照此方法，我们给上面创建的三个信道都绑定上面创建的四个队列

![image-20201130205443958](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130205443958.png)

![image-20201130205509235](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130205509235.png)

![image-20201130205534626](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130205534626.png)

绑定结果如上



- 发布消息

  1. 使用Exchange.direct交换器发布消息，此消息发布模式为点对点发布，只能发布给对应的Routing Key路由键

  ![image-20201130210013008](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130210013008.png)

  ​	发现China已经有了一个消息

  ![image-20201130210128676](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130210128676.png)

  ​	点进China，获取到了发布的消息

  ​	![image-20201130210326390](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130210326390.png)

  ​	

  2. 使用Exchange.fanout交换器发布消息，不管路由键是什么，都会把消息全部发送出去

     ![image-20201130210828783](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130210828783.png)

      ![image-20201130211039955](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130211039955.png)

     发现所有的队列都收到了消息

     

     

  3. 使用Exchange.topic交换器发布消息，它是根据路由键的规则进行匹配的

     ![image-20201130212237740](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130212237740.png)

  ​     因此，以下四个可以接收到

  ![image-20201130212346479](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130212346479.png)

## 三、整合SpringBoot

1. 创建项目，引入web框架

2. 引入依赖

   ```xml
   				<dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-messaging</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.amqp</groupId>
               <artifactId>spring-rabbit</artifactId>
           </dependency>
   ```

   

3. 自动配置原理

   在RabbitAutoConfiguration类中有自动配置了连接工厂ConnectionFactory；

   ```java
           @Bean
           @ConditionalOnMissingBean
           public RabbitTemplateConfigurer rabbitTemplateConfigurer(RabbitProperties properties, ObjectProvider<MessageConverter> messageConverter, ObjectProvider<RabbitRetryTemplateCustomizer> retryTemplateCustomizers) {
               RabbitTemplateConfigurer configurer = new RabbitTemplateConfigurer();
               configurer.setMessageConverter((MessageConverter)messageConverter.getIfUnique());
               configurer.setRetryTemplateCustomizers((List)retryTemplateCustomizers.orderedStream().collect(Collectors.toList()));
               configurer.setRabbitProperties(properties);
               return configurer;
           }
   ```

   ​	

   RabbitProperties 封装了 RabbitMQ的配置

   ```java
   public class RabbitProperties {
       private static final int DEFAULT_PORT = 5672;
       private static final int DEFAULT_PORT_SECURE = 5671;
       private String host = "localhost";
       private Integer port;
       private String username = "guest";
       private String password = "guest";
       private final RabbitProperties.Ssl ssl = new RabbitProperties.Ssl();
       private String virtualHost;
       private String addresses;
       private AddressShuffleMode addressShuffleMode;
   .......
   ```

   

    RabbitTemplate ：给RabbitMQ发送和接受消息；

   ```java
     @Bean
           @ConditionalOnSingleCandidate(ConnectionFactory.class)
           @ConditionalOnMissingBean({RabbitOperations.class})
           public RabbitTemplate rabbitTemplate(RabbitTemplateConfigurer configurer, ConnectionFactory connectionFactory) {
               RabbitTemplate template = new RabbitTemplate();
               configurer.configure(template, connectionFactory);
               return template;
           }
   ```

   ```java
   public class RabbitTemplate extends RabbitAccessor implements BeanFactoryAware, RabbitOperations, MessageListener, ListenerContainerAware, Listener, BeanNameAware, DisposableBean {
       private static final String UNCHECKED = "unchecked";
       private static final String RETURN_CORRELATION_KEY = "spring_request_return_correlation";
       private static final String DEFAULT_EXCHANGE = "";
       private static final String DEFAULT_ROUTING_KEY = "";
       private static final long DEFAULT_REPLY_TIMEOUT = 5000L;
       private static final long DEFAULT_CONSUME_TIMEOUT = 10000L;
       private static final String DEFAULT_ENCODING = "UTF-8";
       private static final SpelExpressionParser PARSER = new SpelExpressionParser();
       private final ThreadLocal<Channel> dedicatedChannels = new ThreadLocal();
       private final AtomicInteger activeTemplateCallbacks = new AtomicInteger();
   ......
   ```

   

     AmqpAdmin ： RabbitMQ系统管理功能的组件;

   ```java
      @ConditionalOnMissingBean
           public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {
               return new RabbitAdmin(connectionFactory);
           }
       }
   ```

   ```java
   public interface AmqpAdmin {
       void declareExchange(Exchange var1);
       boolean deleteExchange(String var1);
       @Nullable
       Queue declareQueue();
       @Nullable
       String declareQueue(Queue var1);
       boolean deleteQueue(String var1);
       void deleteQueue(String var1, boolean var2, boolean var3);
       void purgeQueue(String var1, boolean var2);
       int purgeQueue(String var1);
       void declareBinding(Binding var1);
       void removeBinding(Binding var1);
       @Nullable
       Properties getQueueProperties(String var1);
       @Nullable
       QueueInformation getQueueInfo(String var1);
       default void initialize() {
       }
   }
   ```

   

   AmqpAdmin：创建和删除 Queue，Exchange，Binding

   @EnableRabbit +  @RabbitListener 监听消息队列的内容

4. 测试

   使用RabbitTemplate给消息队列发送消息

   ```java
    @Autowired
       RabbitTemplate rabbitTemplate;
   
       @Test
       void contextLoads() {
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("msg","中华民资繁荣昌盛!");
           map.put("msg2","中华民族伟大复兴！");
           //对象被默认序列化发送出！
           rabbitTemplate.convertAndSend("exchange.direct","China",map);
       }
       @Test
       public void receive(){
           Object news = rabbitTemplate.receiveAndConvert("China");
           System.out.println(news.getClass());
           System.out.println(news);
       }
   ```

   ![image-20201130222050106](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130222050106.png)

   进入控制面板也能收到消息

   ![image-20201130222209945](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130222209945.png)



我们看到消息的序列化机制是使用java的序列化，那么如何改为json的序列化呢

![image-20201130223018976](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130223018976.png)

默认是使用SimpleMessageConverter

可以自己创建个配置类进行修改

```java
@Configuration
public class MyConfiguration {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

![image-20201130224346931](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130224346931.png)

 **消息的监听**

@EnableRabbit +  @RabbitListener 监听消息队列的内容

在只启动类上添加开启rabbiemq的注解@EnableRabbit 

编写service类

```java
@EnableRabbit @Service
public class AddressService {

    @RabbitListener(queues = "America.news" )  //监听America.news这个消息
    public void receive(Address address){

        System.out.println("收到的消息为：" + address);
    }
}
```

运行SpringBoot程序发现之前添加的所有消息都打印到控制台中



**如何在SpringBoot程序中创建删除队列和添加各种绑定规则？**



AmqpAdmin 是RabbitMQ系统管理功能的组件;

```java
 @Autowired
    AmqpAdmin amqpAdmin;

    @Test
    public void test(){
        amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange")); //创建交换器
        amqpAdmin.declareQueue(new Queue("Japan.news",true));   //创建一个消息
        amqpAdmin.declareBinding(new Binding("Japan.news", Binding.DestinationType.QUEUE,"amqpadmin.exchange","Japan.news",null));
    }
```





![image-20201130233412156](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130233412156.png)

![image-20201130233440241](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201130233440241.png)



# SprngBoot Webfulx



## Webflux的介绍

Webflux是 Spring5 添加新的模块，用于 web 开发的，功能和 SpringMVC 类似的，Webflux 使用当前一种比较流行的响应式编程而出现的框架。

传统的web框架（比如 SpringMVC）是基于 Servlet 容器的，Webflux 是一种异步非阻塞的框架（异步非阻塞的框架在 Servlet3.1 以后才出现），Spring Boot Webflux 就是基于 Reactor 实现的。一般来说，Spring MVC 用于同步处理，Spring Webflux 用于异步处理。

Spring Boot Webflux 有两种编程模型实现，一种类似 Spring MVC 注解方式，另一种是使用其功能性端点方式



## 什么是同步，异步；什么是阻塞，非阻塞



### 同步和异步

- 同步：A调用B，此时只有等B有结果了才返回。

- 异步: A调用B，B立即返回，无须等待。当B处理完后会通过通知或者回调函数的方式来告诉A结果。

  

  **异步和同步针对调用者**，调用者发送请求，如果等着对方回应之后才去做其他事情就是同步，如果发送请求之后不等着对方回应就去做其他事情就是异步

### 阻塞和非阻塞

- 阻塞：A调用B，A会被被挂起，一直在等待B的结果，什么事都不能干。

- 非阻塞：A调用B，自己用被挂起等待B的结果，可以去干其他的事情。

  

  **阻塞和非阻塞针对被调用者**，被调用者收到请求之后，做完请求任务之后才给出反馈就是阻塞，收到请求之后马上给出反馈然后再去做别的事情就是非阻塞



## Webflux 特点：

- 非阻塞式：

  在有限资源下，提高系统吞吐量和伸缩性，以 Reactor 为基础实现响应式编程

- 函数式编程

  Spring5 框架基于 java8，Webflux 使用 Java8 函数式编程方式实现路由请求

- 响应式API

  Reactor 框架是 Spring Boot Webflux 响应库依赖，通过 Reactive Streams 并与其他响应库交互。提供了两种响应式 API : Mono 和 Flux。一般是将 Publisher 作为输入，在框架内部转换成 Reactor 类型并处理逻辑，然后返回 Flux 或 Mono 作为输出。

- 适用性

  ![img](https:////upload-images.jianshu.io/upload_images/1483536-36c8e6a69f99646f.png?imageMogr2/auto-orient/strip|imageView2/2/w/800)

  

  WebFlux 和 MVC 有交集，方便大家迁移。但是注意：

  1. MVC 能满足场景的，就不需要更改为 WebFlux。
  2. 要注意容器的支持，可以看看下面内嵌容器的支持。
  3. 微服务体系结构，WebFlux 和 MVC 可以混合使用。尤其开发 IO 密集型服务的时候，选择 WebFlux 去实现。

  

  ## WebFlux和SpringMVC的区别

  SpringMVC 方式实现，同步阻塞的方式，基于 SpringMVC+Servlet+Tomcat

  SpringWebflux 方式实现，异步非阻塞 方式，基于 SpringWebflux+Reactor+Netty

  

- 编程模型

  Spring 5 web 模块包含了 Spring WebFlux 的 HTTP 抽象。类似 Servlet API , WebFlux 提供了WebHandler API 去定义非阻塞 API 抽象接口。可以选择以下两种编程模型实现：

  - 注解控制层。和 MVC 保持一致，WebFlux 也支持响应性 @RequestBody 注解。

  - 功能性端点。基于 lambda 轻量级编程模型，用来路由和处理请求的小工具。和上面最大的区别就是，这种模型，全程控制了请求 - 响应的生命流程

    

- 内嵌容器

  跟 Spring Boot 大框架一样启动应用，但 WebFlux **默认是通过 Netty 启动**，并且自动设置了默认端口为 8080。另外还提供了对 Jetty、Undertow 等容器的支持。开发者自行在添加对应的容器 Starter 组件依赖，即可配置并使用对应内嵌容器实例。

  但是要注意，必须是 Servlet 3.1+ 容器，如 Tomcat、Jetty；或者非 Servlet 容器，如 Netty 和 Undertow。

- Starter 组件

  跟 Spring Boot 大框架一样，Spring Boot Webflux 提供了很多 “开箱即用” 的 Starter 组件。Starter 组件是可被加载在应用中的 Maven 依赖项。只需要在 Maven 配置中添加对应的依赖配置，即可使用对应的 Starter 组件。例如，添加 `spring-boot-starter-webflux` 依赖，就可用于构建响应式 API 服务，其包含了 Web Flux 和 Tomcat 内嵌容器等。

  

## 响应式编程

响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。电子表格程序就是响应式编程的一个例子。单元格可以包含字面值或类似"=B1+C1"的公式，而包含公式的单元格的值会依据其他单元格的值的变化而变化。



## Reactor实现

1. 响应式编程操作中，Reactor满足Reactive规范框架

2. Reactor有两个核心类，Mono和Flux，这两个类实现接口Publisher，提供丰富操作符。

   - Mono：实现发布者，并返回 0 或 1 个元素，即单对象

   - Flux：实现发布者，并返回 N 个元素，即 List 列表对象

3. Flux 和 Mono 都是数据流的发布者，使用 Flux 和 Mono 都可以发出三种数据信号：

   元素值，错误信号，完成信号，错误信号和完成信号都代表终止信号，终止信号用于告诉订阅者

   据流结束了，错误信号终止数据流同时把错误信息传递给订阅者



## Mono和Flux



**Mono**

Mono 是什么？ 官方描述如下：A Reactive Streams Publisher with basic rx operators that completes successfully by emitting an element, or with an error.

Mono 是响应流 Publisher 具有基础 rx 操作符。可以成功发布元素或者错误。如图所示：

![image-20201203131405629](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201203131405629.png)

Mono 常用的方法有：

- Mono.create()：使用 MonoSink 来创建 Mono

- Mono.justOrEmpty()：从一个 Optional 对象或 null 对象中创建 Mono。

- Mono.error()：创建一个只包含错误消息的 Mono

- Mono.never()：创建一个不包含任何消息通知的 Mono

- Mono.delay()：在指定的延迟时间之后，创建一个 Mono，产生数字 0 作为唯一值

  

  

**Flux**

Flux 是什么？ 官方描述如下：A Reactive Streams Publisher with rx operators that emits 0 to N elements, and then completes (successfully or with an error).

Flux 是响应流 Publisher 具有基础 rx 操作符。可以成功发布 0 到 N 个元素或者错误。Flux 其实是 Mono 的一个补充。如图所示：

![image-20201203131426708](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201203131426708.png)

所以要注意：如果知道 Publisher 是 0 或 1 个，则用 Mono。

Flux 最值得一提的是 fromIterable 方法。 fromIterable(Iterable<? extends T> it) 可以发布 Iterable 类型的元素。当然，Flux 也包含了基础的操作：map、merge、concat、flatMap、take，这里就不展开介绍了。



## 实验：基于注解式crud模型

创建SpringBoot工程引入Reactive Web模块

![image-20201203132940400](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201203132940400.png)

目录结构

![image-20201203132659624](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201203132659624.png)

创建实体类user

```java
public class User {
    private String name;
    private String gender;
    private Integer age;
  .....
}
```

dao层

```java
@Repository
public class UserDaoImpl implements UserDao {

    private final Map<Integer,User> map = new HashMap<>();

    public UserDaoImpl() {
        map.put(1,new User("Larry","boy",18));
        map.put(2,new User("Betty","girl",12));
        map.put(3,new User("Suyi","girl",15));
    }

    @Override
    public User getUserById(int id) {
        return map.get(id);

    }

    @Override
    public Collection<User> getAllUser() {
        return map.values();
    }

    @Override
    public Mono<Void> saveUser(Mono<User> userMono) {
        return userMono.doOnNext(person->{
            Integer id = map.size() + 1;
            map.put(id,person);
        }).thenEmpty(Mono.empty());
    }
}
```

service层

```java
@Component
public class UserServiceImpl implements UserService {
    @Autowired
    UserDao userDao;

    @Override
    public Mono<User> getUserById(int id) {
        return Mono.justOrEmpty(userDao.getUserById(id));
    }

    @Override
    public Flux<User> getAllUser() {
        return Flux.fromIterable(userDao.getAllUser());
    }

    @Override
    public Mono<Void> saveUser(Mono<User> userMono) {
        return userDao.saveUser(userMono);
    }
}

```

controller层

```java
@RestController
public class UserController {

    @Autowired
    UserService userService;

    @GetMapping("/getUserById/{id}")
    public Mono<User> getUserById(@PathVariable("id") Integer id) {
        return userService.getUserById(id);
    }

    @GetMapping("/getAllUsers")
    public Flux<User> getAllUser(){
        return userService.getAllUser();
    }

    @RequestMapping("/saveUser")
    public Mono<Void> saveUser(){
        User user = new User("张立群", "boy", 19);
        Mono<User> userMono = Mono.just(user);
        return userService.saveUser(userMono);
    }
}

```




# SpringBoot整合Dubbo+Zookeeper

  



# Spring Security



# Shiro



