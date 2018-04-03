# 晒晒太阳补补钙

## Mysql

### innodb

### mvvc

### buffer pool

### B树、B+树

### 联合索引

### 事务的隔离级别

#### 读未提交 Read uncommitted

#### 读提交 Read committed

#### 重复读 Repeatable read

幻读
事务在插入已经检查过不存在的记录时，发现这些数据已经存在了，之前的检测获取到的数据如同鬼影一般。

#### Serializable

### java

#### 类加载机制

#### 重入锁

#### synchronized 和 Lock的区别

#### synchronized的实现原理

synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入临界区，同时它还可以保证共享变量的内存可见性。

同步代码块是使用monitorenter和monitorexit指令实现的，同步方法依靠的是方法修饰符上的ACC_SYNCHRONIZED实现。

同步代码块：monitorenter指令插入到同步代码块的开始位置，monitorexit指令插入到同步代码块的结束位置，JVM需要保证每一个monitorenter都有一个monitorexit与之相对应。任何对象都有一个monitor与之相关联，当一个monitor被持有之后，他将处于锁定状态。线程执行monitorenter指令时，将会尝试获取对象所对应的monitor所有权，即尝试获取对象的锁；

同步方法：synchronized方法则会被翻译成普通的方法调用和返回指令如invokevirtual、areturn指令，在字节码层面没有任何特别的指令来实现被synchronized修饰的方法，而是在Class文件的方法表中将该方法的access_flags字段中的synchronized标志位置1，表示该方法是同步方法并调用该方法的对象或者该方法所属的Class的JVM的内部对象表示Klass作为锁对象。

#### java的内存模型和jvm的内存模型 [reference](https://blog.csdn.net/suifeng3051/article/details/52611310)

Java内存模型即Java Memory Model，简称JMM。JMM定义了Java虚拟机（JVM）在计算机内存（RAM）中的工作方式。JVM是整个计算机虚拟模型，所以JMM隶属于JVM。JMM定义了多线程之间共享变量的可见性以及如何在需要的时候对共享变量进行同步。原始的Java内存模型效率并不是很理想，因此Java1.5对其进行了重构，java1.8沿用了1.5版本。

java线程之间的通信采用的是共享内存模型，指的就是Java内存模型（JMM)，JMM决定一个线程对共享变量的写入何时对另一个线程可见。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。

在JVM内部，Java内存模型把内存分成了两部分：线程栈区和堆区

当对象和变量存储到计算机的各个内存区域时，主要会面临的两个问题：

1. 共享对象对各个线程的可见性
2. 共享对象的竞争现象

##### 共享对象的可见性

当多个线程同时操作同一个对象时，如果没有合理的使用volatile和synchronized关键字，一个线程共享对象的更新有可能导致其他线程不可见。

共享对象存储在主内存，一个CPU的线程读取主内存数据到CPU缓存，然后对共享对象做了更改，但CPU缓存中更改后的对象还没有flush到主存，此时线程对共享对象的更改对其他CPU中的线程是不可见的。最终就是对每个线程都会拷贝共享对象，而且拷贝的对象位于不同的CPU缓存中。

要解决共享对象可见性问题，可以使用volatile关键字。volatile关键字可以保证变量会直接从主内存读取，而对变量的更新也会写到主内存。volatile原理是基于CPU内存屏障指令实现的。

##### 竞争现象

如果多个线程共享一个对象，如果他们同事修改这个对象，就会产生竞争现象。

synchronized代码块可以保证同一时刻只能有一个线程进入代码竞争区，synchronized代码块也能保证代码块中所有变量都将会从主内存中读，当线程退出代码块时，对所有变量的更新将会flush到主存。

##### 指令重排序

在执行程序是，为了提高性能，编译器和处理器会对指令做重排序。但是，JMM确保在不同编译器和不同的处理器平台上，通过插入特定类型的Memory Barrier来禁止特定类型的编译器重排序和处理器重排序，为上层提供一致的内存可见性保证。

1. 编译器优化重排序：编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
2. 指令集并行的重排序：如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
3. 内存系统的重排序：处理器使用缓存和读写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

##### 数据依赖

如果两个操作访问同一个变量，其中一个为写操作，此时这两个操作之间存在数据依赖性。编译器和处理器不会改变存在数据依赖性关系的两个操作的执行顺序，即不会重排序。

##### as-if-serial

不管怎么重排序，单线程下的执行结果不能被改变，编译器、runtime和处理器都必须遵循as-if-serial语义。

##### 内存屏障（Memory Barrier）

内存屏障可以禁止特定类型处理器的重排序。内存屏障又称内存栅栏，是一个CPU指令：

1. 保证特定操作的执行顺序。
2. 影响某些数据（或是某条指令的执行结果）的内存可见性。

编译器和CPU能够重排序指令，保证最终相同的结果，尝试优化性能。插入一条Memory Barrier会告诉编译器和CPU：不管什么指令都不能和这条Memory Barrier指令重排序。

Memory Barrier所做的另一件事就是强制刷出各种CPU cache，如一个Write-Barrier（写入屏障）将刷出所有在Barrier之前写入的cache数据，因此任何CPU上的线程都能读到这些数据的最新版本。

如果一个变量时volatile修饰的，JMM会在写入这个字段之后插入一个Write-Barrier指令，并在读这个字段之前插入一个Read-Barrier指令，这就保证：

1. 一个线程写入变量a后，任何线程访问该变量都会拿到最新值。
2. 在写入变量a之前的写入操作，其更新的数据对其他线程也是可见的。以为Memory Barrier会刷出cache中所有先前的写入。

happens-before

从jdk5开始，java使用心得JSR-133内存模型，基于happens-before的概念来阐述操作之间的内存可见性。
在JMM中，如果一个操作的执行结果需要对另一个操作可见，那么这两个操作之间必须要存在happends-before干洗，这两个操作既可以在同一个线程中，也可以在不同的两个线程中。

1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中任意的后续操作。
2. 监视器锁规则：对一个锁的解锁操作，happends-before于随后对这个锁的加锁操作。
3. volatile域规则：对一个volatile域的写操作，happens-before于任意线程后续对这个volatile域的读。
4. 传递性规则：如果A happends-before B，且B happends-before C，那么A happens-before C。

两个操作之间具有happends-before关系，并不意味着前一个操作必须要在后一个操作之前执行！仅仅要求前一个操作的执行结果，对于后一个操作是可见的，且前一个操作按照顺序排在后一个操作之前。

## Redis [Redis命令](http://redisdoc.com/)    [Redis Doc](https://github.com/antirez/redis-doc)

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

### [数据类型](https://github.com/antirez/redis-doc/blob/master/topics/data-types.md)

#### Strings

  Strings are the most basic kind of Redis value. Redis Strings are binary safe, this means that a Redis string can contain any kind of data, for instance a JPEG image or a serialized Ruby object.
  A String value can be at max 512Megabytes in length.
  You can do a number of interesting things using strings in Redis, for instance you can:
* Use Strings as atomic counters using commands in INCR family: INCR, DECR, INCRBY.
* Append to strings with the APPEND command.
* Use Strings as a random access vectors with GETRANGE and SETRANGE.
* Encode a lot of data in little space, or create a Redis backed Bllom Filter using GETBIT and SETBIT.

#### Lists

  Redis Lists are simply lists of strings, sorted by insertion order. It is possible to add elements to a Redis List pushing new elements on head(on the left) or on the tail(on the right) of the list.

  The LPUSH command inserts a new element on the head, while RPUSH inserts a new element on the tail. A new list is created when one of this operations is performed against an empty key. Similarly the key is removed from the key space if a list operation will empty the list. There are very handy semantics since all the list commands will behave exactly like they were called with an empty list if called with a non-existing key as argument.

  The max length of a list is 2^32 - 1 elements (4294967295, more than 4 billion elements per list).

  The main feathers of Redis Lists from the point of view of time complexity are the support for constant time insertion and deletion of elements near the head and tail, even with many millions of inserted items. Accessing elements is very fast near the extremes of the list but is slow if you try accessing the middle of a very big list, as it it an O(N) operation.

  You can do many interesting things with Redis Lists, for instance you can:
* Model a timeline in a social network, using LPUSH in order to add new elements in the user time line, and using LRANGE in order to retrieve a few recently inserted items.
* You can use LPUSH together with LTRIM to create a list that never exceeds a given number of elements, but just remembers the latest N elements.
* Lists can be used as a message passing primitive, See for instance the well known Resque Ruby library for creating background jobs.
* You can do alot more with lists, this data type supports a number of commands, including blocking commands like BLPOP.

#### Sets

Redis sets are an unordered collection of Strings. It is possible to add, remove, and test for existence of members in O(1)(constant time regardless of the number of elements contained inside of Set).

Redis Sets have the desirable property of not allowing repeated members. Adding the same element multiple times will result in a set having a single copy of this element. Practically speaking this means that adding a member does not require a check if exist then add operation.

A very interesting thing about Redis Sets is that they support a number of server side commands to compute sets starting from existing sets, so you can do unions, intersections, differences of sets in very short time.

The max number of members in a set is 2^32-1(4294967295, more than 4 billion of members per set).

You can do many interesting things using Redis Sets, for instance you can:

* You can track unique things using Redis Sets. Want to know all the unique IP addresses visting a giving blog post? Simply using SADD every time you process a page view. You are sure repeated IPs will not be inserted.
* Redis Sets are good to represent relations. You can create a tagging system with Redis using a Set to represent every tag. Then you can add all the IDs of all the Objects having three tags at the same time? Just use SINTER.
* You can use Sets to extract elements at a random using the SPOP or SRANDMEMBER commands.

#### Hashes

Redis Hashes are maps between string fields and string values, so they are the perfect data type to represent objects(e.g. A User with a number of fields like name, surname, age, and so forth):

A hash with a few fields (where few means up to one hundred or so) is stored in a way that takes very little space, so you can store millions of objects in a small Redis instance.

While Hashes are used mainly to represent objects, they are capable of storing many elements, so you can use Hashes for many other tasks as well.

Every hash can store up to 2^32-1 field-value pairs(more than 4 billion).

Sorted sets

Redis Sorted Sets are, similar to Redis Sets, non repeating collections of Strings. The difference is that every member of a Sorted Set is associated with scope, that is used in order to take the sorted set ordered, from the smallest to the greatest score. While members are unique, scores may be repeated.

With sorted sets you can add, remove, or update elements in a very fast way(in a time proportional to the logarithm of the number of elements). Since elements are taken in order and not ordered afterwards, you can get range by score or by rank(position) in a very fast way. Accessing the middle of a sorted set is also very fast, so you can Sorted Sets as a smart list non repeating elements where you can quickly access everything you need: element in order, fast existence test, fast access to elements in the middle!

In short with sorted sets you can do a lot of tasks with great performance that are really hard to model in other kind of databases.

With Sorted Sets you can:

* Take a leader board in a massive online game, where every time a new score is submitted you update it using ZADD. You can easily take the top use using ZRANGE, you can also, given an user name, return its rank in the listing using ZRANK. Using ZRANK and ZRANGE together you can show users with a score similar to a given user. All very quickly.
* Sorted Sets are often used in order to index data that is stored inside Redis. For instance if you have many hashes representing users, you can use a sorted set with elements having the age of user as the score and the ID of the user as the value. So using ZRANGEBYSCORE it will be brivial and fast to retrieve all the users with a given interval of ages.

Sorted Sets are probably the most advanced Redis data types.

#### Bitmaps and HyperLogLogs

Redis also supports Bitmaps and HyperLogLogs which are actually data types based on the String base type, but having their own semantics.

1. String
    string类型是二进制安全的。redis的string可以包含任何数据。比如jpg图片或者序列化的对象。
    string类型是redis最基本的数据类型，一个redis中字符串value最多可以是512M。
2. Hash
    Redis hash是一个键值对集合。
    Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。
3. List
    Redis列表是简单的字符串列表，按照出入顺序排序。可以添加元素到列表的头部（左边）或者尾部（右边）。他的底层实际是一个链表。
4. Set
    Set是string类型的无序集合。通过HashTable实现。
5. zset
    zset和set一样也是string类型元素的集合，且不允许重复成员。不同的是每个元素会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的，但分数（score）却可以重复。

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

### 版本---保证分布式数据原子操作

Zookeeper会为每个Znode维护一个叫Stat的数据结构，存在三个版本信息：

* version：znode数据的修改次数
* cversion：znode子节点修改次数
* aversion：znode的ACL修改次数

version表示对数据节点数据内容的变更次数，强调的是变更次数，因此就算数据内容的值没有发生变化，version的值也会递增。

### Leader选举机制

### ACL---保障数据的安全

ACL全称为Access Control List（访问控制列表），用于控制资源的访问权限。zk利用ACL策略控制节点的访问权限，如节点数据读写、节点创建、节点删除、读取子节点列表、设置节点权限等。
在传统的文件系统中，ACL分为两个维度，一个是属组，一个是权限，一个属组包含多个权限，一个文件或目录拥有某个组的权限即拥有了组里的所有权限，文件或子目录默认会继承自父目录的ACL。（例如linux系统的权限控制）
而在Zookeeper中，znode的ACL是没有继承关系的，每个znode的权限都是独立控制的，只有客户端满足znode设置的权限要求时，才能完成相应的操作。Zookeeper的ACL，氛围三个维度：schema、id、permission，通常表示为：schema:id:permission，schema代表授权策略，id代表用户，permission代表权限。

### Paxos算法

#### 定义

算法中的参与者主要分为三个角色，同时每个参与者又可兼领多个角色：

1. proposer提出提案，提案信息包括提案编号和提议的value；

2. acceptor收到提案后可以接受（accept）提案；

3. learner只能“学习”被批准的填；

算法保证一致性的基本语义：

1. 决议（value）只有在被proposers提出后才能被批准（未经批准的决议称为“提案（proposal）”）；

2. 在一次Paxos算法的执行实例中，只批准（chose）一个value；

3. learners只能获得被批准（chosen）的value；

有上面的三个语义可以演化为四个约束：

1. P1：一个acceptor必须接受（accept）第一次收到的提案；

2. P2a：一旦一个具有value v的提案被批准（chosen），那么之后的任何acceptor再次接受（accept）的提案必须具有value v;

3. P2b：一旦一个具有value v的提案被批准（chosen），那么以后任何proposer提出的提案必须具有value v；

4. P2c：如果一个编号为n的提案具有value v，那么存在一个多数派，要么他们中所有人都没有接受（accept）编号小于n的任何提案，要么他们已经接受（accept）的所有编号小于n的提案中编号最大的那个提案具有的value v;

#### 基本算法

算法（决议的提出与批准）主要分为两个阶段：

1. prepare阶段：

1.1 当Pro

### 悲观锁、乐观锁

悲观锁：具有严格的独占和排他特性，能有效的避免不同事务在同一数据并发更新而造成的数据一致性问题。实现原理就是：假设A事务正在对数据进行处理，那么在整个处理过程中，都会将数据处于锁定状态，在这期间，其他事物将无法对这个数据进行更新操作，直到事务A完成对该数据的处理，释放对应的锁。一份数据只会分配一把钥匙，如数据库的表锁或者行级锁。

乐观锁：具体实现是，表中有一个版本字段，第一次读的时候，获取到这个字段。处理完业务逻辑开始更新的时候，需要再次查看该字段的值是否和第一次的一样。如果一样更新，反之拒绝。乐观锁就是假定多个事务在处理过程中不会相互影响，悲观锁正好相反，因此乐观锁在事务处理的大部分时间不需要加锁处理。乐观锁非常适用于数据竞争不大，事务冲突较少的应用场景中。最经典的应用是：JDK的CAS处理。

### 几个小算法题

1. 求中位数
2. 蓄水池问题
3. 数组循环赋值问题
