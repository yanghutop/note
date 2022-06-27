[TOC]



# 第1章 Spring Boot概要

## 1.1 SpringBoot介绍

随着动态语言的流行（Ruby、Scala、Node.js）,  Java的开发显得格外的笨重；繁多的配置、低下的开发效率、复杂的部署流程以及第三方技术整合难度大。

在上述环境下，Spring Boot由此诞生，它的设计是为了使您能够尽可能快地启动和运行。它使用 “习惯优于配置” （项目中存在大量的配置，而 Spring Boot 内置一个习惯性的配置，让你无须手动进行配置）的理念让你的项目快速运行起来。使用 Spring Boot 很容易创建一个独立运行（运行jar，内嵌 Servlet 容器）、准生产强力的基于 Spring 框架的项目，使用 Spring Boot你可以不用或者只需要很少的 Spring 配置。提供了 J2EE 开发的一站式解决方案。

2014 年 4 月，Spring Boot 1.0.0 发布。Spring的顶级项目之一(https://spring.io)。

![1609752821377](1609752821377.png)

![1639452472585](1639452472585.png)

## 1.2 SpringBoot优点

- Create stand-alone Spring applications

  - 创建独立Spring应用

- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)

  - 内嵌web服务器

- Provide opinionated 'starter' dependencies to simplify your build configuration

  - 自动starter依赖，简化构建配置
  -  起步依赖 ，起步依赖本质上是一个Maven项目对象模型（Project Object Model，POM），定义了对其他库的传递依赖 ，这些东西加在一起即支持某项功能。 简单的说，起步依赖就是将具备某种功能的坐标打包到一起，并提供一些默认的功能

- Automatically configure Spring and 3rd party libraries whenever possible

  - 自动配置Spring以及第三方功能

- Provide production-ready features such as metrics, health checks, and externalized configuration

  - 提供生产级别的监控、健康检查及外部化配置

- Absolutely no code generation and no requirement for XML configuration

  - 无代码生成、无需编写XML

  

  > SpringBoot是整合Spring技术栈的一站式框架
  >
  > SpringBoot是简化Spring技术栈的快速开发脚手架
  >
  > Spring Boot 并不是对 Spring 功能上的增强，而是提供了一种快速使用 Spring 的方式




## 1.3 SpringBoot缺点

- 人称版本帝，迭代快，需要时刻关注变化
- 封装太深，内部原理复杂，不容易精通



## 1.4 时代背景

### 1.4.1 微服务

[James Lewis and Martin Fowler (2014)](https://martinfowler.com/articles/microservices.html)  提出微服务完整概念。https://martinfowler.com/microservices/

> In short, the **microservice architectural style** is an approach to developing a single application as a **suite of small services**, each **running in its own process** and communicating with **lightweight** mechanisms, often an **HTTP** resource API. These services are **built around business capabilities** and **independently deployable** by fully **automated deployment** machinery. There is a **bare minimum of centralized management** of these services, which may be **written in different programming languages** and use different data storage technologies.-- [James Lewis and Martin Fowler (2014)](https://martinfowler.com/articles/microservices.html)

- 微服务是一种架构风格
- 一个应用拆分为一组小型服务
- 每个服务运行在自己的进程内，也就是可独立部署和升级
- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署
- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术



### 1.4.2 分布式

![1609753367278](1609753367278.png)

**分布式的困难**

- 远程调用
- 服务发现
- 负载均衡
- 服务容错
- 配置管理
- 服务监控
- 链路追踪
- 日志管理
- 任务调度
- ......

**分布式的解决**

- SpringBoot + SpringCloud

  ![1609753420112](1609753420112.png)



## 1.5 如何学习SpringBoot

**官网文档架构**

![1609753516331](1609753516331.png)

![1609753535189](1609753535189.png)

查看版本新特性；

https://github.com/spring-projects/spring-boot/wiki#release-notes

![1609753691378](1609753691378.png)



# 第2章Spring Boot入门开发

**技术点要求**

熟练 Spring 框架的使用

熟练 Maven 依赖管理与项目构建熟练使用 Eclipse 或 IDEA

## 2.1 环境要求

环境准备：

- java version "1.8.0_181"
- Maven-3.6.1
- SpringBoot 2.x 最新版

![1592383330063](1592383330063.png)

 

## 2.2 快速构建SpringBoot项目

Spring官方提供了非常方便的工具让我们快速构建应用

Spring Initializr：https://start.spring.io/

**项目创建方式一：使用Spring Initializr 的 Web页面创建项目**

1. 打开  https://start.spring.io/

2. 填写项目信息

3. 点击”Generate Project“按钮生成项目；下载此项目

4. 解压项目包，并用IDEA以Maven项目导入，一路下一步即可，直到项目导入完毕。

5. 如果是第一次使用，可能速度会比较慢，包比较多、需要耐心等待一切就绪。




**项目创建方式二：使用 IDEA 直接创建项目**

1. 创建一个新项目

2. 选择spring initalizr ， 可以看到默认就是去官网的快速构建工具那里实现

3. 填写项目信息

4. 选择初始化的组件（初学勾选 Web 即可）

5. 填写项目路径

6. 等待项目构建成功




**项目创建方式三：使用 IDEA 创建Maven项目并改造为springBoot**

**项目结构分析：**

1. 程序的主启动类
2. 一个 application.properties 配置文件

3. 一个 测试类

4. 一个 pom.xml




### 2.2.1 实现处理请求案例

1、在主程序的同级目录下，新建一个controller包，一定要在同级目录下，否则识别不到

2、在包中新建一个HelloController类

3、编写完毕后，从主程序启动项目，浏览器发起请求，看页面返回；控制台输出了 Tomcat 访问的端口号！



### 2.2.2 项目打jar包

![1639452988818](1639452988818.png)

使用指令  java   -jar   项目名



### 2.2.3 banner修改

1. 项目下的 resources 目录下新建一个banner.txt 即可。

2. 图案可以到：https://www.bootschool.net/ascii 这个网站生成，然后拷贝到文件中即可！




## 2.3 运行原理初步探究

### 2.3.1 **起步依赖原理分析**

**1） spring-boot-starter-parent**

**2） spring-boot-starter-web**

**小结** 

- 在spring-boot-starter-parent中定义了各种技术的版本信息，组合了一套最优搭配的技术版本。 

- 在各种starter中，定义了完成该功能需要的坐标合集，其中大部分版本信息来自于父工程。 

- 我们的工程继承parent，引入starter后，通过依赖传递，就可以简单方便获得需要的jar包，并且不会存在 

  版本冲突等问题。
  
  ```java
  spring-boot-starter-web==>spring-boot-starter-parent==>spring-boot-dependencies==>
  发现很多坐标，并且还能找到spring-boot-starter-web==>在查找看到了所有依赖包
  ```
  
  

无需关注版本号，自动版本仲裁

```pro
1、引入依赖默认都可以不写版本
2、引入非版本仲裁的jar，要写版本号。
```

可以修改默认版本号

```xml
1、查看spring-boot-dependencies里面规定当前依赖的版本用的key。
2、在当前项目里面重写配置
    <properties>
        <mysql.version>5.1.43</mysql.version>
    </properties>
```

### 2.3.2 自动配置

- 自动配好Tomcat

  - 引入Tomcat依赖。
  - 配置Tomcat

```xml
	<dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-tomcat</artifactId>
          <version>2.3.4.RELEASE</version>
          <scope>compile</scope>
    </dependency>
```

- 自动配好SpringMVC

- - 引入SpringMVC全套组件
  - 自动配好SpringMVC常用组件（功能）
  - 思考springMVC之前配置什么，可以在控制台一一查找到
  - ![1609803045194](1609803045194.png)
  - ![1609803093526](1609803093526.png)

- 自动配好Web常见功能，如：字符编码问题(也可以在控制台搜索到字符过滤器)

- - SpringBoot帮我们配置好了所有web开发的常见场景

- 默认的包结构

- - 主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来
  - ![1609803187296](1609803187296.png)
  - 无需以前的包扫描配置
  - 想要改变扫描路径，@SpringBootApplication(scanBasePackages=**"com.apesource"**)

- - - 或者@ComponentScan 指定扫描路径

```
@SpringBootApplication
等同于
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan("com.apesource.boot")
```

- 各种配置拥有默认值

- - 默认配置最终都是映射到某个类上，如：MultipartProperties
  - ![1609803258891](1609803258891.png)
  - 配置文件的值最终会绑定每个类上，这个类会在容器中创建对象

- 按需加载所有自动配置项

- - 非常多的starter
  - 引入了哪些场景这个场景的自动配置才会开启
  - SpringBoot所有的自动配置功能都在 spring-boot-autoconfigure 包里面
  - 

- ......

  



# 第3章 Spring Boot 核心配置

## 3.1 Spring Boot配置文件分类

SpringBoot是基于约定的，所以很多配置都有默认值，但如果想使用自己的配置替换默认配置的话，就可以使用 

application.properties或者application.yml（application.yaml）进行配置



- application.properties

  - 语法结构 ：key=value

  - ```properties
    server.port=8081
    ```

    

- application.yml

  - 语法结构 ：key：空格 value  （冒号后面必须要有空格）

  - ```yaml
    server:
    	port: 8081
    ```

- **小结** 

  - SpringBoot提供了2种配置文件类型：properteis和yml/yaml 
  - 默认配置文件名称：application 
  - 在同一级目录下优先级为：properties > yml > yaml



## 3.2 YAML概述

YAML全称是 YAML Ain't Markup Language 。YAML是一种直观的能够被电脑识别的的数据数据序列化格式，并

且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，比如： C/C++, 

Ruby, Python, Java, Perl, C#, PHP等。YML文件是以数据为核心的，比传统的xml方式更加简洁。 

YAML文件的扩展名可以使用.yml或者.yaml。



 properties: 

```properties
server.port=8080
server.address=127.0.0.1
```

xml: 

```xml
<server> 
	<port>8080</port> 
	<address>127.0.0.1</address> 
</server> 
```

yml: 

```yaml
server: 
	port: 8080 
	address: 127.0.0.1 
```

==简洁，以数据为核心==



## 3.3 YAML基础语法

1. 大小写敏感 

2. 数据值前边必须有空格，作为分隔符 

3. 使用缩进表示层级关系 

4. 缩进时不允许使用Tab键，只允许使用空格（各个系统 Tab对应的 空格数目可能不同，导致层次混乱）。 

5. 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可 

6. "#"表示注释，从这个字符一直到行尾，都会被解析器忽略。

7. 字符串无需加引号，如果要加，''与""表示字符串内容 会被 转义/不转义

**YAML：数据格式**

- ​	字面量：普通的值  [ 数字，布尔值，字符串  ]


```yaml
msg1: 'apesource \n edu' # 单引忽略转义字符  输出 ：apesource\n   edu
msg2: "apesource \n edu" # 双引识别转义字符  输出 ：apesource换行  edu
```

- ​	数组：一组按次序排列的值（ List、set ）

```yaml
address:
	- beijing
	- shanghai
# 行内写法
company: [阿里巴巴,华为,腾讯,字节跳动]
```

- ​	对象、Map（键值对）

```yaml
person:
	name: wangzhuo
# 行内写法
person: {name: wangzhuo}
```

**YAML：参数引用**

```yaml
name: wangzhuo

person:
	name: xuelaoshi 
	pet: ${name}	# 引用上边定义的name值
	name: xuls${random.uuid} # 配置文件占位符,随机uuid
	name: ${person.name}_真帅
```

 

**YAML：小结**

1） 配置文件类型 

- properties：和以前一样 (设置文件的语言编码UTF-8)
- yml/yaml：注意空格 

2） yaml：简洁，以数据为核心 

-  基本语法 
  - 大小写敏感 
  - 数据值前边必须有空格，作为分隔符 
  - 使用空格缩进表示层级关系，相同缩进表示同一级 

- 数据格式 
  - 对象 
  - 数组: 使用 " - "表示数组每个元素 
  - 纯量 

- 参数引用 
  - ${key}
  
    

## 3.4 **读取配置内容**

yaml文件更强大的地方在于，他可以给我们的实体类直接注入匹配值！

1、在springboot项目中的resources目录下新建一个文件 application.yml

2、编写一个实体类 Dog；

```java
package com.apesource.springboot.pojo;

@Component  //注册bean到容器中
public class Dog {
    private String name;
    private Integer age;
    
    //有参无参构造、get、set方法、toString()方法  
}
```

3、思考，我们原来是如何给bean注入属性值的！@Value，给狗狗类测试一下：

```java
@Component //注册bean
public class Dog {
    @Value("阿黄")
    private String name;
    @Value("18")
    private Integer age;
}
```

4、在SpringBoot的测试类下注入狗狗输出一下；

```java
@SpringBootTest
class DemoApplicationTests {

    @Autowired //将狗狗自动注入进来
    Dog dog;

    @Test
    public void contextLoads() {
        System.out.println(dog); //打印看下狗狗对象
    }

}
```

加载指定的配置文件**@PropertySource ：**加载指定的配置文件；

我们去在resources目录下新建一个**dog.properties**文件

```properties
name=阿黄
age=18
```

然后在我们的代码中指定加载person.properties文件

```java
@PropertySource(value = "classpath:dog.properties")
@Component //注册bean
public class Dog{

    @Value("${name}")
    private String name;
    @Value("${age}")
    private Integer age; 
}
```



 5、我们在编写一个复杂一点的实体类：Person 类

```java
@Component //注册bean到容器中
public class Person {
    
    private String name;
    private integer age;
    private boolean marry;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
    
    //有参无参构造、get、set方法、toString()方法  
}
```

6、我们来使用yaml配置的方式进行注入，大家写的时候注意区别和优势，我们编写一个yaml配置！

```yml
person1:
  name: wangls
  age: 18
  marry: true
  birth: 1990/10/19
  maps: {k1: v1,k2: v2}
  lists:
   - code
   - bodybuilding
   - music
  dog:
    name: summer
    age: 1
```

7、我们刚才已经把person这个对象的所有值都写好了，我们现在来注入到我们的类中！

```java
/*
@ConfigurationProperties作用：
将配置文件中配置的每一个属性的值，映射到这个组件中；
告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
参数 prefix = “person” : 将配置文件中的person下面的所有属性一一对应
*/
@Component //注册bean
@ConfigurationProperties(prefix = "person1")
public class Person {
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
} 
```

8、IDEA 提示，springboot配置注解处理器没有找到，让我们看文档，我们可以查看文档，找到一个依赖！

![img](1418974-20200310171716557-1171634646.png)

```xml
<!-- 导入配置文件处理器，配置文件进行绑定就会有提示，需要重启 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
  <optional>true</optional>
</dependency> 
```

9、确认以上配置都OK之后，我们去测试类中测试一下：

```java
@SpringBootTest
class DemoApplicationTests {

    @Autowired
    Person person; //将person自动注入进来

    @Test
    public void contextLoads() {
        System.out.println(person); //打印person信息
    }

}
```



**对比小结**

@Value这个使用起来并不友好！我们需要为每个属性单独注解赋值，比较麻烦；我们来看个功能对比图

 ![img](1418974-20200310172022780-374285033.png)

 

1、@ConfigurationProperties只需要写一次即可 ， @Value则需要每个字段都添加

2、松散绑定：这个什么意思呢? 比如我的yml中写的**last-name**，这个和lastName是一样的， - 后面跟着的字母默认是大写的。这就是松散绑定。可以测试一下

3、JSR303数据校验 ， 这个就是我们可以在字段是增加一层过滤器验证 ， 可以保证数据的合法性

4、复杂类型封装，yml中可以封装对象 ， 使用value就不支持

**结论：**

配置yml和配置properties都可以获取到值 ， 强烈推荐 yml；

如果我们在某个业务中，只需要获取配置文件中的某个值，可以使用一下 @value；

如果说，我们专门编写了一个JavaBean来和配置文件进行一一映射，就直接@configurationProperties，不要犹豫！



## 3.5  JSR303数据校验

对于`web`服务来说，为防止非法参数对业务造成影响，在`Controller`层一定要做参数校验的！大部分情况下，请求参数分为如下两种形式：

1. `POST`、`PUT`请求，使用`requestBody`传递参数；
2. `GET`请求，使用`requestParam/PathVariable`传递参数。

### 什么是 JSR303 标准？

**JSR**的全称是**Java Specification Requests（**Java 规范提案**）**，是指向**JCP ( Java Community Process )**提出新增一个标准化技术规范的正式请求。

`Java API`规范(`JSR303`)定义了`Bean`校验的标准`validation-api`，但没有提供实现。`hibernate validation`是对这个规范的实现，并增加了校验注解如`@Email`、`@Length`等。`Spring Validation`是对`hibernate validation`的二次封装，用于支持`spring mvc`参数自动校验。接下来，我们以`spring-boot`项目为例，介绍`Spring Validation`的使用

### JSR303校验注解的使用步骤

1.添加依赖，导入spring-boot-starter-validation启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

2.在实体类需要校验的成员字段（Field）上，添加校验注解。

```java
@Component  //注册bean到容器中
//@ConfigurationProperties(prefix = "dog")
//@Validated
public class Dog {
    @NotNull(message = "狗名不能为空")
    private String name;
    @NotNull(message = "狗命不能为空")
    @Max(value = 15,message = "狗命拿来")
    private Integer age;
    @Min(value = -1,message = "狗还是个细胞")
    private Integer length;
    @Email(message = "邮箱必须合法")
    private String email;
}
```

3.在Controller控制器的校验参数前，使用@Valid注解开启校验，使用**BindingResult** 绑定校验结果

```java
@RequestMapping("/validationDemo")
    public String validationDemo(@Valid Dog dog, BindingResult result){
        //得到所有错误信息计数
        int errorCount = result.getErrorCount();
        //错误数大于0
        if (errorCount>0){
            //得到所有错误
            List<FieldError> fieldErrors = result.getFieldErrors();
            //迭代错误
            for (FieldError fieldError:fieldErrors) {
                //错误信息
                System.out.println( "属性：{},"+fieldError.getField()+"传来的值是：{}"+fieldError.getRejectedValue()+",出错的提示消息：{}"+fieldError.getDefaultMessage());
            }
            return "数据不合法";
        }else{
            return "数据合法";
        }
    }
```

**JSR303校验注解的分类**

**值校验：**

```java
  // 被注解的元素必须为null
  @Null(message = "必须为null") 
  
  // 被注解的元素必须不为null
  @NotNull(message = "必须不为null") 
  
  // 校验注解的元素值不为空（不为null、去除首位空格后长度为0）,类型为String
  @NotBlank(message = "必须不为空") 
  
  // 校验注解的元素值不为null且不为空（字符串长度不为0、集合大小不为0）,类型为CharSequence、Collection、Map、Array
  @NotEmpty(message = "必须不为null且不为空") 
  
  // 被注解的元素必须为true，并且类型为boolean
  @AssertTrue(message = "必须为true") 
  
  // 被注解的元素必须为false，并且类型为boolean
  @AssertFalse(message = "必须为false")
```

**范围校验**

```java
// 被注解的元素其值必须大于等于最小值，并且类型为int，long，float，double
@Min(value = 18, message = "必须大于等于18")

// 被注解的元素其值必须小于等于最小值，并且类型为int，long，float，double
@Max(value = 18, message = "必须小于等于18") 

// 校验注解的元素值大于等于@DecimalMin指定的value值，并且类型为BigDecimal
@DecimalMin(value = "150", message = "必须大于等于150") 

// 校验注解的元素值小于等于@DecimalMax指定的value值 ，并且类型为BigDecimal
@DecimalMax(value = "300", message = "必须大于等于300")

// 校验注解的元素值在最小值和最大值之间，并且类型为BigDecimal，BigInteger，CharSequence，byte，short，int，long。
@Range(max = 80, min = 18, message = "必须大于等于18或小于等于80") 

// 被注解的元素必须为过去的一个时间，并且类型为java.util.Date
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") @Past(message = "必须为过去的时间")

// 被注解的元素必须为未来的一个时间，并且类型为java.util.Date
@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") @Future(message = "必须为未来的时间")
```

**长度校验：**

```java
// 被注解的元素的长度必须在指定范围内，并且类型为String，Array，List，Map
@Size(max = 11, min = 7, message = "长度必须大于等于7或小于等于11")

// 校验注解的元素值长度在min和max区间内 ，并且类型为String
@Length(max = 11, min = 7, message = "长度必须大于等于7或小于等于11")
```

**格式校验：**

```java
// 校验注解的元素值的整数位数和小数位数上限 ，并且类型为float，double，BigDecimal。
@Digits(integer=3,fraction = 2,message = "整数位上限为3位，小数位上限为2位")

// 被注解的元素必须符合指定的正则表达式，并且类型为String
@Pattern(regexp = "\\d{11}",message = "必须为数字，并且长度为11")

// 校验注解的元素值是Email，也可以通过regexp和flag指定自定义的email格式，类型为String
@Email(message = "必须是邮箱")
```



###  **Controller统一异常处理**

@ControllerAdvice：统一为Controller进行"增强"

@ExceptionHandler : 异常处理 

```java
import org.springframework.validation.BindException;
import org.springframework.web.bind.annotation.*;

@ControllerAdvice
public class ZeroAdvice {
    // 验证参数未使用@RequestBody注解
    @ExceptionHandler(ArithmeticException.class)
    @ResponseBody//选择添加
    public String handlerBindException(ArithmeticException ex, HttpServletRequest request){
		return "全局异常处理器成功"; // 跳转至统一提示错误信息的视图
    }
}
```





## 3.6 多环境切换**profile**

我们在开发Spring Boot应用时，通常同一套程序会被安装到不同环境，比如：开发、测试、生产等。其中数据库

地址、服务 器端口等等配置都不同，如果每次打包时，都要修改配置文件，那么非常麻烦。profile功能就是来进

行动态配置切换的；

命名语法：

例如：application-环境简称.properties/yml

- application-dev.properties/yml 开发环境 
- application-test.properties/yml 测试环境 
- application-pro.properties/yml 生产环境

**1） profile配置方式** 

- 多profile文件方式 
- yml多文档方式 

**2） profile激活方式** 

- 配置文件 

- 虚拟机参数 

- 命令行参数

  配置激活:

  ```properties
  #比如在配置文件中指定使用dev环境，我们可以通过设置不同的端口号进行测试；
  #我们启动SpringBoot，就可以看到已经切换到dev下的配置了；
  spring.profiles.active=dev
  ```

  yaml的多文档:与properties配置文件中一样，但是使用yml去实现不需要创建多个配置文件，更加方便

  ```yml
  server:
    port: 8081
  #选择要激活那个环境块
  spring:
    profiles:
      active: prod
  
  ---
  server:
    port: 8083
  spring:
    profiles: dev #配置环境的名称
  
  ---
  server:
    port: 8084
  spring:
    profiles: prod  #配置环境的名称
  ```

  

**Profile-小结**

- profile是用来完成不同环境下，配置动态切换功能的。 

- profile配置方式 

  - 多profile文件方式：提供多个配置文件，每个代表一种环境。 
    - application-dev.properties/yml 开发环境 
    - application-test.properties/yml 测试环境 
    - application-pro.properties/yml 生产环境 

  - yml多文档方式： 
    - 在yml中使用 --- 分隔不同配置 

- profile激活方式 
  - 配置文件： 再配置文件中配置：spring.profiles.active=dev 
  - 虚拟机参数：在VM options 指定：-Dspring.profiles.active=dev 
  - 命令行参数：java –jar xxx.jar --spring.profiles.active=dev



 

## 3.7 配置文件加载位置

### 3.7.1 **内部配置加载顺序**

```properties
Springboot程序启动时，会从以下位置加载配置文件：
1. file:./config/：当前项目下的/config目录下
2. file:./ ：当前项目的根目录
3. classpath:/config/：classpath的/config目录
4. classpath:/ ：classpath的根目录
加载顺序为上文的排列顺序，高优先级配置的属性会生效
```



### 3.7.2 **外部配置加载顺序**

通过官网查看外部属性加载顺序： 

https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；这种情况，一般是后期运维做的多，相同配置，外部指定的配置文件优先级最高

```properties
java -jar spring-boot-config.jar --spring.config.location=F:/application.properties
```


