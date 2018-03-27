# 晒晒太阳补补钙

## Mysql

### innodb

### mvvc

### buffer pool

### B树、B+树

### 联合索引

## Redis

Redis中数据存储有2种模式：

* cache-only: 只做“缓存”服务，不持久化数据。

* persistence: 内存中的数据持久备份到磁盘文件，在服务重启后可以恢复。

### AOF、RDB

* AOF(Append-only file)
    将“操作+数据”以格式化指令的方式追加到操作日志文件的尾部，在append操作返回后，才进行实际的数据变更。实际是操作日志，类似Mysql的binlog。AOF的3种同步选项：
  * always: 每条aof记录都立即同步到文件
  * everysec: 每秒同步一次，redis推荐模式
  * no: redis不直接调用文件同步，交给操作系统处理，操作系统根据Buffer填充情况/通道空闲时间等择机触发同。

* RDB(Redis DataBase)
    在某个时间点将数据写入一个临时文件，持久化结束后，用这个临时文件替换上次持久化的文件。通过配置redis在n秒内如果超过m个key被修改，执行一次RDB操作。这个操作类似于保存一次redis所有数据，所以也叫snapshots。

### 数据类型

1. String
    string类型是二进制
2. Hash
3. List
4. Set
5. zset

## 分布式锁

## java并发

## 网络

### 三次握手、四次挥手

## Dubbo

### 集群容错

* Failover
  失败自动切换，当出现失败，重试其他服务器。通常用于读操作，但重试会带来更长的延迟。可通过retries来设置重试次数
* Failfast
  快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的操作，比如新增记录。
* Failsafe
  失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。
* Failback
  失败自动回复，后台记录失败请求，定时重发。通常用于消息通知操作。
* Foking
  并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多的服务资源，可以通过 ```forks="2"``` 来设置最大并行数。
* Broadcast
  广播调用所有提供者，逐个调用，任意一台报错则报错。通常用于通知所有提供者更新缓存或者日志等本地资源信息。

### 负载均衡

* Random 随机

* RoundRobin 轮询

* LeastActive 最少活跃调用数

* ConsistentHash [一致性Hash](http://en.wikipedia.org/wiki/Consistent_hasing)

### 线程模型

* Dispatcher [补充1](http://manzhizhen.iteye.com/blog/2391177)
  * all 所有消息都派发到线程池，包括请求，响应，链接事件，断开事件，心跳等。
  * direct 所有消息都不派发到线程池，全部在IO县城上直接执行。
  * message 只有请求响应消息派发到线程池，其他连接断开事件，心跳等消息，直接在IO线程上执行。
  * execution 只有请求消息派发到线程池，不含响应，响应和其他连接断开事件，心跳等消息，直接在IO线程上执行。
  * connection 在IO线程上，将连接断开事件放入队列，有序逐个执行，其他消息派发到线程池。

### LoadBalance策略

### 多协议

* 不同服务在性能上试用不同协议进行传输，比如大数据用短链接协议，小数据大并发用长连接协议。
* 协议可以自行扩展

## Zookeeper

### 集群角色

有三种角色

* Leader
* Follower
* Observer

Zookeeper集群中的任何一台机器都可以响应客户端的读操作，且全量数据都存在于内存中，因此Zookeeper更适合以读为主的应用场景。当不是Leader的服务器收到客户端的事物操作，也会转发到Leader，让Leader进行处理。
Zookeeper集群的所有机器通过一个Leader选举过程来选定一台被称为Leader的机器，Leader服务器为客户端提供读和写服务。Follower和Observer都能提供读服务，不能提供写服务。两者唯一的区别在于，Observer机器不参与Leader选举过程，也不参与写操作的“过半写成功“策略，因此Observer可以在不影响性能的情况下提升集群的读性能。

每个Server在内存中存储了一份数据；
Zookeeper启动时，将从实例中选举一个Leader（Paxos协议）；
Leader负责处理数据更新等操作（Zab协议）；
一个更新操作成功，当且仅当大多数Server在内存中成功修改

### Leader选举机制
