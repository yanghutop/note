# Spring Boot中的事务管理

**事务管理操作**

在SpringBoot中，当我们使用了spring-boot-starter-jdbc或spring-boot-starter-data-jpa依赖的时候，框架会自动默认分别注入DataSourceTransactionManager或JpaTransactionManager。所以我们不需要任何额外配置就可       以用@Transactional注解进行事务的使用

默认创建表类型是MyISAM,是非事务安全的，所以无法实现事物回滚; Innodb才可以进行对事物的回滚

**隔离级别**

隔离级别是指在发生并发的事务之间的隔离程度，与我们开发时候主要相关的场景包括：脏读、不可重复读、幻  读。

**脏读**：A事务执行过程中修改了id=1的数据，未提交前，B事务读取了A修改的id=1的数据，而A事务却回滚了，这样B事务就形成了脏读。

**不可重复读**：A事务先读取了一条数据，然后执行逻辑的时候，B事务将这条数据改变了，然后A事务再次读取的时 候，发现数据不匹配了，就是所谓的不可重复读了。

**幻读**：A事务先根据条件查询到了N条数据，然后B事务新增了M条符合A事务查询条件的数据，导致A事务再次查询发现有N+M条数据了，就产生了幻读。

我们可以看**org.springframework.transaction.annotation.Isolation**  枚举类中定义了五个表示隔离级别的值：

```java
public enum Isolation {
    DEFAULT(-1),
    READ_UNCOMMITTED(1),
    READ_COMMITTED(2),
    REPEATABLE_READ(4),
	SERIALIZABLE(8);
}
```

**DEFAULT**这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就**READ_COMMITTED**是

**READ_UNCOMMITTED:**该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别

**READ_COMMITTED:**该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值，性能最好

**REPEATABLE_READ：**该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读

**SERIALIZABLE：**所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别

```java
//设置语法
@Transactional(isolation = Isolation.DEFAULT)
```



**传播行为**

传播行为是指，如果在开始当前事务之前，已经存在一个事务，此时可以指定这个要开始的这个事务的执行行为

我们可以看**org.springframework.transaction.annotation.Propagation**枚举类中定义了6个表示传播行为的枚举值

```java
public enum Propagation {
    REQUIRED(0), 	//（默认）如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
    SUPPORTS(1), 	//如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
    MANDATORY(2),	//如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
    REQUIRES_NEW(3),//创建一个新的事务，如果当前存在事务，则把当前事务挂起。
    NOT_SUPPORTED(4), //以非事务方式运行，如果当前存在事务，则把当前事务挂起。
    NEVER(5),		//以非事务方式运行，如果当前存在事务，则抛出异常。
	NESTED(6);		//如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务则                       该取值等价于REQUIRED
}
```

设置语法：

```java
@Transactional(propagation = Propagation.REQUIRED)
```

操作步骤：

1、启动类上加 @EnableTransactionManagement//开启事务自动配置

2、在service层加事务控制  在需要事务处理的类上加 @Transactional

# Spring Boot异步任务与定时任务

在项目开发中，绝大多数情况下都是通过同步方式处理业务逻辑的，但是比如批量处理数据，批量发送邮件，批量发送短信等操作 容易造成阻塞的情况，之前大部分都是使用多线程来完成此类任务而在Spring 3+之后，就已经内置了**@Async**注解来完美解决这个问题，从而提高效率。

使用的注解:

- @EnableAysnc 启动类上开启基于注解的异步任务
- @Aysnc 标识的方法会异步执行





# SpringBoot定时任务调度

在项目开发中，经常需要执行一些定时任务，比如 每月1号凌晨需要汇总上个月的数据分析报表; 每天凌晨分析前一天的日志信息等定时操作。Spring 为我们提供了异步执行定时任务调度的方式

使用的注解：

- @EnableScheduling启动类上开启基于注解的定时任务
- 编写任务类注入容器
- 在任务方法上加 @Scheduled标识的方法会进行定时处理
- 需要通过 cron 属性来指定 cron 表达式：秒 分 时 日 月 星 期 几

| **位置** | **取值范围**                               | **可指定的特殊字符** |
| -------- | ------------------------------------------ | -------------------- |
| 秒       | 0-59                                       | , - * /              |
| 分       | 0-59                                       | , - * /              |
| 小时     | 0-23                                       | , - * /              |
| 日期     | 1-31                                       | , - * ? / L W C      |
| 月份     | 1-12                                       | , - * /              |
| 星期     | 0-7或SUN-SAT 0和7都是周日，1-6是周一到周六 | , - * ? / L C #      |

| **特殊字符** | **代表含义**                                                 |
| ------------ | ------------------------------------------------------------ |
| ,            | 枚举，一个位置上指定多个值，以逗号 ， 分隔                   |
| -            | 区间                                                         |
| *            | 任意                                                         |
| /            | 步长，每隔多久执行一次                                       |
| ?            | 日/星期冲突匹配 ,指定哪个值,另外个就是?，比如： * * * ? * 1 每周1执行，则日用 ？ 不能用* ，不是每一天都是周一； * * * * 2 * ? 每月2号,则星期不能用* |
| L            | 最后                                                         |
| W            | 工作日                                                       |
| C            | 和calendar联系后计算过的值                                   |
| #            | 这个月的第几个星期几，4#2，第2个星期四                       |

在线生成cron表达式 http://cron.qqe2.com/



# Spring Boot 整合缓存

**缓存简介**

- 缓存是每一个系统都应该考虑的功能，它用于加速系统的访问，以及提速系统的性能
- 经常访问的高频热点数据
- 电商网站的商品信息：每次查询数据库耗时，可以引入缓存
- 微博阅读量、点赞数、热点话题等
- 临时性的数据：发送手机验证码，1分钟有效，过期则删除，存数据库负担有点大，这些临时性的数据也  可以放到缓存中, 直接从缓存中存取数据

Spring从3.1后定义了 org.springframework.cache.CacheManager 和 org.springframework.cache.Cache接口来统一不同的缓存技术

**CacheManager 缓存管理器**，用于管理各种Cache缓存组件

**Cache定义了缓存的各种操作**， Spring在Cache接口下提供了各种xxxCache的实现； 比如EhCacheCache，RedisCache， ConcurrentMapCache……

**Spring 提供了缓存注解：** @EnableCaching、@Cacheable、@CachePut、@CacheEvict



使用步骤:

1. 引入缓存启动器：spring-boot-starter-cache

2. @EnableCaching：在启动类上，开启基于注解的缓存

3. @Cacheable : 标在方法上，返回的结果会进行缓存

   属性:	value/cacheNames缓存的名字

   ​			 key : 作为缓存中的Key值，可自已使用 SpEL表达式指定(不指定就是参数值)， 缓存结果是方法返回值

| **名字**      | **描述**                                                     | **示例**             |
| ------------- | ------------------------------------------------------------ | -------------------- |
| methodName    | 当前被调用的方法名                                           | #root.methodName     |
| target        | 当前被调用的目标对象                                         | #root.target         |
| targetClass   | 当前被调用的目标对象类                                       | #root.targetClass    |
| args          | 当前被调用的方法的参数列表                                   | #root.args[0]        |
| caches        | 当前方法调用使用的缓存列表（如@Cacheable(value={"cache1", "cache2"})），则有两个cache | #root.caches[0].name |
| argument name | 方法参数的名字. 可以直接 #参数名 ，也可以使用 #p0或#a0 的形式，0代表参数的索引； | #iban 、 #a0 、 #p0  |
| result        | 方法执行后的返回值（仅当方法执行之后的判断有效,在@CachePut 使用于更新数据后可用） | #result              |

 

**@CachePut ：**保证方法被调用后，又将对应缓存中的数据更新（先调用方法，调完方法再将结果放到缓存） 比如：修改了表中某条数据后，同时更新缓存中的数据，使得别人查询这条更新的数据时直接从缓存中获取

测试更新User数据效果：

1. 先查询id=1的用户，放在缓存中；

2. 后面查询id=1的用户直接从缓存中查询;

3. 更新id=1的用户，同时会更新缓存数据;

4. 再查询id=1的用户应该是更新后的数据，是从缓存中查询，因为在更新时同时再新了缓存数据

   注意：需要指定key属性key="#user.id" 参数对象的id key = "#result.id" 返回值对象id



**@CacheEvict ：**清除缓存

​	属性：

​			key：指要清除的数据，如 key="#id"

​			allEntries =true : 指定清除这个缓存中所有数据

​			beforeInvocation = true : true在方法之前执行；默认false在方法之后执行,出现异常则不会清除缓存



**@CacheConﬁg** 指定缓存公共属性值

​	@CacheConﬁg(cacheNames = “user”) 指定在类上，其他方法上就不需要写缓存名





# SpringBoot整合邮件收发

1、坐标

```xml
 		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>

```

2、配置

```yml
spring：
	mail:
        username: yanghu180@163.com
        password: LXATNKFFEDDMJLTT
        host: smtp.163.com
        properties:
              smtp:
                ssl:
                  enable: true
                  
                  

spring.mail.username=1655020321@qq.com
spring.mail.password=
spring.mail.host=smtp.qq.com
spring.mail.properties.smtp.ssl.enable=true
```

3、得到JavaMailSenderImpl

```java
	@Autowired
    JavaMailSenderImpl sender;
```

4、

```java
//简单邮件
	@Autowired
    JavaMailSenderImpl sender;
@Test
    public void simplemail(){
        SimpleMailMessage message = new SimpleMailMessage();
        message.setSubject("开学通知");
        message.setText("2022-2-26开学");
        message.setFrom("yanghu180@163.com");//
        message.setTo("1442948578@qq.com");//发给的人
        sender.send(message);
    }
//带附件
@Test
    public void minmail() throws MessagingException {
        MimeMessage message = sender.createMimeMessage();
        
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setSubject("分享");
        helper.setText("好东西");
        helper.addAttachment("1.jpg",new File("C:\\Users\\86198\\Desktop\\1.jpg"));

        helper.setFrom("yanghu180@163.com");
        helper.setTo("162229486@qq.com");
        sender.send(message);
    }
```

