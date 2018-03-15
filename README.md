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
- AOF(Append-only file)
    将“操作+数据”以格式化指令的方式追加到操作日志文件的尾部，在append操作返回后，才进行实际的数据变更。实际是操作日志，类似Mysql的binlog。AOF的3种同步选项：
    * always: 每条aof记录都立即同步到文件
    * everysec: 每秒同步一次，redis推荐模式
    * no: redis不直接调用文件同步，交给操作系统处理，操作系统根据Buffer填充情况/通道空闲时间等择机触发同。
- RDB(Redis DataBase)
    在某个时间点将数据写入一个临时文件，持久化结束后，用这个临时文件替换上次持久化的文件。通过配置redis在n秒内如果超过m个key被修改，执行一次RDB操作。这个操作类似于保存一次redis所有数据，所以也叫snapshots。

### 数据类型
1. String
    string类型是二进制
2. Hash
3. List
4. Set
5. zset

### 

## 分布式锁


## java并发



