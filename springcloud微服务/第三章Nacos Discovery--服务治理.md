# 第三章Nacos Discovery--服务治理

## **3.1** **服务治理介绍** 

**先来思考一个问题** 

通过上一章的操作，我们已经可以实现微服务之间的调用。但是我们把服务提供者的网络地址 

（ip，端口）等硬编码到了代码中，这种做法存在许多问题： 

- 一旦服务提供者地址变化，就需要手工修改代码 
- 一旦是多个服务提供者，无法实现负载均衡功能 
- 一旦服务变得越来越多，人工维护调用关系困难

那么应该怎么解决呢， 这时候就需要通过注册中心动态的实现**服务治理**。

**什么是服务治理** 

服务治理是微服务架构中最核心最基本的模块。用于实现各个微服务的**自动化注册与发现**。 

- **服务注册：**在服务治理框架中，都会构建一个注册中心，每个服务单元向注册中心登记自己提供服 

  务的详细信息。并在注册中心形成一张服务的清单，服务注册中心需要以心跳的方式去监测清单中 

  的服务是否可用，如果不可用，需要在服务清单中剔除不可用的服务。

- **服务发现：**服务调用方向服务注册中心咨询服务，并获取所有服务的实例清单，实现对具体服务实 

  例的访问。

![1625563475318](1625563475318.png)

通过上面的调用图会发现，除了微服务，还有一个组件是**服务注册中心**，它是微服务架构非常重要 

的一个组件，在微服务架构里主要起到了协调者的一个作用。注册中心一般包含如下几个功能： 

1. **服务发现：** 

- 服务注册：保存服务提供者和服务调用者的信息 
- 服务订阅：服务调用者订阅服务提供者的信息，注册中心向订阅者推送提供者的信息 

2. **服务配置：** 

- 配置订阅：服务提供者和服务调用者订阅微服务相关的配置 
- 配置下发：主动将配置推送给服务提供者和服务调用者 

3. **服务健康检测** 

- 检测服务提供者的健康情况，如果发现异常，执行服务剔除

**常见的注册中心** 

**Zookeeper** 

​	zookeeper是一个分布式服务框架，是Apache Hadoop 的一个子项目，它主要是用来解决分布式 

应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用 

配置项的管理等。 

**Eureka** 

​	Eureka是Springcloud Netflflix中的重要组件，主要作用就是做服务注册和发现。但是现在已经闭源 

**Consul**

​	Consul是基于GO语言开发的开源工具，主要面向分布式，服务化的系统提供服务注册、服务发现 

和配置管理的功能。Consul的功能都很实用，其中包括：服务注册/发现、健康检查、Key/Value 

存储、多数据中心和分布式一致性保证等特性。Consul本身只是一个二进制的可执行文件，所以 

安装和部署都非常简单，只需要从官网下载后，在执行对应的启动脚本即可。 

**Nacos** 

​	Nacos是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。它是 Spring 

Cloud Alibaba 组件之一，负责服务注册发现和服务配置，可以这样认为nacos=eureka+confifig。 



## 3.2 nacos简介

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速 

实现动态服务发现、服务配置、服务元数据及流量管理。 

从上面的介绍就可以看出，**nacos**的作用就是一个注册中心，用来管理注册上来的各个微服务

## **3.3 nacos实战入门**

接下来，我们就在现有的环境中加入nacos，并将我们的两个微服务注册上去。

### **3.3.1** **搭建nacos环境**

第1步: 安装nacos

```java
下载地址: https://github.com/alibaba/nacos/releases 
下载zip格式的安装包，然后进行解压缩操作
```

第2步: 启动nacos

```java
#切换目录 
cd nacos/bin 
#命令启动 
startup.cmd -m standalone

或者直接双击startup.cmd运行
```

第3步: 访问nacos

打开浏览器输入http://localhost:8848/nacos，即可访问服务， 默认密码是nacos/nacos

![1625564891158](1625564891158.png)



### **3.3.2** 将商品微服务注册到nacos

开始修改 shop-product 模块的代码， 将其注册到nacos服务上

1 在pom.xml中添加nacos的依赖

2 在主类上添加**@EnableDiscoveryClient**注解

3 在application.yml中添加nacos服务的地址

4 启动服务， 观察nacos的控制面板中是否有注册上来的商品微服务



### **3.3.3** **将订单微服务注册到**nacos

开始修改 shop_order 模块的代码， 将其注册到nacos服务上

1 在pom.xml中添加nacos的依赖

2 在主类上添加**@EnableDiscoveryClient**注解

3 在application.yml中添加nacos服务的地址

4 启动服务， 观察nacos的控制面板中是否有注册上来的订单微服务

5 修改OrderController， 实现微服务调用



实现步骤：

1.坐标

```xml
		<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        </dependency>
```

2.配置nacos服务的地址

```yml
spring:
    cloud:
        nacos:
          discovery:
            server-addr: 127.0.0.1:8848
```

3.启动类上加 @EnableDiscoveryClient注解

4.使用配置文件配置的服务名

```java
spring:
  application:
    name: service-product
    -----------------------------
ServiceInstance instance = discoveryClient.getInstances("service-product").get(0);
String url=instance.getHost()+":"+instance.getPort();

    Ribbon负载均衡后可以
String url="service-product";
Product product = restTemplate.getForObject("http://" + url + "/findproductbyid/" + pid, Product.class);
```



## **3.4** **实现服务调用的负载均衡** 

### **3.4.1** **什么是负载均衡** 

通俗的讲， 负载均衡就是将负载（工作任务，访问请求）进行分摊到多个操作单元（服务器,组件）上 

进行执行。 

根据负载均衡发生位置的不同,一般分为**服务端负载均衡**和**客户端负载均衡**。 

服务端负载均衡指的是发生在服务提供者一方,比如常见的nginx负载均衡 

而客户端负载均衡指的是发生在服务请求的一方，也就是在发送请求之前已经选好了由哪个实例处理请 

求。

![1625569917114](1625569917114.png)

我们在微服务调用关系中一般会选择客户端负载均衡，也就是在服务调用的一方来决定服务由哪个提供 

者执行。

### **3.4.2** **自定义实现负载均衡**

1 通过idea再启动一个 shop-product 微服务，设置其端口为8082

![1625657261189](1625657261189.png)

2 通过nacos查看微服务的启动情况

![1625657280987](1625657280987.png)

3 修改 shop-order 的代码，实现负载均衡



### **3.4.3** 基于Ribbon实现负载均衡

**Ribbon是Spring Cloud的一个组件， 它可以让我们使用一个注解就能轻松的搞定负载均衡** 

第1步：在RestTemplate 的生成方法上添加@LoadBalanced注解

第2步：修改服务调用的方法

```java
String url="service-product";
Product product = restTemplate.getForObject("http://" + url + "/findproductbyid/" + pid, Product.class);
```

**Ribbon支持的负载均衡策略** 

Ribbon内置了多种负载均衡策略,内部负载均衡的顶级接口为 

com.netflix.loadbalancer.IRule , 具体的负载策略如下图所示

| 策略名                    | 策略描述                                                     | 实现说明                                                     |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| BestAvailableRule         | 选择一个最小的并发 请求的server                              | 逐个考察Server，如果Server被tripped了，则忽略，在选择其中ActiveRequestsCount最小的server |
| AvailabilityFilteringRule | 过滤掉那些因为一直连接失败的被标记为circuit tripped的后端server，并过滤掉那些高并发的的后端server（activeconnections 超过配 置的阈值） | 使用一个AvailabilityPredicate来包含 过滤server的逻辑，其实就就是检查status里记录的各个server的运行状态 |
| WeightedResponseTimeRule  | 根据相应时间分配一 个weight，相应时间越长，weight越小，被选中的可能性越低 | 一个后台线程定期的从status里面读 取评价响应时间，为每个server计算一个weight。Weight的计算也比较简单responsetime 减去每个server自己平均的responsetime是server的权重。当刚开始运行，没有形成statas 时，使用roubine策略选择server |
| RetryRule                 | 对选定的负载均衡策 略机上重试机制。                          | 在一个配置时间段内当选择server不 成功，则一直尝试使用subRule的方式选择一个可用的server |
| RoundRobinRule            | 轮询方式轮询选择 server                                      | 轮询index，选择index对应位置的 server                        |
| RandomRule                | 随机选择一个server                                           | 在index上随机，选择index对应位置                             |

我们可以通过修改配置来调整Ribbon的负载均衡策略，具体代码如下

```yml
service-product: # 调用的提供者的名称 
		ribbon: 
			NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```



## **3.5** 基于Feign实现服务调用

### **3.5.1** 什么是Feign 

​			Feign是Spring Cloud提供的一个声明式的伪Http客户端， 它使得调用远程服务就像调用本地服务一样简单， 只需要创建一个接口并添加一个注解即可。 

​		Nacos很好的兼容了Feign， Feign默认集成了 Ribbon， 所以在Nacos下使用Fegin默认就实现了负 载均衡的效果

### 3.5.2 Feign的使用

1 加入Fegin的依赖

```xml
<!--fegin组件--> 
<dependency> 
    <groupId>org.springframework.cloud</groupId> 
    <artifactId>spring-cloud-starter-openfeign</artifactId> 
</dependency>
```

2 在主类上添加Fegin的注解          

```yml
@EnableDiscoveryClient 
@EnableFeignClients//开启Fegin
```

3 创建一个service， 并使用Fegin实现微服务调用

order 需要调用查询一个product    所以在order 中 创建一个接口   @FeignClient声明nacos中的提供者（如service-product）

在方法身上标志  @GetMapper("/product/{pind}")  查询product 的路径



```java
@FeignClient("service-product")//声明调用的提供者的name  并自动注入
public interface ProductService { 
    //指定调用提供者的哪个方法 
    //@FeignClient+@GetMapping 就是一个完整的请求路径 http://service- product/product/{pid} 
    @GetMapping(value = "/product/{pid}") 
    Product findByPid(@PathVariable("pid") Integer pid); 
}
```

4 修改controller代码，并启动验证

```java
	@Autowired
    ProductService productService;


    Product product = productService.findProductById(pid);
```

5 重启order微服务,查看效果