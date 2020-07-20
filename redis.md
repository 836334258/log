#### reids持久化策略

- RDB(redis database) 快照方式，将某一时刻的内存数据，已二进制的方式写入磁盘
- AOF(append only file)文件追加方式，记录所有操作命令，并以文本形式追加到文件中
- 混合持久化方式，Redis4.0新增，混合持久化方式结合了RDB和AOF的优点，在写入的时候，先把当前数据以RDB的形式写入文件的开头，再将后续的操作命令以AOF的格式写入文件，这样既能保证Redis重启的速度，又能降低数据丢失的风险。

#### 配置主从同步

[网址]: https://www.jianshu.com/p/ff2df53fb954



- Redis的主从模式采用的是RDB文件同步的方式，因为Redis的服务端，数据量有可能非常的大，所以从性能考虑，没有采用AOF快照来同步。

- 大概过程

  1. 从机上线，主动连接主机，发送`SYNC`命令
  2. 主机接到命令后，执行`BGSAVE` 命令，生成RDB文件
  3. 主机向从机发送RDB文件，开始同步数据
  4. 同步之后，主机将最近的更新，采用命令的形式同步到从机

- 配置

  - `masterauth 123456` 如果主服务器有密码，需要配置

  - `replica-read-only yes` 从服务器是否只读，默认是

  - ` replicaof host.docker.internal 6378` docker从服务器配置 主机ip 主机port 

    [文档]: https://docs.docker.com/docker-for-mac/networking/	"f文档"

#### 配置文件

- `aof-use-rdb-preamble yes` 开启混合持久化
- 惰性删除 redis4.0
  - `lazyfree-lazy-eviction no` 当redis的运行内存超过maxmeory时，是否开启`lazy free` 机制删除
  - `lazyfree-lazy-expire no` 设置了过期时间的键值，当过期之后是否开启`lazy free`机制
  - `lazyfree-lazy-server-del no`  有些指令在处理已存在的键时，会带有一个隐式的 del 键的操作，比如 rename 命令，当目标键已存储，Redis 会先删除目标键，如果这些目标键是一个 big key, 就会造成阻塞删除的问题，此配置表示在这种场景中是否开启 lazy free 机制删除.
  - `slave-lazy-flush` 针对 slave (从节点) 进行全量数据同步，slave 在加载 master 的 RDB 文件前运行 flushall 来清理自己的数据，它表示此时是否开启 lazy free 机制删除.
    建议开启其中的 lazyfree-lazy-eviction, lazyfree-lazy-expire, lazyfree-lazy-server-del 等配置，这样就可以有效的提高主线程的执行效率.
- `save 60 1000` 快照保存，默认保存名为dump.rdb ，当满足`1000秒内至少有60个键被改动时` 自动保存一次数据
- `dbfilename dump.rdb` 快照默认保存名
- `appendonly no` 开启aof，当redis执行改变数据集的命令时，这个命令会被追加到aof文件的末尾
- `appendfilename "appendonly.aof"` 
- `appendfsync everysec` 有no，always，everysec三个选项
- `requirepass 123456` 连接redis需要的密码

#### 事务

启动事务`mulit` 执行事务 `exec`  观察一个key，如果在事务执行过程中这个key发生改变，则事务全部回滚 `watch`



#### 命令

`discard` 取消事务