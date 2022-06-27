[TOC]

# 七.SpringBoot整合redis

SpringBoot 操作数据：spring-data-jpa 		spring-data-jdbc   		**spring-data-redis**

说明： 在 SpringBoot2.x 之后，原来使用的jedis 被替换为了 lettuce

**jedis :** 采用的直连，多个线程操作的话是不安全的，如果想要避免不安全，使用 jedis pool 连接池

**lettuce**  [ˈletɪs]: 采用netty，实例可以再多个线程中进行共享，不存在线程不安全的情况！可以减少线程数据了

1.加入Redis相关依赖

```xml
 	<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
	</dependency> 
```

2.application.properties中加入redis相关配置

```bash
# Redis数据库索引（默认为0）  
spring.redis.database=0  
# Redis服务器地址  
spring.redis.host=192.168.0.24  
# Redis服务器连接端口  
spring.redis.port=6379  
# Redis服务器连接密码（默认为空）  
spring.redis.password=  
# 连接池最大连接数（使用负值表示没有限制）  
spring.redis.pool.max-active=200  
# 连接池最大阻塞等待时间（使用负值表示没有限制）  
spring.redis.pool.max-wait=-1  
# 连接池中的最大空闲连接  
spring.redis.pool.max-idle=10 
# 连接池中的最小空闲连接  
spring.redis.pool.min-idle=0  
# 连接超时时间（毫秒）  
spring.redis.timeout=1000 
```

3.具体代码了解

```properties
spring data redis中封装了两个模板类,帮助我们实现redis的crud
	 * RedisTemplate			key value泛型都是object
	 * StringRedisTemplate		key value泛型都是string
注意:
	 * 1.两者数据各自存，各自取，数据不互通。
	 * 	RedisTemplate不能取StringRedisTemplate存入的数据
	 *	StringRedisTemplate不能取RedisTemplate存入的数据
	 *
	 * 2.序列化策略不同:
	 * 	RedisTemplate采用JDK的序列化策略（JdkSerializationRedisSerializer）保存的key和value都是      *  采用此策略序列化保存的
	 * 	存储时，先将数据序列化为字节数组，再存入Redis数据库。查看Redis会发现，是字节数组的形式类似乱码
	 * 	读取时，会将数据当做字节数组转化为我们需要的数据,以用来存储对象，但是要实现Serializable接口
	 *
	 * 	StringRedisTemplate采用String的序列化策略（StringRedisSerializer）保存的key和value都是      *  采用此策略序列化保存的当存入对象时，会报错:can not cast into String
	 * 	存储和读取，都为可读的数据
	 *
	 * 3.两者的关系是StringRedisTemplate继承RedisTemplate
	 *
	 * 4.使用场景:
	 *  当你的redis数据库里面本来存的是字符串数据或者你要存取的数据就是字符串类型数据的时候，那么你就使用	    *  StringRedisTemplate即可。
	 *  但是如果你的数据是复杂的对象类型，而取出的时候又不想做任何的数据转换，直接从Redis里面取出一个对	   *  象，那么使用RedisTemplate是更好的选择。
	 
五大数据类型
	 *         redisTemplate.opsForValue();//操作字符串
	 *         redisTemplate.opsForList();//操作List
	 *         redisTemplate.opsForSet();//操作Set
	 *         redisTemplate.opsForZSet();//操作ZSet
	 *         redisTemplate.opsForHash();//操作Hash
```

如果保存对象使用redisTemplate，但是数据库里可读性太差，所以自定义redisTemplate

查看源码RedisAutoConfiguration类中（双击shift全文检索）

```JAVA
@Bean
@ConditionalOnMissingBean(name = "redisTemplate") 
//看到这个@ConditionalOnMissingBean注解后，代表此类如果容器中无则注入，如有则被覆盖
public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory
redisConnectionFactory)
throws UnknownHostException {
        // 默认的 RedisTemplate 没有过多的设置，redis 对象都是需要序列化！
        // 两个泛型都是 Object, Object 的类型，我们后使用需要强制转换 <String, Object>
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
}

@Bean
@ConditionalOnMissingBean // 由于 String 是redis中最常使用的类型，所以说单独提出来了一
个bean！
public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory
redisConnectionFactory)
throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
}
```

```bash
改变序列化策略
默认序列化方式存储到redis的数据人工不可读
不同策略序列化的过程有性能高低的
spring-data-redis提供如下几种序列化策略
GenericToStringSerializer: 可以将任何对象泛化为字符串并序列化
Jackson2JsonRedisSerializer: 跟JacksonJsonRedisSerializer实际上是一样的
JacksonJsonRedisSerializer: 序列化object对象为json字符串
JdkSerializationRedisSerializer: 序列化java对象
StringRedisSerializer: 简单的字符串序列化
```



# 八.redis的配置

```bash
#是否在后台运行；no：不是后台运行
daemonize yes
 
#是否开启保护模式，默认开启。要是配置里没有指定bind和密码。开启该参数后，redis只会本地进行访问，拒绝外部访问。
protected-mode yes
 
#redis的进程文件
pidfile /var/run/redis/redis-server.pid
 
#redis监听的端口号。
port 6379
 
#此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。该内核参数默认值一般是128，对于负载很大的服务程序来说大大的不够。一般会将它修改为2048或者更大。在/etc/sysctl.conf中添加:net.core.somaxconn = 2048，然后在终端中执行sysctl -p。
tcp-backlog 511
 
#指定 redis 只接收来自于该 IP 地址的请求，如果不进行设置，那么将处理所有请求
#bind 127.0.0.1
#bind 0.0.0.0
 
#配置unix socket来让redis支持监听本地连接。
# unixsocket /var/run/redis/redis.sock
 
#配置unix socket使用文件的权限
# unixsocketperm 700
 
# 此参数为设置客户端空闲超过timeout，服务端会断开连接，为0则服务端不会主动断开连接，不能小于0。
timeout 0
 
#tcp keepalive参数。如果设置不为0，就使用配置tcp的SO_KEEPALIVE值，使用keepalive有两个好处:检测挂掉的对端。降低中间设备出问题而导致网络看似连接却已经与对端端口的问题。在Linux内核中，设置了keepalive，redis会定时给对端发送ack。检测到对端关闭需要两倍的设置值。
tcp-keepalive 0
 
#指定了服务端日志的级别。级别包括：debug（很多信息，方便开发、测试），verbose（许多有用的信息，但是没有debug级别信息多），notice（适当的日志级别，适合生产环境），warn（只有非常重要的信息）
loglevel notice
 
#指定了记录日志的文件。空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null。
logfile /var/log/redis/redis-server.log
 
#是否打开记录syslog功能
# syslog-enabled no
 
#syslog的标识符。
# syslog-ident redis
 
#日志的来源、设备
# syslog-facility local0
 
#数据库的数量，默认使用的数据库是DB 0。可以通过SELECT命令选择一个db
databases 16
 
# redis是基于内存的数据库，可以通过设置该值定期写入磁盘。
# 注释掉“save”这一行配置项就可以让保存数据库功能失效
# 900秒（15分钟）内至少1个key值改变（则进行数据库保存--持久化） 
# 300秒（5分钟）内至少10个key值改变（则进行数据库保存--持久化） 
# 60秒（1分钟）内至少10000个key值改变（则进行数据库保存--持久化）
save 900 1
save 300 10
save 60 10000
 
#当RDB持久化出现错误后，是否依然进行继续进行工作，yes：不能进行工作，no：可以继续进行工作，可以通过info中的rdb_last_bgsave_status了解RDB持久化是否有错误
stop-writes-on-bgsave-error yes
 
#使用压缩rdb文件，rdb文件压缩使用LZF压缩算法，yes：压缩，但是需要一些cpu的消耗。no：不压缩，需要更多的磁盘空间
rdbcompression yes
 
#是否校验rdb文件。从rdb格式的第五个版本开始，在rdb文件的末尾会带上CRC64的校验和。这跟有利于文件的容错性，但是在保存rdb文件的时候，会有大概10%的性能损耗，所以如果你追求高性能，可以关闭该配置。
rdbchecksum yes
 
#rdb文件的名称
dbfilename dump.rdb
 
#数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录
dir /data
 
 
############### 主从复制 ###############
 
#复制选项，slave复制对应的master。
# slaveof <masterip> <masterport>
 
#如果master设置了requirepass，那么slave要连上master，需要有master的密码才行。masterauth就是用来配置master的密码，这样可以在连上master后进行认证。
# masterauth <master-password>
 
#当从库同主机失去连接或者复制正在进行，从机库有两种运行方式：1) 如果slave-serve-stale-data设置为yes(默认设置)，从库会继续响应客户端的请求。2) 如果slave-serve-stale-data设置为no，除去INFO和SLAVOF命令之外的任何请求都会返回一个错误”SYNC with master in progress”。
slave-serve-stale-data yes
 
#作为从服务器，默认情况下是只读的（yes），可以修改成NO，用于写（不建议）。
slave-read-only yes
 
#是否使用socket方式复制数据。目前redis复制提供两种方式，disk和socket。如果新的slave连上来或者重连的slave无法部分同步，就会执行全量同步，master会生成rdb文件。有2种方式：disk方式是master创建一个新的进程把rdb文件保存到磁盘，再把磁盘上的rdb文件传递给slave。socket是master创建一个新的进程，直接把rdb文件以socket的方式发给slave。disk方式的时候，当一个rdb保存的过程中，多个slave都能共享这个rdb文件。socket的方式就的一个个slave顺序复制。在磁盘速度缓慢，网速快的情况下推荐用socket方式。
repl-diskless-sync no
 
#diskless复制的延迟时间，防止设置为0。一旦复制开始，节点不会再接收新slave的复制请求直到下一个rdb传输。所以最好等待一段时间，等更多的slave连上来。
repl-diskless-sync-delay 5
 
#slave根据指定的时间间隔向服务器发送ping请求。时间间隔可以通过 repl_ping_slave_period 来设置，默认10秒。
# repl-ping-slave-period 10
 
#复制连接超时时间。master和slave都有超时时间的设置。master检测到slave上次发送的时间超过repl-timeout，即认为slave离线，清除该slave信息。slave检测到上次和master交互的时间超过repl-timeout，则认为master离线。需要注意的是repl-timeout需要设置一个比repl-ping-slave-period更大的值，不然会经常检测到超时。
# repl-timeout 60
 
#是否禁止复制tcp链接的tcp nodelay参数，可传递yes或者no。默认是no，即使用tcp nodelay。如果master设置了yes来禁止tcp nodelay设置，在把数据复制给slave的时候，会减少包的数量和更小的网络带宽。但是这也可能带来数据的延迟。默认我们推荐更小的延迟，但是在数据量传输很大的场景下，建议选择yes。
repl-disable-tcp-nodelay no
 
#复制缓冲区大小，这是一个环形复制缓冲区，用来保存最新复制的命令。这样在slave离线的时候，不需要完全复制master的数据，如果可以执行部分同步，只需要把缓冲区的部分数据复制给slave，就能恢复正常复制状态。缓冲区的大小越大，slave离线的时间可以更长，复制缓冲区只有在有slave连接的时候才分配内存。没有slave的一段时间，内存会被释放出来，默认1m。
# repl-backlog-size 5mb
 
#master没有slave一段时间会释放复制缓冲区的内存，repl-backlog-ttl用来设置该时间长度。单位为秒。
# repl-backlog-ttl 3600
 
#当master不可用，Sentinel会根据slave的优先级选举一个master。最低的优先级的slave，当选master。而配置成0，永远不会被选举。
slave-priority 100
 
#redis提供了可以让master停止写入的方式，如果配置了min-slaves-to-write，健康的slave的个数小于N，mater就禁止写入。master最少得有多少个健康的slave存活才能执行写命令。这个配置虽然不能保证N个slave都一定能接收到master的写操作，但是能避免没有足够健康的slave的时候，master不能写入来避免数据丢失。设置为0是关闭该功能。
# min-slaves-to-write 3
 
#延迟小于min-slaves-max-lag秒的slave才认为是健康的slave。
# min-slaves-max-lag 10
 
# 设置1或另一个设置为0禁用这个特性。
# Setting one or the other to 0 disables the feature.
# By default min-slaves-to-write is set to 0 (feature disabled) and
# min-slaves-max-lag is set to 10.
 
 
############### 安全相关 ###############
 
#requirepass配置可以让用户使用AUTH命令来认证密码，才能使用其他命令。这让redis可以使用在不受信任的网络中。为了保持向后的兼容性，可以注释该命令，因为大部分用户也不需要认证。使用requirepass的时候需要注意，因为redis太快了，每秒可以认证15w次密码，简单的密码很容易被攻破，所以最好使用一个更复杂的密码。注意只有密码没有用户名。
# requirepass foobared
 
#把危险的命令给修改成其他名称。比如CONFIG命令可以重命名为一个很难被猜到的命令，这样用户不能使用，而内部工具还能接着使用。
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
 
#设置成一个空的值，可以禁止一个命令
# rename-command CONFIG ""
 
 
############### 进程限制相关 ###############
 
# 设置能连上redis的最大客户端连接数量。默认是10000个客户端连接。由于redis不区分连接是客户端连接还是内部打开文件或者和slave连接等，所以maxclients最小建议设置到32。如果超过了maxclients，redis会给新的连接发送’max number of clients reached’，并关闭连接。
# maxclients 10000
 
#redis配置的最大内存容量。当内存满了，需要配合maxmemory-policy策略进行处理。注意slave的输出缓冲区是不计算在maxmemory内的。所以为了防止主机内存使用完，建议设置的maxmemory需要更小一些。
# maxmemory <bytes>
 
#内存容量超过maxmemory后的处理策略。
#volatile-lru：利用LRU算法移除设置过过期时间的key。
#volatile-random：随机移除设置过过期时间的key。
#volatile-ttl：移除即将过期的key，根据最近过期时间来删除（辅以TTL）
#allkeys-lru：利用LRU算法移除任何key。
#allkeys-random：随机移除任何key。
#noeviction：不移除任何key，只是返回一个写错误。
#上面的这些驱逐策略，如果redis没有合适的key驱逐，对于写命令，还是会返回错误。redis将不再接收写请求，只接收get请求。写命令包括：set setnx setex append incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby getset mset msetnx exec sort。
# maxmemory-policy noeviction
 
#lru检测的样本数。使用lru或者ttl淘汰算法，从需要淘汰的列表中随机选择sample个key，选出闲置时间最长的key移除。
# maxmemory-samples 5
 
 
############### APPEND ONLY 持久化方式 ###############
 
#默认redis使用的是rdb方式持久化，这种方式在许多应用中已经足够用了。但是redis如果中途宕机，会导致可能有几分钟的数据丢失，根据save来策略进行持久化，Append Only File是另一种持久化方式，可以提供更好的持久化特性。Redis会把每次写入的数据在接收后都写入 appendonly.aof 文件，每次启动时Redis都会先把这个文件的数据读入内存里，先忽略RDB文件。
appendonly no
 
#aof文件名
appendfilename "appendonly.aof"
 
#aof持久化策略的配置
#no表示不执行fsync，由操作系统保证数据同步到磁盘，速度最快。
#always表示每次写入都执行fsync，以保证数据同步到磁盘。
#everysec表示每秒执行一次fsync，可能会导致丢失这1s数据。
appendfsync everysec
 
# 在aof重写或者写入rdb文件的时候，会执行大量IO，此时对于everysec和always的aof模式来说，执行fsync会造成阻塞过长时间，no-appendfsync-on-rewrite字段设置为默认设置为no，是最安全的方式，不会丢失数据，但是要忍受阻塞的问题。如果对延迟要求很高的应用，这个字段可以设置为yes，，设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,不会造成阻塞的问题（因为没有磁盘竞争），等rewrite完成后再写入，这个时候redis会丢失数据。Linux的默认fsync策略是30秒。可能丢失30秒数据。因此，如果应用系统无法忍受延迟，而可以容忍少量的数据丢失，则设置为yes。如果应用系统无法忍受数据丢失，则设置为no。
no-appendfsync-on-rewrite no
 
#aof自动重写配置。当目前aof文件大小超过上一次重写的aof文件大小的百分之多少进行重写，即当aof文件增长到一定大小的时候Redis能够调用bgrewriteaof对日志文件进行重写。当前AOF文件大小是上次日志重写得到AOF文件大小的二倍（设置为100）时，自动启动新的日志重写过程。
auto-aof-rewrite-percentage 100
#设置允许重写的最小aof文件大小，避免了达到约定百分比但尺寸仍然很小的情况还要重写
auto-aof-rewrite-min-size 64mb
 
#aof文件可能在尾部是不完整的，当redis启动的时候，aof文件的数据被载入内存。重启可能发生在redis所在的主机操作系统宕机后，尤其在ext4文件系统没有加上data=ordered选项（redis宕机或者异常终止不会造成尾部不完整现象。）出现这种现象，可以选择让redis退出，或者导入尽可能多的数据。如果选择的是yes，当截断的aof文件被导入的时候，会自动发布一个log给客户端然后load。如果是no，用户必须手动redis-check-aof修复AOF文件才可以。
aof-load-truncated yes
 
 
############### LUA SCRIPTING ###############
 
# 如果达到最大时间限制（毫秒），redis会记个log，然后返回error。当一个脚本超过了最大时限。只有SCRIPT KILL和SHUTDOWN NOSAVE可以用。第一个可以杀没有调write命令的东西。要是已经调用了write，只能用第二个命令杀。
lua-time-limit 5000
 
 
############### 集群相关 ###############
 
#集群开关，默认是不开启集群模式。
# cluster-enabled yes
 
#集群配置文件的名称，每个节点都有一个集群相关的配置文件，持久化保存集群的信息。这个文件并不需要手动配置，这个配置文件有Redis生成并更新，每个Redis集群节点需要一个单独的配置文件，请确保与实例运行的系统中配置文件名称不冲突
# cluster-config-file nodes-6379.conf
 
#节点互连超时的阀值。集群节点超时毫秒数
# cluster-node-timeout 15000
 
#在进行故障转移的时候，全部slave都会请求申请为master，但是有些slave可能与master断开连接一段时间了，导致数据过于陈旧，这样的slave不应该被提升为master。该参数就是用来判断slave节点与master断线的时间是否过长。判断方法是：
#比较slave断开连接的时间和(node-timeout * slave-validity-factor) + repl-ping-slave-period
#如果节点超时时间为三十秒, 并且slave-validity-factor为10,假设默认的repl-ping-slave-period是10秒，即如果超过310秒slave将不会尝试进行故障转移 
# cluster-slave-validity-factor 10
 
#master的slave数量大于该值，slave才能迁移到其他孤立master上，如这个参数若被设为2，那么只有当一个主节点拥有2 个可工作的从节点时，它的一个从节点会尝试迁移。
# cluster-migration-barrier 1
 
#默认情况下，集群全部的slot有节点负责，集群状态才为ok，才能提供服务。设置为no，可以在slot没有全部分配的时候提供服务。不建议打开该配置。
# cluster-require-full-coverage yes
 
 
############### SLOW LOG 慢查询日志 ###############
 
###slog log是用来记录redis运行中执行比较慢的命令耗时。当命令的执行超过了指定时间，就记录在slow log中，slog log保存在内存中，所以没有IO操作。
#执行时间比slowlog-log-slower-than大的请求记录到slowlog里面，单位是微秒，所以1000000就是1秒。注意，负数时间会禁用慢查询日志，而0则会强制记录所有命令。
slowlog-log-slower-than 10000
 
#慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录会被删掉。这个长度没有限制。只要有足够的内存就行。你可以通过 SLOWLOG RESET 来释放内存。
slowlog-max-len 128
 
############### 延迟监控 ###############
#延迟监控功能是用来监控redis中执行比较缓慢的一些操作，用LATENCY打印redis实例在跑命令时的耗时图表。只记录大于等于下边设置的值的操作。0的话，就是关闭监视。默认延迟监控功能是关闭的，如果你需要打开，也可以通过CONFIG SET命令动态设置。
latency-monitor-threshold 0
 
############### EVENT NOTIFICATION 订阅通知 ###############
#键空间通知使得客户端可以通过订阅频道或模式，来接收那些以某种方式改动了 Redis 数据集的事件。因为开启键空间通知功能需要消耗一些 CPU ，所以在默认配置下，该功能处于关闭状态。
#notify-keyspace-events 的参数可以是以下字符的任意组合，它指定了服务器该发送哪些类型的通知：
##K 键空间通知，所有通知以 __keyspace@__ 为前缀
##E 键事件通知，所有通知以 __keyevent@__ 为前缀
##g DEL 、 EXPIRE 、 RENAME 等类型无关的通用命令的通知
##$ 字符串命令的通知
##l 列表命令的通知
##s 集合命令的通知
##h 哈希命令的通知
##z 有序集合命令的通知
##x 过期事件：每当有过期键被删除时发送
##e 驱逐(evict)事件：每当有键因为 maxmemory 政策而被删除时发送
##A 参数 g$lshzxe 的别名
#输入的参数中至少要有一个 K 或者 E，否则的话，不管其余的参数是什么，都不会有任何 通知被分发。详细使用可以参考http://redis.io/topics/notifications
 
notify-keyspace-events ""
 
############### ADVANCED CONFIG 高级配置 ###############
#数据量小于等于hash-max-ziplist-entries的用ziplist，大于hash-max-ziplist-entries用hash
hash-max-ziplist-entries 512
#value大小小于等于hash-max-ziplist-value的用ziplist，大于hash-max-ziplist-value用hash。
hash-max-ziplist-value 64
 
#数据量小于等于list-max-ziplist-entries用ziplist，大于list-max-ziplist-entries用list。
list-max-ziplist-entries 512
#value大小小于等于list-max-ziplist-value的用ziplist，大于list-max-ziplist-value用list。
list-max-ziplist-value 64
 
#数据量小于等于set-max-intset-entries用iniset，大于set-max-intset-entries用set。
set-max-intset-entries 512
 
#数据量小于等于zset-max-ziplist-entries用ziplist，大于zset-max-ziplist-entries用zset。
zset-max-ziplist-entries 128
#value大小小于等于zset-max-ziplist-value用ziplist，大于zset-max-ziplist-value用zset。
zset-max-ziplist-value 64
 
#value大小小于等于hll-sparse-max-bytes使用稀疏数据结构（sparse），大于hll-sparse-max-bytes使用稠密的数据结构（dense）。一个比16000大的value是几乎没用的，建议的value大概为3000。如果对CPU要求不高，对空间要求较高的，建议设置到10000左右。
hll-sparse-max-bytes 3000
 
#Redis将在每100毫秒时使用1毫秒的CPU时间来对redis的hash表进行重新hash，可以降低内存的使用。当你的使用场景中，有非常严格的实时性需要，不能够接受Redis时不时的对请求有2毫秒的延迟的话，把这项配置为no。如果没有这么严格的实时性要求，可以设置为yes，以便能够尽可能快的释放内存。
activerehashing yes
 
##对客户端输出缓冲进行限制可以强迫那些不从服务器读取数据的客户端断开连接，用来强制关闭传输缓慢的客户端。
#对于normal client，第一个0表示取消hard limit，第二个0和第三个0表示取消soft limit，normal client默认取消限制，因为如果没有寻问，他们是不会接收数据的。
client-output-buffer-limit normal 0 0 0
#对于slave client和MONITER client，如果client-output-buffer一旦超过256mb，又或者超过64mb持续60秒，那么服务器就会立即断开客户端连接。
client-output-buffer-limit slave 256mb 64mb 60
#对于pubsub client，如果client-output-buffer一旦超过32mb，又或者超过8mb持续60秒，那么服务器就会立即断开客户端连接。
client-output-buffer-limit pubsub 32mb 8mb 60
 
#redis执行任务的频率为1s除以hz。
hz 10
 
#在aof重写的时候，如果打开了aof-rewrite-incremental-fsync开关，系统会每32MB执行一次fsync。这对于把文件写入磁盘是有帮助的，可以避免过大的延迟峰值。
aof-rewrite-incremental-fsync yes
```



# 九.redis持久化

Redis 是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了持久化功能！

**持久化过程保存什么**

1.将当前数据状态进行保存，快照形式，存储数据结果，存储格式简单，关注点在数据  

2.将数据的操作过程进行保存，日志形式，存储操作过程，存储格式复杂，关注点在数据的操作过程



10011001110000001 															   删除第3行 

00101001011010110																第4行末位添加字符x 

10110011001110000 															   删除第2到第4行 

00100101001011011																复制第3行粘贴到第5行

 	**数据（快照）**																				**过程** **（日志）** 

​	 **RDB**																									**AOF**



### **RDB启动方式 （手动）**

**save指令**

命令 :**save**

作用 :手动执行一次保存操作



**save指令相关配置**

**dbfilename dump.rdb** 

说明：设置本地数据库文件名，默认值为 dump.rdb 

经验：通常设置为      dump-端口号.rdb



**dir** 

说明：设置存储.rdb文件的路径 

经验：通常设置成存储空间较大的目录中，目录名称data



**rdbcompression yes** 

说明：设置存储至本地数据库时是否压缩数据，默认为 yes，采用 LZF 压缩 

经验：通常默认为开启状态，如果设置为no，可以节省 CPU 运行时间，但会使存储的文件变大（巨大） 



**rdbchecksum yes** 

说明：设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程均进行 

经验：通常默认为开启状态，如果设置为no，可以节约读写性过程约10%时间消耗，但是存储一定的数据损坏风险



**save指令工作原理**(**单线程任务执行序列**)

**客户端1** 127.0.0.1:6379>set key1 value1

**客户端2** 127.0.0.1:6379>set key2 value2

**客户端3** 127.0.0.1:6379>save

**客户端4** 127.0.0.1:6379>get key

对redis数据库执行4此指令顺序===> set  set  save  get

**注意：**save指令的执行会阻塞当前Redis服务器，直到当前RDB过程完成为止，有可能会造成长时间阻塞，线上环境不建议使用



**bgsave指令**

命令 ：bgsave

作用 ：手动启动后台保存操作，但不是立即执行



**bgsave指令工作原理**

![1618044283425](1618044283425.png)

注意： bgsave命令是针对save阻塞问题做的优化。Redis内部所有涉及到RDB操作都采用bgsave的方式，save命令可以放弃使用。



### **RDB启动方式(自动)**

**save配置**

配置 ：save *second changes*

作用 :满足限定时间范围内key的变化数量达到指定数量即进行持久化 

参数 :

second：监控时间范围 

changes：监控key的变化量 

位置 :在conf文件中进行配置 

范例 :

save *900 1* 

save *300 10* 

save *60 10000*

注意： 

save配置要根据实际业务情况进行设置，频度过高或过低都会出现性能问题，结果可能是灾难性的 

 save配置中对于second与changes设置通常具有互补对应关系，尽量不要设置成包含性关系 

 save配置启动后执行的是bgsave操作



**RDB启动方式对比**

![1618044840003](1618044840003.png)



**RDB优点**

RDB是一个紧凑压缩的二进制文件，存储效率较高 

RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景  

RDB恢复数据的速度要比AOF快很多  

应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。 

**Rdb缺点**

RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据 

bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能 

Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象



**RDB存储的弊端**

存储数据量较大，效率较低 

基于快照思想，每次读写都是全部数据，当数据量巨大时，效率非常低

基于fork创建子进程，内存产生额外消耗

宕机带来的数据丢失风险

**解决思路**

不写全数据，仅记录部分数据

对所有操作均进行记录，排除丢失数据的风险



### **AOF概念**

AOF(append only file)持久化：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令 达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式



**AOF写数据过程**

![1618045054162](1618045054162.png)

**AOF写数据三种策略(appendfsync)**

always(每次） 

每次写入操作均同步到AOF文件中，数据零误差，性能较低 



everysec（每秒） 

每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高 

在系统突然宕机的情况下丢失1秒内的数据 



no（系统控制） 

由操作系统控制每次同步到AOF文件的周期，整体过程不可控



**AOF功能开启**

配置 ：**appendonly yes|no**

作用 ：是否开启AOF持久化功能，默认为不开启状态 

配置 ：**appendfsync always|everysec|no**

作用 ：AOF写数据策略 



**AOF相关配置**

配置：appendfilename  filename

作用：AOF持久化文件名，默认文件名未appendonly.aof，建议配置为appendonly-端口号.aof

配置：dir

作用 ：AOF持久化文件保存路径，与RDB持久化文件保持一致即可



**AOF写数据遇到的问题**

![1618186844239](1618186844239.png)

**AOF重写**

随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入了AOF重写机制压缩文件体积。AOF文件重写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程。简单说就是将对同一个数据的若干个条命令执行结果转化成最终结果数据对应的指令进行记录

**AOF重写作用**

降低磁盘占用量，提高磁盘利用率 

提高持久化效率，降低持久化写时间，提高IO性能  

降低数据恢复用时，提高数据恢复效率

**AOF重写规则**

进程内已超时的数据不再写入文件 

忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令 

如del key1、 hdel key2、srem key3、set key4 111、set key4 222等 

对同一数据的多条写命令合并为一条命令 

如lpush list1 a、lpush list1 b、 lpush list1 c 可以转化为：lpush list1 a b c。 

为防止数据量过大造成客户端缓冲区溢出，对list、set、hash、zset等类型，每条指令最多写入64个元素

**AOF重写方式**

手动重写	**bgrewriteaof**

自动重写	

![1618046735234](1618046735234.png)

# 十.**Redis 删除策略**

## 1.**过期数据**

**Redis中的数据特征**

Redis是一种内存级数据库，所有数据均存放在内存中，内存中的数据可以通过TTL指令获取其状态

- XX ：具有时效性的数据
- -1 ：永久有效的数据
- \-2 ：已经过期的数据或被删除的数据或未定义的数据

**过期的数据真的删除了吗？**

![1632907205648](1632907205648.png)

![1632907282373](1632907282373.png)



## 2.**数据删除策略**

1. 定时删除 

2. 惰性删除 

3. 定期删除

**redis数据保存策略**

![1](1.png)

**数据删除策略的目标** 

在内存占用与CPU占用之间寻找一种平衡，顾此失彼都会造成整体redis性能的下降，甚至引发服务器宕

机或内存泄露



### 2.1**定时删除**

- 创建一个定时器，当key设置有过期时间，且过期时间到达时，由定时器任务立即执行对键的删除操作 
- 优点：节约内存，到时就删除，快速释放掉不必要的内存占用 
- 缺点：CPU压力很大，无论CPU此时负载量多高，均占用CPU，会影响redis服务器响应时间和指令吞吐量 
- 总结：用处理器性能换取存储空间（拿时间换空间）

![2](2.png)



### 2.2 **惰性删除**

- 数据到达过期时间，不做处理。等下次访问该数据时 
  - 如果未过期，返回数据 
  - 发现已过期，删除，返回不存在
- 优点：节约CPU性能，发现必须删除的时候才删除 
- 缺点：内存压力很大，出现长期占用内存的数据
- 总结：用存储空间换取处理器性能（拿时间换空间）



### 2.3 **定期删除**

**两种方案都走极端，有没有折中方案?**

- Redis启动服务器初始化时，读取配置server.hz的值，默认为10
- 每秒钟执行server.hz次**serverCron()**中的方法

![1632908135503](1632908135503.png)



**定期删除**:周期性轮询redis库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度

- 优点1：CPU性能占用设置有峰值，检测频度可自定义设置 
- 优点2：内存压力不是很大，长期占用内存的冷数据会被持续清理
- 总结：周期性抽查存储空间 （随机抽查，重点抽查） 



### 2.4 **删除策略比对**

1. 定时删除 	节约内存，无占用 	不分时段占用CPU资源，频度高    拿时间换空间
2. 惰性删除 	内存占用严重	        延时执行，CPU利用率高                拿空间换时间 

3. 定期删除     内存定期随机清理    每秒花费固定的CPU资源维护内存  随机抽查，重点抽查



##  3.**逐出算法**

**当新数据进入redis时，如果内存不足怎么办？**

- Redis使用内存存储数据，在执行每一个命令前，会调用**freeMemoryIfNeeded()**检测内存是否充足。如果内存不满足新加入数据的最低存储要求，redis要临时删除一些数据为当前指令清理存储空间。清理数据的策略称为**逐出算法**。 
- 注意：逐出数据的过程不是100%能够清理出足够的可使用的内存空间，如果不成功则反复执行。当对所有数据尝试完毕后，如果不能达到内存清理的要求，将出现错误信息。





# 十一.**企业级解决方案**

## **缓存预热**

**“宕机”**服务器启动后迅速宕机

**问题排查**

1. 请求数量较高 
2. 主从之间数据吞吐量较大，数据同步操作频度较高,因为刚刚启动时，缓存中没有任何数据

**解决方案**

前置准备工作： 

1. 日常例行统计数据访问记录，统计访问频度较高的热点数据 

准备工作： 

1. 将统计结果中的数据分类，根据级别，redis优先加载级别较高的热点数据 

2. 利用分布式多服务器同时进行数据读取，提速数据加载过程 

3. 热点数据主从同时预热

实施： 

1. 使用脚本程序固定触发数据预热过程 

2. 如果条件允许，使用了CDN（内容分发网络），效果会更好

> [CDN](https://baike.baidu.com/item/CDN)的全称是Content Delivery Network，即内容分发网络。其基本思路是尽可能避开[互联网](https://baike.baidu.com/item/互联网/199186)上有可能影响数据传输速度和稳定性的[瓶颈](https://baike.baidu.com/item/瓶颈/1503615)和环节，使内容传输得更快、更稳定。通过在网络各处放置[节点服务器](https://baike.baidu.com/item/节点服务器/4576219)所构成的在现有的互联网基础之上的一层智能[虚拟网络](https://baike.baidu.com/item/虚拟网络/855117)，CDN系统能够实时地根据[网络流量](https://baike.baidu.com/item/网络流量/7489548)和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务节点上。其目的是使用户可就近取得所需内容，解决 Internet[网络拥挤](https://baike.baidu.com/item/网络拥挤/10004398)的状况，提高用户访问网站的响应速度

**总结**

缓存预热就是系统启动前，提前将相关的缓存数据直接加载到缓存系统。避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据



## **缓存雪崩**

**数据库服务器崩溃（1）** 

1. 系统平稳运行过程中，忽然数据库连接量激增 

2. 应用服务器无法及时处理请求 

3. 大量408超时，500错误页面出现 

4. 客户反复刷新页面获取数据 

5. 数据库崩溃 

6. 应用服务器崩溃 

7. 重启应用服务器无效 

8. Redis服务器崩溃 

9. Redis集群崩溃 

10. 重启数据库后再次被瞬间流量放倒

**问题排查**

1. 在一个**较短**的时间内，缓存中**较多**的key**集中过期** 

2. 此周期内请求访问过期的数据，redis未命中，redis向数据库获取数据 

3. 数据库同时接收到大量的请求无法及时处理 

4. Redis大量请求被积压，开始出现超时现象 

5. 数据库流量激增，数据库崩溃 

6. 重启后仍然面对缓存中无数据可用 

7. Redis服务器资源被严重占用，Redis服务器崩溃 

8. Redis集群呈现崩塌，集群瓦解 

9. 应用服务器无法及时得到数据响应请求，来自客户端的请求数量越来越多，应用服务器崩溃 

10. 应用服务器，redis，数据库全部重启，效果不理想

**问题分析**

- 短时间范围内 

- 大量key集中过期

  

**解决方案（道）**平日要做的

1. 更多的页面静态化处理 

2. 构建多级缓存架构 

   ​	Nginx缓存+redis缓存+ehcache缓存 

3. 检测Mysql严重耗时业务进行优化 

   ​	对数据库的瓶颈排查：例如超时查询、耗时较高事务等 

4. 灾难预警机制 

   ​	监控redis服务器性能指标 

   ​		a.CPU占用、CPU使用率 

   ​		b.内存容量 

   ​		c.查询平均响应时间 

   ​		d.线程数 

5. 限流、降级 

   短时间范围内牺牲一些客户体验，限制一部分请求访问，降低应用服务器压力，待业务低速运转后再逐步放开访问



**解决方案（术）** 出现问题如何解决

1. LRU与LFU切换 ，删除策略进行切换

2. 数据有效期策略调整 

   ​	根据业务数据有效期进行分类错峰，A类90分钟，B类80分钟，C类70分钟 

   ​	过期时间使用固定时间+随机值的形式，稀释集中到期的key的数量 

3. 超热数据使用永久key 

4. 定期维护（自动+人工） 

   ​	对即将过期数据做访问量分析，确认是否延时，配合访问量统计，做热点数据的延时 

5. 加锁 慎用

**总结**

缓存雪崩就是瞬间过期数据量太大，导致对数据库服务器造成压力。如能够有效避免过期时间集中，可以有效解决雪崩现象的出现（约40%），配合其他策略一起使用，并监控服务器的运行数据，根据运行记录做快速调整。

![1632913610973](1632913610973.png)



## **缓存击穿**

**数据库服务器崩溃（2）**

1. 系统平稳运行过程中 

2. 数据库连接量瞬间激增 

3. Redis服务器无大量key过期 

4. Redis内存平稳，无波动 

5. Redis服务器CPU正常 

6. 数据库崩溃

**问题排查** 

1. Redis中某个key过期，该key访问量巨大 

2. 多个数据请求从服务器直接压到Redis后，均未命中 

3. Redis在短时间内发起了大量对数据库中同一数据的访问

**问题分析** 

- 单个key高热数据 
- key过期

**解决方案（术）**

1. 预先设定 

   以电商为例，每个商家根据店铺等级，指定若干款主打商品，在购物节期间，加大此类信息key的过期时长 注意：购物节不仅仅指当天，以及后续若干天，访问峰值呈现逐渐降低的趋势 

2. 现场调整 

   监控访问量，对自然流量激增的数据延长过期时间或设置为永久性key 

3. 后台刷新数据 

   启动定时任务，高峰期来临之前，刷新数据有效期，确保不丢失 

4. 二级缓存 

   设置不同的失效时间，保障不会被**同时**淘汰就行 

5. 加锁 

   分布式锁，防止被击穿，但是要注意也是性能瓶颈，慎重！

**总结** 

缓存击穿就是单个高热数据过期的瞬间，数据访问量较大，未命中redis后，发起了大量对同一数据的数

据库访问，导致对数据库服 务器造成压力。应对策略应该在业务数据分析与预防方面进行，配合运行监

控测试与即时调整策略，毕竟单个key的过期监控难度 较高，配合雪崩处理策略即可。



## **缓存穿透**

**数据库服务器崩溃（3）**

1. 系统平稳运行过程中 

2. 应用服务器流量随时间增量较大 

3. Redis服务器命中率随时间逐步降低 

4. Redis内存平稳，内存无压力 

5. Redis服务器CPU占用激增 

6. 数据库服务器压力激增 

7. 数据库崩溃

**问题排查** 

1. Redis中大面积出现未命中 

2. 出现非正常URL访问

**问题分析** 

- 获取的数据在数据库中也不存在，数据库查询未得到对应数据 
- Redis获取到null数据未进行持久化，直接返回 
- 下次此类数据到达重复上述过程 
- 出现黑客攻击服务器

**解决方案（术）** 

1. 缓存null 

   对查询结果为null的数据进行缓存（长期使用，定期清理），设定短时限，例如30-60秒，最高5分钟 

2. 白名单策略 

   a.提前预热各种分类数据id对应的bitmaps，id作为bitmaps的offset，相当于设置了数据白名单。	当加载正常数据时，放行，加载异常数据时直接拦截（效率偏低） 

   b.使用布隆过滤器（有关布隆过滤器的命中问题对当前状况可以忽略） 

3. 实施监控 

   实时监控redis命中率（业务正常范围时，通常会有一个波动值）与null数据的占比 

   ​		非活动时段波动：通常检测3-5倍，超过5倍纳入重点排查对象 

   ​		活动时段波动：通常检测10-50倍，超过50倍纳入重点排查对象 

   根据倍数不同，启动不同的排查流程。然后使用黑名单进行防控（运营） 

4. key加密 

   问题出现后，临时启动防灾业务key，对key进行业务层传输加密服务，设定校验程序，过来的key校验 

   例如每天随机分配60个加密串，挑选2到3个，混淆到页面数据id中，发现访问key不满足规则，驳回数据访问

**总结** 

缓存击穿访问了不存在的数据，跳过了合法数据的redis数据缓存阶段，每次访问数据库，导致对数据库服务器造成压力。通常此类 

数据的出现量是一个较低的值，当出现此类情况以毒攻毒，并及时**报警**。应对策略应该在临时预案防范方面多做文章。 

无论是黑名单还是白名单，都是对整体系统的压力，警报解除后尽快移除。