

# SpringBoot **数据访问操作**

## 一.SpringBoot整合JDBC

1.JDBC坐标

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
```

2.配置实体类映射

```java
public class MyRowMapper implements RowMapper<Country> {

    @Override
    public Country mapRow(ResultSet rs, int rowNum) throws SQLException {
        // 从结果集中拿数据，并封装到对象中
        int cid=rs.getInt("cid");
        String cname=rs.getString("cname");
        String ccapital=rs.getString("ccapital");

        Country country=new Country();
        country.setCid(cid);
        country.setCname(cname);
        country.setCcapital(ccapital);

        return country;

    }
}
```

3.使用JdbcTemplate 调用

```java
	@Autowired
    JdbcTemplate jdbcTemplate;
```



## 二.SpringBoot整合Mybatis与Mybatis-Plus

MyBatis

1，MyBatis坐标

```xml
	<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>2.1.1</version>
		</dependency>
```

2，MyBatis配置

```yaml
#配置mybatis相关文件路径
mybatis:
  type-aliases-package: com.apesource.springboot_data_mybatis_01.pojo
  configuration:
    map-underscore-to-camel-case: true
```

3.@Mapper 或 @MapperScan("com.apesource.springboot_data_mybatis_02.dao") 扫描

@Mapper逐个注册mapper映射器，每一个dao上都编写

@MapperScan("dao包路径")批量注册mapper映射器,别写在引导类上



MyBatis-plus

1.MyBatis-plus坐标

```xml
	<!--mybatis_plus启动器-->
		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-boot-starter</artifactId>
			<version>3.3.1</version>
		</dependency>
```

2.MyBatis-plus配置不用配



3.@Mapper 或 @MapperScan("com.apesource.springboot_data_mybatis_02.dao") 扫描

@Mapper逐个注册mapper映射器，每一个dao上都编写

@MapperScan("dao包路径")批量注册mapper映射器,别写在引导类上



## 三.SpringBoot切换druid数据源

1.加数据源坐标

```
<!--连接池-->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.1.12</version>
		</dependency>
```

2.配置yml文件中的数据源

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
    url: jdbc:mysql://127.0.0.1:3306/2109db?serverTimezone=GMT
    type: com.alibaba.druid.pool.DruidDataSource
    #初始化时建立物理连接的个数
    initialSize: 8
    #最小连接池数量
    minIdle: 5
    #最大连接池数量
    maxActive: 20
    #获取连接时最大等待时间，单位毫秒
    maxWait: 60000
    #Destroy线程会检测连接的间隔时间
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    #用来检测连接是否有效的sql，要求是一个查询语句
    validationQuery: SELECT 1 FROM DUAL
    #实现个别检测操作
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    #是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭
    poolPreparedStatements: true
#配置mybatis相关文件路径
mybatis:
  configuration:
    map-underscore-to-camel-case: true
```

3.用@Bean 和 @ConfigurationProperties(prefix = "spring.datasource") 注入

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return new DruidDataSource();
    }
}
```



### **3.1DRUID配置参数**

| 配置                          | 缺省值             | 说明                                                         |
| ----------------------------- | ------------------ | ------------------------------------------------------------ |
| name                          |                    | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。  如果没有配置，将会生成一个名字，格式是："DataSource-" + System.identityHashCode(this) |
| jdbcUrl                       |                    | 连接数据库的url，不同数据库不一样。例如：  mysql : jdbc:mysql://10.20.153.104:3306/druid2  oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
| username                      |                    | 连接数据库的用户名                                           |
| password                      |                    | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。详细看这里：https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter |
| driverClassName               | 根据url自动识别    | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置下) |
| initialSize                   | 0                  | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                     | 8                  | 最大连接池数量                                               |
| maxIdle                       | 8                  | 已经不再使用，配置了也没效果                                 |
| minIdle                       |                    | 最小连接池数量                                               |
| maxWait                       |                    | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements        | false              | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxOpenPreparedStatements     | -1                 | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| validationQuery               |                    | 用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。 |
| testOnBorrow                  | true               | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                  | false              | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
| testWhileIdle                 | false              | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| timeBetweenEvictionRunsMillis |                    | 有两个含义：  1) Destroy线程会检测连接的间隔时间2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| numTestsPerEvictionRun        |                    | 不再使用，一个DruidDataSource只支持一个EvictionRun           |
| minEvictableIdleTimeMillis    |                    |                                                              |
| connectionInitSqls            |                    | 物理连接初始化的时候执行的sql                                |
| exceptionSorter               | 根据dbType自动识别 | 当数据库抛出一些不可恢复的异常时，抛弃连接                   |
| filters                       |                    | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：  监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall |
| proxyFilters                  |                    | 类型是List<com.alibaba.druid.filter.Filter>，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |

### **3.2 yaml文件密码加密**

​	实现方式：

​	a.druid自带可以对密码加密（有且只能对密码）

​	b.Jasypt任意内容加密

### 3.3Jasypt任意内容加密

**实现步骤:**

- 添加坐标

  ​		

  ```xml
  		<!--加密-->
          <dependency>
              <groupId>com.github.ulisesbocchio</groupId>
              <artifactId>jasypt-spring-boot-starter</artifactId>
              <version>2.1.0</version>
          </dependency>
  
  ```

  

- 启动类添加注解

  ​	@EnableEncryptableProperties //开启加密自动配置注解

- 通过测试类生成密码

  ​	

  ```java
  @Test//加密
      public void testEncrypt() throws Exception {
          StandardPBEStringEncryptor standardPBEStringEncryptor = new StandardPBEStringEncryptor();
          EnvironmentPBEConfig config = new EnvironmentPBEConfig();
          // 加密的算法，这个算法是默认的
          config.setAlgorithm("PBEWithMD5AndDES");
          // 加密的密钥，随便自己填写，很重要千万不要告诉别人
          config.setPassword("Hello");
          standardPBEStringEncryptor.setConfig(config);
          //自己的密码
          String plainText = "123456";
          String encryptedText = standardPBEStringEncryptor.encrypt(plainText);
          System.out.println(encryptedText);
  
      }
  
      @Test//解密
      public void testDe() throws Exception {
          StandardPBEStringEncryptor standardPBEStringEncryptor = new StandardPBEStringEncryptor();
          EnvironmentPBEConfig config = new EnvironmentPBEConfig();
          config.setAlgorithm("PBEWithMD5AndDES");
          config.setPassword("Hello");
          standardPBEStringEncryptor.setConfig(config);
          //加密后的密码
          String encryptedText = "rT8euvjae4Ie+8MdwWrE4Q==";
          String plainText = standardPBEStringEncryptor.decrypt(encryptedText);
          System.out.println(plainText);
      }
  ```

  

- 配置yaml文件

  ​	

  ```
  jasypt:
    encryptor:
      password: Hello  #配置密钥
  ```

  

### 3.4Druid监控平台

1.数据源改为Druid

2.配置：filters: stat,wall,logback 并注入   数据源，Servlet, Filter

```java
@Configuration
public class DruidConfig {

    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid(){
        return new DruidDataSource();
    }

    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean<>();
        bean.setServlet(new StatViewServlet());
        bean.setUrlMappings(Arrays.asList("/druid/*"));
        HashMap<String, String> map = new HashMap<>();
        map.put(StatViewServlet.PARAM_NAME_USERNAME,"admin");
        map.put(StatViewServlet.PARAM_NAME_PASSWORD,"123456");

        bean.setInitParameters(map);
        return bean;
    }

    @Bean
    public FilterRegistrationBean webStatFilter(){
        FilterRegistrationBean<Filter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new WebStatFilter());
        HashMap<String, String> map = new HashMap<>();
        map.put(WebStatFilter.PARAM_NAME_EXCLUSIONS,"*.js,*.css,/druid/*");

        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }

}
```



## 四.**SpringBoot 监控**

SpringBoot自带监控功能Actuator [ˈæktjʊeɪtə]，可以帮助实现对程序内部运行情况监控，比如监控状况、Bean加载情况、配置属性 、日志信息等

**使用步骤**

① 导入依赖坐标 

```xml
<dependency> 
	<groupId>org.springframework.boot</groupId> 
	<artifactId>spring-boot-starter-actuator</artifactId> 
</dependency> 
```

② 访问http://localhost:8080/actuator访问后可以通过json.cn查看json

注意:使用时一定要先访问一个普通接口，否则不开启监控，报错404

![1635757599794](1635757599794.png)

**具体使用**

![1635759378177](1635759378177.png)

![1635759462159](1635759462159.png)

http://localhost:8080/actuator/beans查看容器中所有bean对象

http://localhost:8080/actuator/mapping查看容器中所有接口



### **Spring Boot Admin**

- Spring Boot Admin是一个开源社区项目，用于管理和监控SpringBoot应用程序。 

-  Spring Boot Admin 有两个角色，客户端(Client)和服务端(Server)。 

-  应用程序作为Spring Boot Admin Client向为Spring Boot Admin Server注册 

-  Spring Boot Admin Server 的UI界面将Spring Boot Admin Client的Actuator Endpoint上的一些

  监控信息。



**使用步骤**

**admin-server：** 

① 创建 admin-server 模块 

② 导入依赖坐标 admin-starter-server 

③ 在引导类上启用监控功能@EnableAdminServer 

**admin-client：** 

① 创建 admin-client 模块 

② 导入依赖坐标 admin-starter-client 

③ 配置相关信息：server地址等 

④ 启动server和client服务，访问server





## 五.SpringBoot整合JPA

### ORM概述[了解]

ORM（Object-Relational Mapping） 表示对象关系映射。在面向对象的软件开发中，通过ORM，就可以把对象映射到关系型数据库中。只要有一套程序能够做到建立对象与数据**库的关联，操作对象就可以直接操作数据库数据，就可以说这套程序实现了ORM对象关系映射**

 简单的说：ORM就是建立实体类和数据库表之间的关系，从而达到操作实体类就相当于操作数据库表的目的。

### **5.1** **为什么使用ORM**

当实现一个应用程序时（不使用O/R Mapping），我们可能会写特别多数据访问层的代码，从数据库保存数据、修改数据、删除数据，而这些代码都是重复的。而使用ORM则会大大减少重复性代码。对象关系映射（Object Relational Mapping，简称ORM），主要实现程序对象到关系数据库数据的映射。

### 5.2 常见ORM框架

常见的orm框架：Mybatis（ibatis）、Hibernate、Jpa





### 5.3hibernate概述

Hibernate是一个开放源代码的对象关系映射框架，它对JDBC进行了非常轻量级的对象封装，它将POJO与数据库表建立映射关系，是一个全自动的orm框架，hibernate可以自动生成SQL语句，自动执行，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库



### 5.4 JPA概述

JPA(Java Persistence API)是Sun官方提出的Java持久化**规范**。它为Java开发人员提供了一种对象/关联映射工具来管理Java应用中的关系数据。他的出现主要是为了简化现有的持久化开发工作和整合ORM技术，结束现在Hibernate，TopLink，JDO等ORM框架各自为营的局面。值得注意的是，JPA是在充分吸收了现有Hibernate，TopLink，JDO等ORM框架的基础上发展而来的，具有易于使用，伸缩性强等优点。从目前的开发社区的反应上看，JPA受到了极大的支持和赞扬，其中就包括了Spring与EJB3.0的开发团队。

JPA通过JDK 5.0注解描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中

> 注意:JPA是一套规范，不是一套产品，那么像Hibernate,TopLink,JDO他们是一套产品，如果说这些产品实现了这个JPA规范，那么我们就可以叫他们为JPA的实现产品。



### 5.5 JPA的优势

**1. 标准化**

JPA 是 JCP 组织发布的 Java EE 标准之一，因此任何声称符合 JPA 标准的框架都遵循同样的架构，提供相同的访问API，这保证了基于JPA开发的企业应用能够经过少量的修改就能够在不同的JPA框架下运行。

**2. 容器级特性的支持**

JPA框架中支持大数据集、事务、并发等容器级事务，这使得 JPA 超越了简单持久化框架的局限，在企业应用发挥更大的作用。

**3. 简单方便**

 JPA的主要目标之一就是提供更加简单的编程模型：在JPA框架下创建实体和创建Java 类一样简单，没有任何的约束和限制，只需要使用 javax.persistence.Entity进行注释，JPA的框架和接口也都非常简单，没有太多特别的规则和设计模式的要求，开发者可以很容易的掌握。JPA基于非侵入式原则设计，因此可以很容易的和其它框架或者容器集成

**4. 查询能力**

   JPA的查询语言是面向对象而非面向数据库的，它以面向对象的自然语法构造查询语句，可以看成是Hibernate HQL的等价物。JPA定义了独特的JPQL（Java Persistence Query Language），JPQL是EJB QL的一种扩展，它是针对实体的一种查询语言，操作对象是实体，而不是关系数据库的表，而且能够支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能够提供的高级查询特性，甚至还能够支持子查询。

**5. 高级特性**

   JPA 中能够支持面向对象的高级特性，如类之间的继承、多态和类之间的复杂关系，这样的支持能够让开发者最大限度的使用面向对象的模型设计企业应用，而不需要自行处理这些特性在关系数据库的持久化。

### 5.6 JPA与hibernate的关系

JPA规范本质上就是一种ORM规范，注意不是ORM框架——因为JPA并未提供ORM实现，它只是制订了一些规范，提供了一些编程的API接口，但具体实现则由服务厂商来提供实现

![1606969965083](1606969965083.png)

JPA和Hibernate的关系就像JDBC和JDBC驱动的关系，JPA是规范，Hibernate除了作为ORM框架之外，它也是一种JPA实现。JPA怎么取代Hibernate呢？JDBC规范可以驱动底层数据库吗？答案是否定的，也就是说，如果使用JPA规范进行数据库操作，底层需要hibernate作为其实现类完成数据持久化工作



### 5.7 Jpa与jdbc的区别

**JDBC**

JDBC提供一种接口，它是由各种数据库厂商提供类和接口组成的数据库驱动，为多种数据库提供统一访问。我们使用数据库时只需要调用JDBC接口就行了。

 **JPA**

 JPA是Java持久层API。它是对java应用程序访问ORM（对象关系映射）框架的规范。为了我们能用相同的方法使用各种ORM框架。

不同点：

 1.使用的sql语言不同：

 	JDBC使用的是基于关系型数据库的标准SQL语言；

 	JPA通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。

 2.操作的对象不同：

 	JDBC操作的是数据，将数据通过SQL语句直接传送到数据库中执行：

 	JPA操作的是持久化对象，由底层持久化对象的数据更新到数据库中



### 5.8  Jpa基本使用中常用注解:

```properties
		@Entity
        	作用：指定当前类是实体类。
        @Table
        	作用：指定实体类和表之间的对应关系。
        	属性：
        		name：指定数据库表的名称
        @Id
        	作用：指定当前字段是主键。
        @GeneratedValue
        	作用：指定主键的生成方式。。
        	属性：
        		strategy ：指定主键生成策略。
        @Column
        	作用：指定实体类属性和数据库表之间的对应关系
        	属性：
        		name：指定数据库表的列名称。
        		unique：是否唯一  
        		nullable：是否可以为空  
        		inserttable：是否可以插入  
        		updateable：是否可以更新  
        		columnDefinition: 定义建表时创建此列的DDL  
        		secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表									的名字搭建开发环境[重点]
```





## 六.spring data jpa

### 6.1 Spring Data JPA的概述

Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架，可使开发者用极简的代码即可实现对数据的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！

> spring data jpa让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现

### 6.2 Spring Data JPA的特性

SpringData Jpa 极大简化了数据库访问层代码。 如何简化的呢？ 使用了SpringDataJpa，我们的dao层中只需要写接口，就自动具有了增删改查、分页查询等方法

​	

### 6.3 Spring Data JPA 与 JPA和hibernate之间的关系

**JPA**是一套规范，内部是有接口和抽象类组成的。

**hibernate**是一套成熟的ORM框架，而且Hibernate实现了JPA规范，所以也可以称hibernate为JPA的一种实现方式，我们使用JPA的API编程，意味着站在更高的角度上看待问题（面向接口编程）

**Spring Data JPA**是Spring提供的一套对JPA操作更加高级的封装，是在JPA规范下的专门用来进行数据持久化的解决方案。

![1606984644511](1606984644511.png)



### 扩展知识补充:什么是 Spring Data

Spring Data的使命是为数据访问提供熟悉且一致的基于Spring的编程模型，这是一个伞形项目，其中包含许多特定于给定数据库的子项目。

**Spring Data 包含多个模块：**

**Spring Data Commons** 提供共享的基础框架，适合各个子项目使用，支持跨数据库持久化

**Spring Data JPA** 

**Spring Data MongoDB**

**Spring Data Redis** 

**Spring Data for Apache Solr** 



### 6.4 Spring Data JPA基本使用

Spring Data JPA是spring提供的一款对于数据访问层（Dao层）的框架，使用Spring Data JPA，**只需要按照框架的规范提供dao接口，不需要实现类就可以完成数据库的增删改查、分页查询等方法的定义，极大的简化了我们的开发过程**

### 6.5 Spring Data JPA统一的核心接口

1. Repository 统一的根接口，其他接口继承该接口

   ​									↓子类

2. CrudRepository基本的增删改查接口,提供了最基本的对实体类CRUD操作

   ​									↓子类

3. PagingAndSortingRepository增加了分页和排序操作

   ​									↓子类

4. **JpaRepository**增加了批量操作，并重写了父接口一些方法的返回类型，用来完成基本CRUD操作

5. **JpaSpecificationExecutor**用于复杂查询(独立接口)



### 6.6 Spring Data JPA支持的查询方式

1. 继承父类JpaRepository实现基础增，删，改

2. 使用JPQL的方式查询

   使用Spring Data JPA提供的查询方法已经可以解决大部分的应用场景，但是对于某些业务来说，我们还需要灵活的构造查询条件，这时就可以使用@Query注解，结合JPQL的语句方式完成查询

3. 方法命名

   顾名思义，方法命名规则查询就是根据方法的名字，就能创建查询。只需要按照Spring Data JPA提供的方法命名规则定义方法的名称，就可以完成查询工作。
   
   按照Spring Data JPA 定义的规则，查询方法以findBy开头，涉及条件查询时，条件的属性用条件关键字连接，要注意的是：条件属性首字母需大写。框架在进行方法名解析时，会先把方法名多余的前缀截取掉，然后对剩下部分进行解析。

| **Keyword**       | **Sample**                              | **JPQL**                                     |
| ----------------- | --------------------------------------- | -------------------------------------------- |
| And               | findByLastnameAndFirstname              | … where x.lastname = ?1 and x.firstname = ?2 |
| Or                | findByLastnameOrFirstname               | … where x.lastname = ?1 or x.firstname = ?2  |
| Is,Equals         | findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                     |
| Between           | findByStartDateBetween                  | … where x.startDate between ?1 and ?2        |
| LessThan          | findByAgeLessThan                       | … where x.age < ?1                           |
| LessThanEqual     | findByAgeLessThanEqual                  | … where x.age <= ?1                          |
| GreaterThan       | findByAgeGreaterThan                    | … where x.age > ?1                           |
| GreaterThanEqual  | findByAgeGreaterThanEqual               | … where x.age >= ?1                          |
| After             | findByStartDateAfter                    | … where x.startDate > ?1                     |
| Before            | findByStartDateBefore                   | … where x.startDate < ?1                     |
| IsNull            | findByAgeIsNull                         | … where x.age is null                        |
| IsNotNull,NotNull | findByAge(Is)NotNull                    | … where x.age not null                       |
| Like              | findByFirstnameLike                     | … where x.firstname like ?1                  |
| NotLike           | findByFirstnameNotLike                  | … where x.firstname not like ?1              |

|   Keyword    |           **Sample**           |                           **JPQL**                           |      |      |
| :----------: | :----------------------------: | :----------------------------------------------------------: | ---- | ---- |
| StartingWith |  findByFirstnameStartingWith   | … where x.firstname like ?1 (parameter bound with appended %) |      |      |
|  EndingWith  |   findByFirstnameEndingWith    | … where x.firstname like ?1 (parameter bound with prepended %) |      |      |
|  Containing  |   findByFirstnameContaining    |  … where x.firstname like ?1 (parameter bound wrapped in %)  |      |      |
|   OrderBy    |  findByAgeOrderByLastnameDesc  |         … where x.age = ?1 order by x.lastname desc          |      |      |
|     Not      |       findByLastnameNot        |                   … where x.lastname <> ?1                   |      |      |
|      In      |  findByAgeIn(Collection ages)  |                     … where x.age in ?1                      |      |      |
|    NotIn     | findByAgeNotIn(Collection age) |                   … where x.age not in ?1                    |      |      |
|     TRUE     |       findByActiveTrue()       |                   … where x.active = true                    |      |      |
|    FALSE     |      findByActiveFalse()       |                   … where x.active = false                   |      |      |
|  IgnoreCase  |   findByFirstnameIgnoreCase    |            … where UPPER(x.firstame) = UPPER(?1)             |      |      |

 4.动态查询