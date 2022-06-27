 

[TOC]

# 第4章Spring Boot 的Web开发

**官网**

![1609991614535](1609991614535.png)

![1609991640188](1609991640188.png)

![1609991721689](1609991721689.png)

![1609991739849](1609991739849.png)

## **SpringMVC自动配置概览**

Spring Boot provides auto-configuration for Spring MVC that **works well with most applications.(大多场景我们都无需自定义配置)**

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

- - 内容协商视图解析器和BeanName视图解析器

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content))).

- - 静态资源（包括webjars）

- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.

- - 自动注册 `Converter，GenericConverter，Formatter `

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)).

- - 支持 `HttpMessageConverters` （后来我们配合内容协商理解原理）

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-message-codes)).

- - 自动注册 `MessageCodesResolver` （国际化用）

- Static `index.html` support.

- - 静态index.html 页支持

- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-favicon)).

- - 自定义 `Favicon`  

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)).

- - 自动使用 `ConfigurableWebBindingInitializer` ，（DataBinder负责将请求数据绑定到JavaBean上）



## 4.1 静态资源映射规则

**我们先来聊聊这个静态资源映射规则：**

SpringBoot中，SpringMVC的web配置都在 WebMvcAutoConfiguration 这个配置类里面；

我们可以去看看 WebMvcAutoConfigurationAdapter 中有很多配置方法；

有一个方法：addResourceHandlers 添加资源处理

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        // 已禁用默认资源处理
        logger.debug("Default resource handling disabled");
        return;
    }
    // 缓存控制
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    // webjars 配置
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                                             .addResourceLocations("classpath:/META-INF/resources/webjars/")
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
    // 静态资源配置
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                                             .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
}
```

读一下源代码：比如所有的 /webjars/** ， 都需要去 classpath:/META-INF/resources/webjars/ 找对应的资源；

### 第一种 webjars 

Webjars本质就是以jar包的方式引入我们的静态资源 ， 我们以前要导入一个静态资源文件，直接导入即可。

使用SpringBoot需要使用Webjars，我们可以去搜索一下：

网站：https://www.webjars.org

![1597229920823](1597229920823.png)

要使用jQuery，我们只要要引入jQuery对应版本的pom依赖即可！

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

导入完毕，查看webjars目录结构，并访问Jquery.js文件！

![img](1418974-20200317124625447-915223066.png)

访问：只要是静态资源，SpringBoot就会去对应的路径寻找资源，我们这里访问：http://localhost:8080/webjars/jquery/3.4.1/jquery.js

![img](1418974-20200317124646310-1681461698.png)

### 第二种静态资源映射规则

**帮助文档**

![1609992268639](1609992268639.png)

**总结：**

只要静态资源放在类路径下： called `/static` (or `/public` or `/resources` or `/META-INF/resources`

访问 ： 当前项目根路径/ + 静态资源名 

原理： 静态映射/**。

请求进来，先去找Controller看能不能处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面

![1609992441752](1609992441752.png)

**静态资源访问前缀**

```yaml
spring:
	mvc:
		static-path-pattern: /apesource/**
```

![1609992583830](1609992583830.png)

### 第三种自定义静态资源路径（不常用）

![1609992777456](1609992777456.png)

```properties
spring.resources.static-locations=classpath:/coding/,classpath:/apesource/
```

一旦自己定义了静态文件夹的路径，原来的自动配置就都会失效了！



## 4.2 首页处理

静态资源文件夹说完后，我们继续向下看源码！可以看到一个欢迎页的映射，就是我们的首页！

```java
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
                                                           FormattingConversionService mvcConversionService,
                                                           ResourceUrlProvider mvcResourceUrlProvider) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
        new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(), // getWelcomePage 获得欢迎页
        this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
    return welcomePageHandlerMapping;
}
```

点进去继续看

```java
private Optional<Resource> getWelcomePage() {
    String[] locations = getResourceLocations(this.resourceProperties.getStaticLocations());
    // ::是java8 中新引入的运算符
    // Class::function的时候function是属于Class的，应该是静态方法。
    // this::function的funtion是属于这个对象的。
    // 简而言之，就是一种语法糖而已，是一种简写
    return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
}
// 欢迎页就是一个location下的 index.html 而已
private Resource getIndexHtml(String location) {
    return this.resourceLoader.getResource(location + "index.html");
}
```

分析结果: 收到  " /** "   请求后，会在四个静态资源目录下与根路径查找 (按顺序) index.html页面；

```yml
classpath:/META-INF/resources/
classpath:/resources/
classpath:/static/
classpath:/public/
```

- 静态资源路径下  index.html
  - 可以配置静态资源路径
  - 但是不可以配置静态资源的访问前缀。否则导致 index.html不能被默认访问
- controller能处理/index

## 4.3 关于网站图标说明

![1597231508892](1597231508892.png)

与其他静态资源一样，Spring Boot在配置的静态内容位置中查找 favicon.ico文件,如果存在这样的文件，它将自动用作应用程序的favicon图标，如果不存在则会使用默认图标（设置未显示请清除浏览器缓存）





## 4.4 Thymeleaf 模板引擎

前端交给我们的页面，是html页面。如果是我们以前开发，我们需要把他们转成jsp页面，jsp好处就是当我们查出一些数据转发到JSP页面以后，我们可以用jsp轻松实现数据的显示，及交互等。

jsp支持非常强大的功能，包括能写Java代码，但是呢，我们现在的这种情况，SpringBoot这个项目首先是以jar的方式，不是war，像第二，我们用的还是嵌入式的Tomcat，所以呢，**他现在默认是不支持jsp的**。

那不支持jsp，如果我们直接用纯静态页面的方式，那给我们开发会带来非常大的麻烦，那怎么办呢？

**SpringBoot推荐你可以来使用模板引擎：**

模板引擎，我们其实大家听到很多，其实jsp就是一个模板引擎，还有用的比较多的freemarker，包括SpringBoot给我们推荐的Thymeleaf，模板引擎有非常多，但再多的模板引擎，他们的思想都是一样的

 ![img](1418974-20200318131836219-1993983957.png)

模板引擎的作用就是我们来写一个页面模板，比如有些值呢，是动态的，我们写一些表达式。而这些值，从哪来呢，就是我们在后台封装一些数据。然后把这个模板和这个数据交给我们模板引擎，模板引擎按照我们这个数据帮你把这表达式解析、填充到我们指定的位置，然后把这个数据最终生成一个我们想要的内容给我们写出去，这就是我们这个模板引擎，不管是jsp还是其他模板引擎，都是这个思想。只不过呢，就是说不同模板引擎之间，他们可能这个语法有点不一样。我主要来介绍一下SpringBoot给我们推荐的Thymeleaf模板引擎，这模板引擎呢，是一个高级语言的模板引擎，他的这个语法更简单。而且呢，功能更强大。

### 使用步骤:

#### A.引入Thymeleaf

```xml
<!--thymeleaf-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Maven会自动下载jar包，我们可以去看下下载的东西；

![img](1418974-20200318131944764-315816598.png)

 

**Thymeleaf分析**

一起看下Thymeleaf的自动配置规则,我们去找一下Thymeleaf的自动配置类：ThymeleafProperties

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
    private Charset encoding;
}
```

我们可以在其中看到默认的前缀和后缀！

我们只需要把我们的html页面放在类路径下的templates下，thymeleaf就可以帮我们自动渲染了。

使用thymeleaf什么都不需要配置，只需要将他放在指定的文件夹下即可！



#### B.导入Thymeleaf 的名称空间

```html
<html xmlns:th="http://www.thymeleaf.org">
```

#### C. 使用Thymeleaf基础语法进行数据渲染



### 语法学习:

![1597235191363](1597235191363.png)

#### 一、标准表达式语法

`${...}` 变量表达式，**用途:**获取请求域、session域、对象等值

`@{...}` 链接表达式，**用途:**生成链接

`#{...}` 消息表达式，**用途:**获取国际化等值

`~{...}` 代码块表达式，**用途:**jsp:include 作用，引入公共页面片段

`*{...}` 选择变量表达式，**用途:**获取上下文对象值

变量表达式使用频率最高，其功能也是非常的丰富。然后是消息表达式，再是链接表达式，最后是变量表达式

**表达式内可以编写的内容如下：**

**1、字面量**

文本值: **'one text'** **,** **'Another one!'** **,…**数字: **0** **,** **34** **,** **3.0** **,** **12.3** **,…**布尔值: **true** **,** **false**

空值: **null**

变量： one，two，.... 变量不能有空格

**2、文本操作**

字符串拼接: **+**

变量替换: **|The name is ${name}|** 

**3、数学运算**

运算符: + , - , * , / , %

**4、布尔运算**

运算符:  **and** **,** **or**

一元运算: **!** **,** **not** 

**5、比较运算**

比较: **>** **,** **<** **,** **>=** **,** **<=** **(** **gt** **,** **lt** **,** **ge** **,** **le** **)**等式: **==** **,** **!=** **(** **eq** **,** **ne** **)** 

**6、条件运算**

If-then: **(if) ? (then)**

If-then-else: **(if) ? (then) : (else)**

Default: (value) **?: (defaultvalue)** 

举例：

###### ~{...} 代码块表达式

支持两种语法结构

推荐：`~{templatename::fragmentname}`

支持：`~{templatename::#msg1}`

templatename：模版名，Thymeleaf会根据模版名解析完整径：/resources/templates/templatename.html，要注意文件的路径。

fragmentname：片段名，Thymeleaf通过th:fragment声明定义代码块，即：`th:fragment="fragmentname"`

id：HTML的id选择器，使用时要在前面加上#号，不支持class选择器

###### #{...} 消息表达式

消息表达式一般用于国际化的场景。结构：`th:text="#{msg}"` 

###### @{...} 链接表达式

**链接表达式好处**

不管是静态资源的引用，form表单的请求，凡是链接都可以用`@{...}` 。这样可以动态获取项目路径，即便项目名变了，依然可以正常访问

```properties
#修改项目名，链接表达式会自动修改路径，避免资源文件找不到
server.context-path=/apesource
```

**链接表达式结构**

无参：`@{/xxx}`

有参：`@{/xxx(k1=v1,k2=v2)}` 对应url结构：`xxx?k1=v1&k2=v2`

引入本地资源：`@{/项目本地的资源路径}`

引入外部资源：`@{/webjars/资源在jar包中的路径}`

列举：第三部分的实战引用会详细使用该表达式

```html
<link th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.css}" rel="stylesheet">
<link th:href="@{/main/css/apesource.css}" rel="stylesheet">
<form class="form-login" th:action="@{/user/login}" th:method="post" >
<a class="btn btn-sm" th:href="@{/login.html(l='zh_CN')}">中文</a>
<a class="btn btn-sm" th:href="@{/login.html(l='en_US')}">English</a>
```

###### ${...}变量表达式

变量表达式有丰富的内置方法，使其更强大，更方便。

**变量表达式功能**

一、可以获取对象的属性和方法

二、可以使用ctx，vars，locale，request，response，session，servletContext内置对象

三、可以使用dates，numbers，strings，objects，arrays，lists，sets，maps等内置方法（重点介绍）

**常用的内置对象**

一、**ctx** ：上下文对象。

二、**vars** ：上下文变量。

三、**locale**：上下文的语言环境。

四、**request**：（仅在web上下文）的 HttpServletRequest 对象。

五、**response**：（仅在web上下文）的 HttpServletResponse 对象。

六、**session**：（仅在web上下文）的 HttpSession 对象。

七、**servletContext**：（仅在web上下文）的 ServletContext 对象

这里以常用的Session举例，用户刊登成功后，会把用户信息放在Session中，Thymeleaf通过内置对象将值从session中获取。

```java
// java 代码将用户名放在session中
session.setAttribute("userinfo",username);
// Thymeleaf通过内置对象直接获取
th:text="${session.userinfo}"
```

**常用的内置方法**

一、**strings**：字符串格式化方法，常用的Java方法它都有。比如：equals，equalsIgnoreCase，length，trim，toUpperCase，toLowerCase，indexOf，substring，replace，startsWith，endsWith，contains，containsIgnoreCase等

二、**numbers**：数值格式化方法，常用的方法有：formatDecimal等

三、**bools**：布尔方法，常用的方法有：isTrue，isFalse等

四、**arrays**：数组方法，常用的方法有：toArray，length，isEmpty，contains，containsAll等

五、**lists**，**sets**：集合方法，常用的方法有：toList，size，isEmpty，contains，containsAll，sort等

六、**maps**：对象方法，常用的方法有：size，isEmpty，containsKey，containsValue等

七、**dates**：日期方法，常用的方法有：format，year，month，hour，createNow等

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title> Thymeleaf 内置方法</title>
</head>
<body>
    <h2> Thymeleaf 内置方法</h2>
    <h3>#strings </h3>
    <div th:if="${not #strings.isEmpty(apesource)}" >
        <p>Old Str : <span th:text="${apesource}"/></p>
        <p>toUpperCase : <span th:text="${#strings.toUpperCase(apesource)}"/></p>
        <p>toLowerCase : <span th:text="${#strings.toLowerCase(apesource)}"/></p>
        <p>equals : <span th:text="${#strings.equals(apesource, 'apesourcelogo')}"/></p>
        <p>equalsIgnoreCase : <span th:text="${#strings.equalsIgnoreCase(apesource, 'apesourcelogo')}"/></p>
        <p>indexOf : <span th:text="${#strings.indexOf(apesource, 's')}"/></p>
        <p>substring : <span th:text="${#strings.substring(apesource, 2, 8)}"/></p>
        <p>replace : <span th:text="${#strings.replace(apesource, 'ap', 'el')}"/></p>
        <p>startsWith : <span th:text="${#strings.startsWith(apesource, 'so')}"/></p>
        <p>contains : <span th:text="${#strings.contains(apesource, 'ce')}"/></p>
    </div>

    <h3>#numbers </h3>
    <div>
        <p>formatDecimal 整数部分随意，小数点后保留两位，四舍五入: <span th:text="${#numbers.formatDecimal(apesource, 0, 2)}"/></p>
        <p>formatDecimal 整数部分保留五位数，小数点后保留两位，四舍五入: <span th:text="${#numbers.formatDecimal(apesource, 5, 2)}"/></p>
    </div>

    <h3>#bools </h3>
    <div th:if="${#bools.isTrue(apesource)}">
        <p th:text="${apesource}"></p>
    </div>

    <h3>#arrays </h3>
    <div th:if="${not #arrays.isEmpty(apesourceArray)}">
        <p>length : <span th:text="${#arrays.length(apesourceArray)}"/></p>
        <p>contains : <span th:text="${#arrays.contains(apesourceArray, 5)}"/></p>
        <p>containsAll : <span th:text="${#arrays.containsAll(apesourceArray, apesourceArray)}"/></p>
    </div>
    <h3>#lists </h3>
    <div th:if="${not #lists.isEmpty(apesourceList)}">
        <p>size : <span th:text="${#lists.size(apesourceList)}"/></p>
        <p>contains : <span th:text="${#lists.contains(apesourceList, 0)}"/></p>
        <p>sort : <span th:text="${#lists.sort(apesourceList)}"/></p>
    </div>
    <h3>#maps </h3>
    <div th:if="${not #maps.isEmpty(apesourceMap)}">
        <p>size : <span th:text="${#maps.size(apesourceMap)}"/></p>
        <p>containsKey : <span th:text="${#maps.containsKey(apesourceMap, 'thName')}"/></p>
        <p>containsValue : <span th:text="${#maps.containsValue(apesourceMap, '#maps')}"/></p>
    </div>
    <h3>#dates </h3>
    <div>
        <p>format : <span th:text="${#dates.format(apesourceDate)}"/></p>
        <p>custom format : <span th:text="${#dates.format(apesourceDate, 'yyyy-MM-dd HH:mm:ss')}"/></p>
        <p>day : <span th:text="${#dates.day(apesourceDate)}"/></p>
        <p>month : <span th:text="${#dates.month(apesourceDate)}"/></p>
        <p>monthName : <span th:text="${#dates.monthName(apesourceDate)}"/></p>
        <p>year : <span th:text="${#dates.year(apesourceDate)}"/></p>
        <p>dayOfWeekName : <span th:text="${#dates.dayOfWeekName(apesourceDate)}"/></p>
        <p>hour : <span th:text="${#dates.hour(apesourceDate)}"/></p>
        <p>minute : <span th:text="${#dates.minute(apesourceDate)}"/></p>
        <p>second : <span th:text="${#dates.second(apesourceDate)}"/></p>
        <p>createNow : <span th:text="${#dates.createNow()}"/></p>
    </div>
</body>
</html>
```

后台给负责给变量赋值，和跳转页面

```java
@RequestMapping("varexpressions")
public String varexpressions(ModelMap map) {
  map.put("apesourceStr", "apesourcelog");
  map.put("apesourceBool", true);
  map.put("apesourceArray", new Integer[]{1,2,3,4});
  map.put("apesourceList", Arrays.asList(1,3,2,4,0));
  Map itdragonMap = new HashMap();
  itdragonMap.put("thName", "${#...}");
  itdragonMap.put("desc", "变量表达式内置方法");
  map.put("apesourceMap", itdragonMap);
  map.put("apesourceDate", new Date());
  map.put("apesourceNum", 888.888D);
  return "grammar/varexpressions";
}
```



Thymeleaf 行内表达式双中括号:[[表达式]]

```html
<input type="checkbox" /> [[${desc}]]
<p>Hello, [[${desc}]] 。。。</p>
```



#### 二、th属性

##### A.常用th属性解读

html有的属性，Thymeleaf基本都有，而常用的属性大概有七八个。其中th属性执行的优先级从1~8，数字越低优先级越高。

一、**th:text** ：设置当前元素的文本内容，相同功能的还有**th:utext**，两者的区别在于前者不会转义html标签，后者会。优先级不高：order=7

二、**th:value**：设置当前元素的value值，类似修改指定属性的还有**th:src**，**th:href**。优先级不高：order=6

三、**th:each**：遍历循环元素，和th:text或th:value一起使用。注意该属性修饰的标签位置，详细往后看。优先级很高：order=2

四、**th:if**：条件判断，类似的还有**th:unless**，**th:switch**，**th:case**。优先级较高：order=3

五、**th:insert**：代码块引入，类似的还有**th:replace**，**th:include**，三者的区别较大，若使用不恰当会破坏html结构，常用于公共代码块提取的场景。优先级最高：order=1

六、**th:fragment**：定义代码块，方便被th:insert引用。优先级最低：order=8

七、**th:object**：声明变量，一般和*{}一起配合使用，达到偷懒的效果。优先级一般：order=4

八、**th:attr**：修改任意属性，实际开发中用的较少，因为有丰富的其他th属性帮忙，类似的还有th:attrappend，th:attrprepend。优先级一般：order=5



##### B.常用th属性使用

使用Thymeleaf属性需要注意点以下五点：

一、若要使用Thymeleaf语法，首先要声明名称空间： `xmlns:th="http://www.thymeleaf.org"`

二、设置文本内容 th:text，设置input的值 th:value，循环输出 th:each，条件判断 th:if，插入代码块 th:insert，定义代码块 th:fragment，声明变量 th:object

三、th:each 的用法需要格外注意，打个比方：如果你要循环一个div中的p标签，则th:each属性必须放在p标签上。若你将th:each属性放在div上，则循环的是将整个div。

四、变量表达式中提供了很多的内置方法，该内置方法是用#开头，请不要与#{}消息表达式弄混。

五、th:insert，th:replace，th:include 三种插入代码块的效果相似，但区别很大。

```html
<!DOCTYPE html>
<!--名称空间-->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Thymeleaf 语法</title>
</head>
<body>
    <h2>apesource Thymeleaf 语法</h2>
    <!--th:text 设置当前元素的文本内容，常用，优先级不高-->
    <p th:text="${thText}" />
    <p th:utext="${thUText}" />

    <!--th:value 设置当前元素的value值，常用，优先级仅比th:text高-->
    <input type="text" th:value="${thValue}" />

    <!--th:each 遍历列表，常用，优先级很高，仅此于代码块的插入-->
    <!--th:each 修饰在div上，则div层重复出现，若只想p标签遍历，则修饰在p标签上-->
    <div th:each="message : ${thEach}"> <!-- 遍历整个div-p，不推荐-->
        <p th:text="${message}" />
    </div>
    <div> <!--只遍历p，推荐使用-->
        <p th:text="${message}" th:each="message : ${thEach}" />
    </div>

    <!--th:if 条件判断，类似的有th:switch，th:case，优先级仅次于th:each, 其中#strings是变量表达式的内置方法-->
    <p th:text="${thIf}" th:if="${not #strings.isEmpty(thIf)}"></p>

    <!--th:insert 把代码块插入当前div中，优先级最高，类似的有th:replace，th:include，~{} ：代码块表达式 -->
    <div th:insert="~{grammar/common::thCommon}"></div>

    <!--th:object 声明变量，和*{} 一起使用-->
    <div th:object="${thObject}">
        <p>ID: <span th:text="*{id}" /></p><!--th:text="${thObject.id}"-->
        <p>TH: <span th:text="*{thName}" /></p><!--${thObject.thName}-->
        <p>DE: <span th:text="*{desc}" /></p><!--${thObject.desc}-->
    </div>

</body>
</html>
```

后台给负责给变量赋值，和跳转页面

```java
@Controller
public class ThymeleafController {

    @RequestMapping("thymeleaf")
    public String thymeleaf(Map map) {
        map.put("thText", "th:text 设置文本内容 <b>加粗</b>");
        map.put("thUText", "th:utext 设置文本内容 <b>加粗</b>");
        map.put("thValue", "thValue 设置当前元素的value值");
        map.put("thEach", Arrays.asList("th:each", "遍历列表"));
        map.put("thIf", "msg is not null");
        map.put("thObject", new ThObject(1L, "th:object", "用来偷懒的th属性"));
        return "grammar/thymeleaf";
    }
}
```



## 4.5 SpringBoot热部署

默认情况下，在开发中我们修改一个项目文件后，想看到效果不得不重启应用，这会导致浪费大量时间 ，我们希望不重启应用的情况下，程序可以自动部署（热部署）

**如何能实现热部署？**

**1.** **关于模板引擎springBoot** 

在 Spring Boot 开发环境下禁用模板缓存

```properties
#开发环境下关闭thymeleaf模板缓存，thymeleaf默认是开启状态	
spring.thymeleaf.cache=false
```

**2. 添加 SpringBootDevtools 热部署依赖**

```xml
 <!--热部署-->	
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
</dependency>
```

**3.** **IntellijIEDA和Eclipse不同，IntellijIDEA必须做一些小调整**

在 Eclipse 中，修改文件后要手动进行保存，它就会自动编译，就触发热部署现象。在Intellij IEDA 中，修改文件后都是自动保存，默认不会自动编译文件，需要手动编译按 **ctrl+F9** 或者 **Build-->BuildProject**

![QQ截图20210107114200](QQ截图20210107114200.png)



## 4.6 关于springMVC的自动配置

如果想保留 SpringBoot MVC的特性，而且还想扩展新的功能（拦截器, 格式化器, 视图控制器等) 

可以自行创建配置类用@Configuration注解修饰并用@Bean配置

如果你想全面控制SpringMVC（也就是不使用默认配置功能）你在自定义的Web配置类上添加

@Configuration和@EnableWebMvc



## 4.7 注册Servlet三大组件 Servlet/Filter/Listener

而由于 Spring Boot 默认是以 jar 包的方式运行嵌入式Servlet容器来启动应用，没有web.xml文件，Spring提供以下Bean来注册三大组件

ServletRegistrationBean					注册自定义Servlet

FilterRegistrationBean						注册自定义Filter

ServletListenerRegistrationBean 	 注册自定义Listener

![1639551151993](1639551151993.png)



## 4.8 切换为其他嵌入式Servlet容器

SpringBoot 默认针对Servlet容器提供以下支持：

- Tomcat（默认使用）
- Jetty ：支持长连接项目（如：聊天页面）[ˈdʒeti]
- Undertow : 不支持 JSP , 但是并发性能高，是高性能非阻塞的容器[ˈʌndətəʊ] 

默认Tomcat容器

```xml
在spring-boot-starter-web启动器中默认引入了tomcat容器	
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <version>2.1.0.RELEASE</version>
    <scope>compile</scope>
</dependency>
```



切换 Jetty 容器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<!-- 排除tomcat容器 -->
	<exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
    <artifactId>spring-boot-starter-jetty</artifactId>
    <groupId>org.springframework.boot</groupId>
</dependency>
```

**使用外置Servlet容器Tomcat9.x**

**嵌入式Servlet容器**：运行启动类就可启动，或将项目打成可执行的 jar 包

​	优点：简单、快捷；

​	缺点：默认不支持JSP、优化定制比较复杂使用定制器, 还需要知道 每个功能 的底层原理

**外置Servlet容器**：配置 Tomcat, 将项目部署到Tomcat中运行





## 4.9 国际化

**SpringBoot国际化步骤:**

**1.**编写国际化配置文件，需要要显示的国际化内容写到配置中

​	注意:修改 properties 文件的字符编码，不然出现乱码

```properties
#类路径下创建 i18n	目录存放配置文件(i18n 是“国际化”的简称)	
login.properties		(默认国际化文件)
#login_语言代码_国家代码.propertis 
login_zh_CN.properties	(中文_中国 国际化文件) 
login_en_US.properties	(英文_美国 国际化文件)
```

**2.**类路径下创建i18n目录存放配置文件

![1639561120279](1639561120279.png)

```properties
spring.messages.basename = i18n.login
```

**3.**登录页面中通过获取国际化的值   #{ }

显示效果: 通过谷歌浏览器中设置-高级里切换语言查看效果