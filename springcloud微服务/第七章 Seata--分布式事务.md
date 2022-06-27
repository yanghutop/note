# **第七章** **Seata--分布式事务**

## **7.1** **分布式事务基础** 

### **7.1.1** **事务** 

​	事务指的就是一个操作单元，在这个操作单元中的所有操作最终要保持一致的行为，要么所有操作 

都成功，要么所有的操作都被撤销。简单地说，事务提供一种“要么什么都不做，要么做全套”机制。

### **7.1.2** **本地事物**

​	本地事物其实可以认为是数据库提供的事务机制。说到数据库事务就不得不说，数据库事务中的四 

大特性: 

- A：原子性(Atomicity)，一个事务中的所有操作，要么全部完成，要么全部不完成 

- C：一致性(Consistency)，在一个事务执行之前和执行之后数据库都必须处于一致性状态 

- I：隔离性(Isolation)，在并发环境中，当不同的事务同时操作相同的数据时，事务之间互不影响 

- D：持久性(Durability)，指的是只要事务成功结束，它对数据库所做的更新就必须永久的保存下来 

  数据库事务在实现时会将一次事务涉及的所有操作全部纳入到一个不可分割的执行单元，该执行单 

元中的所有操作要么都成功，要么都失败，只要其中任一操作执行失败，都将导致整个事务的回滚

### **7.1.3** **分布式事务**

​	分布式事务指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于不同的分布 

式系统的不同节点之上。 简单的说，就是一次大的操作由不同的小操作组成，这些小的操作分布在不同

的服务器上，且属于不同 的应用，分布式事务需要保证这些小操作要么全部成功，要么全部失败。 

**本质上来说，分布式事务就是为了保证不同数据库的数据一致性。**

### **7.1.4** **分布式事务的场景**

- 单体系统访问多个数据库

  一个服务需要调用多个数据库实例完成数据的增删改操作

![1626315812661](1626315812661.png)

- 多个微服务访问同一个数据库 

  多个服务需要调用一个数据库实例完成数据的增删改操作

![1626315830345](1626315830345.png)

- 多个微服务访问多个数据库 

  多个服务需要调用一个数据库实例完成数据的增删改操作

![1626315846422](1626315846422.png)



## **7.2** **分布式事务解决方案**

### **7.2.1** **全局事务**

全局事务基于DTP模型实现,DTP是由X/Open组织提出的一种分布式事务模型——X/Open 

Distributed Transaction Processing Reference Model。它规定了要实现分布式事务，需要三种角色： 

- AP: Application 应用系统 (微服务) 
- TM: Transaction Manager 事务管理器 (全局事务管理) 
- RM: Resource Manager 资源管理器 (数据库)

整个事务分成两个阶段:

- 阶段一: 表决阶段，所有参与者都将本事务执行预提交，并将能否成功的信息反馈发给协调者。

- 阶段二: 执行阶段，协调者根据所有参与者的反馈，通知所有参与者，步调一致地执行提交或者回 

  滚。

![1626315901814](1626315901814.png)

**优点** 

- 提高了数据一致性的概率，实现成本较低 

**缺点** 

- 单点问题: 事务协调者宕机 
- 同步阻塞: 延迟了提交时间，加长了资源阻塞时间 
- 数据不一致: 提交第二阶段，依然存在commit结果未知的情况，有可能导致数据不一致

### **7.2.2** **可靠消息服务**

​	

### **7.2.3** **最大努力通知**



### 7.2.4 TCC事务





## **7.3 Seata介绍**

​		**2019 年 1 月，阿里巴巴中间件团队发起了开源项目** **Fescar**（Fast & EaSy Commit And 

Rollback），**其愿景是让分布式事务的使用像本地事务的使用一样，简单和高效，并逐步解决开发者们** 

**遇到的分布式事务方面的所有难题。后来更名为** **Seata**，意为：Simple Extensible Autonomous 

Transaction Architecture，是一套分布式事务解决方案。 

​		**Seata的设计目标是对业务无侵入**，因此从业务无侵入的2PC方案着手，在传统2PC的基础上演进。 

**它把一个分布式事务理解成一个包含了若干分支事务的全局事务。全局事务的职责是协调其下管辖的分** 

**支事务达成一致，要么一起成功提交，要么一起失败回滚。此外，通常分支事务本身就是一个关系数据** 

**库的本地事务。**

![1626316212811](1626316212811.png)

**Seata主要由三个重要组件组成：**

- TC：Transaction Coordinator 事务协调器，管理全局的分支事务的状态，用于全局性事务的提交 

  和回滚。 

- TM：Transaction Manager 事务管理器，用于开启、提交或者回滚全局事务。 

- RM：Resource Manager 资源管理器，用于分支事务上的资源管理，向TC注册分支事务，上报分 

  支事务的状态，接受TC的命令来提交或者回滚分支事务。

![1626316240757](1626316240757.png)

**Seata的执行流程如下:**

1. A服务的TM向TC申请开启一个全局事务，TC就会创建一个全局事务并返回一个唯一的XID 

2. A服务的RM向TC注册分支事务，并及其纳入XID对应全局事务的管辖 

3. A服务执行分支事务，向数据库做操作4. A服务开始远程调用B服务，此时XID会在微服务的调用链上

   传播 

5. B服务的RM向TC注册分支事务，并将其纳入XID对应的全局事务的管辖 

6. B服务执行分支事务，向数据库做操作 

7. 全局事务调用链处理完毕，TM根据有无异常向TC发起全局事务的提交或者回滚 

8. TC协调其管辖之下的所有分支事务， 决定是否回滚



## **7.4 Seata实现分布式事务控制**

本示例通过Seata中间件实现分布式事务，模拟电商中的下单和扣库存的过程
我们通过订单微服务执行下单操作，然后由订单微服务调用商品微服务扣除库存

![1626316307629](1626316307629.png)



### **7.4.1** **案例基本代码**

#### **7.4.1.1** 修改订单服务与商品服务代码并完成下订单，减库存的操作

测试：下订单减库存

#### 7.4.1.1 在商品服务的减库存方法内，模拟异常

测试：下单订减库存，发现即使在减库存（reduceInventory）方法添加了@Transaction注解管理事务也没有效果

结果：发现订单生成了，但库存未减



### **7.4.2** 启动Seata

#### **7.4.2.1** **下载seata** 

下载地址：https://github.com/seata/seata/releases/v0.9.0/

#### **7.4.2.2** **修改配置文件** 

将下载得到的压缩包进行解压，进入conf目录，调整下面的配置文件：

- **registry.conf**

```json
registry { 
    type = "nacos" 
    nacos { 
    	serverAddr = "localhost" 
    	namespace = "public" 
    	cluster = "default" //集群
	} 
}

config { 
    type = "nacos" 
    nacos {
    	serverAddr = "localhost" 
    	namespace = "public" 
    	cluster = "default" 
	}
}
```

- **nacos-config.txt**

![1626420894954](1626420894954.png)

将上图的默认代码修改为如下配置

```properties
service.vgroup_mapping.service-product=default 
service.vgroup_mapping.service-order=default
```

这里的语法为： 

service.vgroup_mapping.${your-service-gruop}=default ，中间的 

${your-service-gruop} 为自己定义的服务组名称， 这里需要我们在程序的配置文件中配置。

#### **7.4.2.3** 初始化seata在nacos的配置

```yml
# 初始化seata 的nacos配置
# 注意: 这里要保证nacos是已经正常运行的 
cd conf 
nacos-config.sh 127.0.0.1
```

执行成功后可以打开Nacos的控制台，在配置列表中，可以看到初始化了很多Group为SEATA_GROUP 

的配置。

#### **7.4.2.4** 启动seata服务

```yml
cd bin
seata-server.bat -p 9000 -m file
```

启动后在 Nacos 的**服务列表**下面可以看到一个名为 serverAddr 的服务。9000是手动指定的交互端口，可以随意定义

### **7.4.3** **使用Seata实现事务控制** 

#### **7.4.3.1** **初始化数据表**

在我们的数据库中加入一张undo_log表,这是Seata记录事务日志要用到的表

```sql
CREATE TABLE `undo_log` ( 
    `id` BIGINT(20) NOT NULL AUTO_INCREMENT, 
    `branch_id` BIGINT(20) NOT NULL, 
    `xid` VARCHAR(100) NOT NULL, 
    `context` VARCHAR(128) NOT NULL,
    `rollback_info` LONGBLOB NOT NULL,
    `log_status` INT(11) NOT NULL, 
    `log_created` DATETIME NOT NULL, 
    `log_modified` DATETIME NOT NULL,
    `ext` VARCHAR(100) DEFAULT NULL, 
    PRIMARY KEY (`id`), 
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`) ) ENGINE = INNODB AUTO_INCREMENT = 1 DEFAULT CHARSET = utf8;
```

#### **7.4.3.2** **添加配置** 

在需要进行分布式控制的微服务中进行下面几项配置: 

**添加依赖**（订单与商品服务均要添加）

```xml
<dependency> 
    <groupId>com.alibaba.cloud</groupId> 
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId> 
</dependency> 
<dependency> 
    <groupId>com.alibaba.cloud</groupId> 
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId> 
</dependency>
<!--因为要读取配置，所以config也要添加-->
```

**DataSourceProxyConfig** （订单与商品服务均要添加）

Seata 是通过代理数据源实现事务分支的，所以需要配置 io.seata.rm.datasource.DataSourceProxy 的 

Bean，且是 @Primary默认的数据源，否则事务不会回滚，无法实现分布式事务

```java
@Configuration 
public class DataSourceProxyConfig {
    @Bean 
    @ConfigurationProperties(prefix = "spring.datasource") 
    public DruidDataSource druidDataSource() { 
        return new DruidDataSource(); 
    }
    @Primary
    @Bean 
    public DataSourceProxy dataSource(DruidDataSource druidDataSource) { 
        return new DataSourceProxy(druidDataSource); 
    } 
}
```

**registry.conf** 

在resources下添加Seata的配置文件 registry.conf

```yml
registry { 
	type = "nacos" 
	nacos {
    	serverAddr = "localhost" 
    	namespace = "public"
        cluster = "default" 
	}
}
config { 
	type = "nacos" 
	nacos { 
		serverAddr = "localhost"
        namespace = "public" 
        cluster = "default"
	} 
}
```

**bootstrap.yaml**		切记不要使用application.yaml

```yml
spring: 
	application: 
		name: service-product
    cloud: 
        nacos: 
        	config: 
        		server-addr: localhost:8848 # nacos的服务端地址 
        		namespace: public 					# 命名空间
        		group: SEATA_GROUP 					# 分组
        alibaba: 
        	seata: 
        		tx-service-group: service-product #与下方代码一致
        		
        		
service.vgroup_mapping.service-product=default 
service.vgroup_mapping.service-order=default

```

#### **7.4.3.3** 在order微服务开启全局事务

```java
@GlobalTransactional//全局事务控制 
public Order createOrder(Integer pid) {}
```

