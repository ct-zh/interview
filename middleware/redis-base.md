# redis

## 基础数据类型
### 对象
> 查询对象类型使用 `type <key>`; 查询对象编码使用`object encoding <key>`;

redis实现了6种数据结构: 简单动态字符串、双端链表linkedlist、整数集合intset、哈希表ht、压缩列表ziplist、跳表skiplist;

redis一共有五个对象: `string`、`hash`、`list`、`set`、`zset`;这些对象在不同情况下组成的数据结构不一样,见下表:

|对象|数据结构|
|---|---|
|string|int|
|string|embstr|
|string|raw|
|list|ziplist|
|list|linkedlist|
|hash|ziplist|
|hash|hashtable|
|set|intset|
|set|hashtable|
|zset|ziplist|
|zset|skiplist|

### string对象
在set时判断编码类型:
- int: 可以用long类型(32位和int一样是4字节;64位是8字节)保存的整数;
- embstr: 字符串,不能用long表示的长整数/浮点数;要求大小不能超过39字节

其他情况使用raw类型;

embstr和raw类型的联系与区别: 
- 都是使用简单动态字符串`sdshdr`生成;
- embstr的`redis object`与`sdshdr`的地址分配是连续的,只需要分配一次地址;raw需要分别创建两个对象,在`redis object`的`ptr`字段指向sds;

相互转换:在int和embstr类型的object使用`APPEND`与`SETRANGE`两个命令时,会转换成raw类型;

#### 简单动态字符串sdshdr
结构体,字段为:
- len: 已用长度;
- free: 空闲字节;
- buf: 字节数组,保存数据;

对比C语言字符串优点:
1. 获取len复杂度为O(1);
2. 防止缓冲区溢出;
3. 不需要每次修改都重新分配空间;
4. 二进制安全;
5. 可以使用部分c字符串函数;

##### 空间分配规则
1. 每次修改时先看free是否足够,不够才会重新分配空间;
2. 修改后len小于1MB时,分配的free=len;
3. 修改后len大于1MB时,每次修改free增加1MB;
4. 如果是缩短操作,只是len和free的值修改,数组里被缩短的部分数据并不会变动;


### list对象
符合这些条件时使用ziplist编码(可以配置):1. 所有字符串元素长度小于64字节; 2. 元素数量小于512个;否则使用linkedlist

#### ziplist
压缩列表是一段连续的内存块,按照顺序排序,内容是:
- zlbytes: 压缩列表总字节数;
- zltail: 最后一个结点离起始地址的字节数;
- zllen: 压缩列表结点数;
- entryX: 代表第X个结点;
- zlend: 压缩列表结尾字符, OxFF(十进制255);

对于每个结点entry,又有三个结构:
- previous_entry_length: 前一个结点的长度;(前结点长度小于254,该字段占1字节,否则占5字节; 通过这个参数可以倒序遍历整个ziplist)
- encoding: content的类型与长度
- content: 具体数据

当发生插入更新删除操作,某个结点需要重新分配内存地址,那他往后的所有结点都需要重新分配,这叫做*连锁更新*,最坏复杂度为O(N^2),平均复杂度为O(N); **因此zip只是用来节约内存的一种设计,只适合小数据.**

#### linkedlist
记录了前置结点指针的链表;


### hash对象
符合这些条件时使用ziplist编码(可以配置):1. 所有kv长度小于64字节; 2. k-v数量小于512个;否则使用hashtable;

#### 字典dict
redis字典是由hashtable组成,ht结构: 哈希数组`**table` 大小size、掩码sizemask(用于计算索引)、used节点数量;

redis使用拉链法解决hash冲突: 增加一个next指针指向下一个结点;
> 还有一种解决hash冲突的方法是开放定址法.拉链法的缺点是：指针需要额外的空间，故当结点规模较小时，开放定址法较为节省空间;

根据键计算出hash值,再根据hash值与sizemask计算出索引值; 使用的是`Murmurhash`算法

##### rehash
字典dict里面有两个重要的字段:
- `ht`数组: 固定长度为2;`ht[0]`里面保存了具体数据,`ht[1]`用于rehash,每次字典操作时,都会将索引kv更新到`ht[1]`,直到所有kv都更新完毕,rehash完毕之后替代`ht[0]`;
- rehashidx: -1代表未rehash;大于等于0代表正在rehash;


### set对象
当1. 所有元素都是整数;2. 元素数量不超过512; 时使用intset实现,否则使用hashtable来实现;

#### intset
整数集合组成: 编码encoding、长度length、数组contents; contents根据encoding的类型来,可以是`int16_t`、`int32_t`、`int64_t`;contents的元素是从小到大排列的,使用二分查找,时间复杂度是O(logN);

默认是int16_t,当插入数据大于类型长度时再进行升级操作: 根据新类型分配对应的内存空间,循环将每个元素转换成新类型; 

> 升级操作优点是节约内存,只在需要时才会申请大类型的数组; int set不支持降级操作;

### zset对象
当元素数量小于128个,并且长度都小于64字节时,使用ziplist来实现; 否则使用skiplist;

ziplist实现zset时,两个结点保存一个元素,第一个结点保存member,第二个结点保存score;所有元素按score从小到大在ziplist里排列;

skiplist实现的zset是由跳表和字典共同实现的, 两种数据结构搭配使用:
- skiplist和dict的item保存的都是指针,不会产生空间浪费;
- skiplist可以实现快速的范围型的操作,如ZRANK、ZRANGE;
- dict可以实现O(1)级别的查找操作;

#### 跳表
跳跃表通过维护若干条捷径加快链表的查找效率;

redis4以前跳表最高32层,之后最高64层,可以通过配置来修改;

#### 编码的时间复杂度
- 简单动态字符串: 获取长度O(1);创建/更新/删除:O(N);
- 双向链表: 长度/表头(/表尾)结点/给定结点的值/给定结点的前置(/后置)的值:O(1);查找/更新/删除: O(N)
- 字典: 只有释放字段时是O(N);其他操作都是O(1);
- 跳跃表: 添加/删除/查找,按照分值范围查询/删除: 平均O(logN) (接近二分查找与平衡树) 最坏O(N); 判断是否在跳跃表的分值范围内:O(1);
- 整数集合: 添加/删除: O(N);查找: O(logN) (二分查找); 获取len、占用空间、获取给定索引上的数据: O(1);
- 压缩表: 添加/查找/删除: 平均O(N),最坏O(N^2) (触发更新操作); 给定结点,获取前/后结点:O(1);

## 内存回收
> `info memory`命令查看redis内存情况;
redis使用引用计数的方式实现gc(`redisObject.refcount`字段);另外还有一个字段`redisObject.lru`记录该对象最后被操作的时间,当redis打开`maxmemory`选项并超过对应内存时,lru算法会优先释放空转时间最长的对象;

redis在启动时创建了0-9999这一万个字符串对象,当涉及到对应值操作时可以直接使用这些对象,这些对象的`redisObject.refcount`字段+1;

### 缓冲内存
缓冲内存主要包括：客户端缓冲、复制积压缓冲、AOF重写缓冲:
- 客户端缓冲分为三种普通客户端、发布订阅客户端、从客户端;
- 复制积压缓冲: 1MB大小用于增量复制;
- AOF重写缓冲: AOF重写期间保存写命令

### 内存碎片
Redis默认内存分配器采用jemalloc; 简单地说jemalloc将内存空间划分为三个部分：Small class、Large class、Huge class，每个部分又划分为很多小的内存块单位,比如Large Class里有4kb块、8kb块等等;如果一个对象为5kb,那分配器给他分配的就是8kb的块,剩下的3kb内存就变成了内存碎片，不能再被分配给其他对象;

在对key做频繁更新操作和大量过期key被删除的时候会导致碎片率上升,解决办法:
- 重启节点的方式整理碎片;
- redis4.0之后增加了memory命令,可以使用: `memory purge`命令重新整理内存碎片;

## 过期键删除策略
一般有三种方式进行过期数据回收:
- 惰性删除: 访问时检查key是否过期;缺点是当大量数据不会访问时会变成垃圾数据,几乎可以看作内存泄漏了;
- 定时删除: 设置key的ttl时设置一个定时器,到时间删除key;缺点是每个key都要开个定时器,占用大量cpu时间;
- 定期删除: 定一个时间点,轮询所有key或者选若干key(自定义算法),删除过期key;定期删除对比上面两种方法相对折中,缺点是需要设计足够优秀的算法;

redis使用的是惰性删除+定期删除两种方法组合;定期算法是每次取出一定数量的随机key检查,会有一个全局变量记录检查进度,直到该数据库所有key都被遍历一遍;

> 写RDB数据时会做过期的判断,载入RDB时只有主数据库会做过期判断;AOF记录操作时遇到过期键会写一条delete key的记录;AOF重载时会忽略过期键;
> 
> 从数据库不会主动删除过期键;主数据库删除过期键时会给从数据库发送del命令删除过期键;


## RDB与AOF
- RDB使用`BGSAVE`命令开启一个子进程,将当前数据保存到RDB二进制文件里;
- 可以开启自动间隔保存,在某个时间内对数据库进行了多少次修改,符合条件就会进行`BGSAVE`
- redis启动时会使用RDB从磁盘恢复数据; 因为AOF的更新频率大于RDB,如果AOF开启了,redis会选择使用AOF恢复数据;
- AOF通过将执行的命令写入AOF缓冲区,根据`appendfsync`的配置不同,会在不同机制下将缓冲以APPEND的方式刷入AOF文件保存;
- AOF载入就是重新全部执行一遍AOF文件里的命令即可;
- redis会fork一个子进程用于重写AOF文件,重写完毕后会替换当前AOF文件(只有这个操作会阻塞主进程),重写策略是直接读取当前数据库的值,然后改写成对应的新增语句(处理非string对象时会根据配置的最大元素数量拆分成若干条新增语句);


## 复制、主从、集群
### 复制
- 使用`slaveof host port`命令成为某个服务器的从服务器;使用`info replication`查看状态;
- redis5.0之后改用`REPLICAOF host port`命令;
- 完整重同步通过发送RDB文件实现;
- redis2.8通过`psync`命令实现了部分重同步功能,尽量把redis版本升级到2.8以后再使用复制功能;
- 通过复制偏移量offset与复制积压缓冲区(replication backlog)来实现部分重同步,如果主从的偏移量范围仍然在缓冲区内,则直接使用缓冲区内的数据同步;如果偏移量大于缓冲区,则从服务器需要全部同步主服务器的数据;(注意根据业务设置`repl-backlog-size`的值,一般是`平均重连时间x主服务器平均每秒写命令量`)
- 验证: 需要在主服务设置`requirepass`的值,在从服务器设置`masterauth`的值,用来验证是否有复制权限;


### sentinel与cluster的区别
- sentiel是redis官方推荐的HA(high avaliable高可用)方案,一主多从,主要解决单机缓存可用性的问题;
- cluster是做sharding(分片),一个key通过hash算法分配到不同的slot上，所有不同节点存储的数据是不一样的;

### sentinel相关概念
`redis-server ~/sentinel.conf --sentinel` 启动sentinel; 官方建议至少3个sentinel实例;
- sentinel执行命令`SENTINEL master/slaves <service name>`看到主/从服务器的相关信息;
- 主/从机器通过命令`INFO REPLICATION`,看到复制相关信息;

> 官方不建议配合docker使用: Docker执行端口重新映射，打破了对其他Sentinel进程和主机副本列表的Sentinel自动发现。
> 
> sentinel.conf相关配置与官方文档见: https://redis.io/topics/sentinel


#### 内部运行流程
1. 使用sentinel的代码初始化redis服务器,根据配置文件初始化sentinel监视的主服务器列表; (用sentinel代码启动的redis-server不支持一般的redis命令);
2. sentinel服务器创建面向服务器的连接, 先与master创建连接,再与slave创建连接(master INFO上报slave):
    - 一个命令连接: 1. 每10s一次给master发INFO, 获取master与slave的状态;2.发现新的slave时,同样10s一次给slave发送INFO;
    - 一个订阅连接: 发送`__sentinel__:hello`命令,其他服务器与sentinel都会收到这个信息并更新相关数据;

3. sentinel在获取服务器的INFO后知道其他sentinel的存在,再相互建立命令连接; sentinel之间不会建立订阅连接;
4. sentinel每秒一次向master、slave、其他sentinel发送PING命令,收到PONG、LOADING、MASTERDOWN等回复来更新其他服务器的状态;未回复或者回复超时的则标记为主观下线状态;
5. 当master进入客观下线状态时,向其他sentinel发送`SENTINEL is-master-down-by-addr`命令,询问master是否已经下线;收到足够数量(按配置)的回复后就会将master设置为客观下线状态;
6. 推举leader,具体流程如下:
    1. 如果master进入客观下线状态, sentinel立马会向其他sentinel推举自己为leader(发送`SENTINEL is-master-down-by-addr`并加上自己的run id);
    2. 规则是*先到先得*, 如果sentinel未设置leader,收到消息后回复配置纪元(选举次数的计数器)与leader的run id;
    3. sentinel收到回复消息后会对比配置纪元与run_id与自己是否相同,相同说明自己票数+1;
    4. 获得半数以上sentine支持的leader将当选全局leader,如果在时限内未投出全局leader,则会重新选举,直到出现全局leader;
7. leader会对master进行故障转移: 
    1. 挑选slave,先看状态(5秒内回复过INFO消息)、再看与master断开时间(不能大于一个配置时间)、再看slave的复制偏移量(找最大的)、最后看运行id;
    2. 选出来的slave发送`slaveof no one`命令,观察返回的INFO,当INFO里role字段变成master时继续操作;
    3. 给其他slave发送`slaveof 新的master`,修改复制目标;
    4. 将旧master的role标记为slave,如果旧master重新上线,则给他发送`slaveof 新master`的命令;


### cluster相关概念
redis-server 启动时需要`cluster-enabled`参数为yes; redis-cli需要添加`-c`/`--cluster`参数连接集群;

命令:
- `CLUSTER MEET <ip> <port>`: 将其他节点加入到该节点集群;
- `CLUSTER FORGET`: 将某个节点踢出集群;
- `CLUSTER nodes`: 查看集群节点情况;
- `CLUSTER ADDSLOTS <slot> [slot...]`: 槽指派;
- `CLUSTER REPLICATE <node_id>`将当前节点设置为node_id节点的从节点;

#### 概念详解: 集群结构源码:
每个节点保存了一个`clusterstate`的struct, 重要字段有:
- `*myself`:  保存了自己节点的信息,类型为`clusterNode`;
- `currentEpoch`: 配置纪元,用于选举实现failover;
- `state`: 状态,在线/下线;
- `size`: 槽数量
- `*nodes`: 集群节点名单, `clusterNode`字典;包括自己;
- `slots`: 槽指派数组,这是一个指针数组,key代表槽,value代表该槽属于节点的指针;

`clusterNode`struct重要字段有: `ip`,`port`以及
- `configEpoch`: 这个节点的配置纪元;
- `*link`, 类型为`clusterLink`;
- `slots`: 槽指派数组,这是一个二进制数组,key代表槽,value为1代表该槽属于该节点,为0则不属于;

`clusterLink`重要字段有: `sndbuf`/`rcvbuf`输入/输出缓冲区; clusterNode: 与这个link相关的节点指针;

#### 概念详解: 握手连接
节点A执行`CLUSTER MEET`命令:
- 会将meet的节点数据写入`clusterstate.nodes`,并发送meet消息; 
- 节点B收到消息同样写入数据,并回复 PONG消息;
- A向B发送PING消息,完成握手,AB两个节点相连;
- A使用Gossip协议传播给其他所有节点,其他节点与B握手连接;

#### 概念详解: 分片方式
- 整个数据库分为16384个槽slot
- 通过算法`CRC16(key) && 16383`可以计算出key对应的槽;槽与key的对应关系保存在`clusterState.slots_to_keys`跳跃表里,分值代表槽值;
- 只有将16383个槽全部分配完毕,集群才能上线;
- 节点只能使用0号数据库;
- 当redis-cli发送请求命令时,server发现key不在自己负责的槽上,会给cli发送一个move信息;cli根据move信息转向正确的节点;(对用户不可见)

实现方式: 
- `clusterNode.slots`: 是一个二进制数组,16384/8=2048个字节,数组里面1位代表一个槽,该位数据为1代表该槽属于该结点;
- `clusterState.slots`: 是一个指针数组,key代表槽,value代表该槽属于节点的指针;

#### 概念详解: 重新分片
增加一个新节点, 使用meet命令让其加入到集群中,此时因为没有负责的槽,所以不能接受任何读写操作; 删除一个节点,需要先把其负责的槽分配给其他节点;这里都涉及到重新分片的操作;redis使用`redis-trib`来实现重新分片;

操作流程:
- 给节点A发送命令: `cluster setslot <slot> importing <source_id>`, 准备从source处导入对应槽的数据;
- 给其他节点发送命令: `cluster setslot <slot> migrating <target_id>`, 准备给target复制对应槽的数据;
- 其他节点再执行: `cluster getkeysinslot <slot> <count>`, 
- `migrate <target_ip> <port> <keyname> 0 <timeout> keys <key...>`, 3.0.6往后支持批量迁移;
- 最后执行`cluster setslot <slot> node <target_id>`申明这些槽属于target节点;

ask错误:
- cli向节点获取对应key的数据;
- 键key存在于该节点: 返回数据;
- 键不在该节点, 并且未迁移槽: 返回未吵到;
- 正在迁移槽: 返回ASK错误;
- cli获取ASK信息,去访问迁移节点;(对用户不可见)

> ask与move信息的区别: 都会导致cli转向,但是ask只会转向一次,而move会导致cli往后所有的槽查询都转向去查询move的节点;


#### 概念详解: failover
`cluster replicate <node_id>`开始复制对应node; 使用的是单机的代码;

- 所有节点都会互相发送`PING`消息;
- 未在规定时间内回复`PONG`消息的都设置为疑似下线;
- 半数以上主节点都将某个节点设置为疑似下线,那执行这个判断的节点会广播一遍将其标记为`FAIL`的广播;
- 从节点发现主节点`FAIL`之后进入选举流程,会选中一个从节点成为主节点,并将槽重新指派给自己;
- 新主节点广播一条PONG消息,申明自己已经成为新的主节点了;

选举流程:
- 从节点发现主节点`FAIL`,会广播一条`CLUSTERMSG_TYPE_FAILOVER_AUTH_REQUEST`消息,要求其他主节点给自己投票;
- 当某个从节点收到回复消息时开始计数,当收到 主节点数/2 + 1 个回复时,直接升级为主节点;
- 如果没有从节点收集到足够多的票,则开启新的配置纪元,重新投票;
- 与sentinel的投票一样,这里的投票也会验证配置纪元是否在同一个纪元以内,都是基于raft的头领选举算法;

fail会导致的后果:
- 如果超过一半master fail了,整个集群都会挂掉;
- 某个master与slave全挂, 集群会出现大量失败 挂掉master数/master数;
- 某个master挂掉,在slave选举过程中不能写,导致部分写失败;
- slave挂掉,没有影响;


#### 概念详解: 消息
1. meet消息: 请求对方加入集群;
2. PING: 
    - 每个节点每秒(可配置)随机选5个节点,从这5个节点中挑选最长未联系时间的节点并发送ping消息;
    - 每个节点判断,如果某个link节点的PONG消息超过配置的timeout,则会向该节点发送ping消息;
3. PONG: 用于回复ping与meet消息, 或者用于failover. pong消息会附带自己的最新信息,其他节点用来更新;
4. FAIL: 用于广播某个节点已下线;
5. PUBLISH: 某个节点收到PUBLISH消息,会执行命令,并广播一次PUBLISH消息;

MEET、PING、PONG三个消息组成redis的Gossip协议实现;

## 多路IO复用模型与事件
### 多路IO
linux里面一切都是用文件表示的,一个socket句柄,也是用文件描述符fd来表示;
- select: 将socket.fd与一个bitmap相对应,select接受这个bitmap并传入内核,内核根据bitmap监听指定fd变动,发生变动修改对应的位; 用户每次需要循环整个bitmap拿到有事件的fd,将数据写入buf;

    select存在四个问题: 1. bitmap只有1024位; 2. FD_SET每次读完都需要重置; 3. 用户态内核态开销; 4. 每次需要O(n)遍历bitmap;

- poll: bitmap改成pollfd struct,包含fd、event与revent;用户通过event传入感兴趣事件,内核通过revents反馈就绪事件;

    pool解决了select第1、2个问题,最大支持fd变成65535个;

- epoll: `epoll_create`: 把用户关心的fd放在内核的一个事件表里面; `epoll_ctl`: 传入操作的fd、操作类型等等; 在一个while循环里使用`epoll_wait`等待fd的事件,返回值是就绪fd的个数; 如果检测到事件,就将所有就绪事件从内核事件表复制到他第二个参数events指向的数组里,之后只需要根据就绪文件个数进行循环; 

- LT与ET模式: LT:当`epoll_wait`检测到事件通知应用程序后,应用可以不立即处理事件,下次调用`epoll_wait`时还会向应用程序通告此事件,直到事件被处理; ET: 下次调用`epoll_wait`不会再通知该事件了,所以程序必须在`epoll_wait`通知时就立即处理;

- epoll采用回调的方法检测就绪事件,时间复杂度是O(1),解决了select的第4个问题; 尤其在上百万连接时尤其有用; (epoll适用于连接数量多、活动连接少的情况); epoll可以工作在ET高效模式下,而select和poll只能工作在LT模式下; 但是在小连接数,并且活跃高情况下poll可能时更好的选择, 因为epoll花费了一部分资源去处理惊群效应;

- 注意网上很多文章说epoll是通过用户态与内核态共享内存的方式处理事件表, 解决了用户态内核台切换的开销; *[这个是错误的说法](https://www.zhihu.com/question/39792257)*

> redis通过epoll实现多路IO复用;IO多路复用,复用指定的是对主线程的复用,而不是对IO的复用;

> 注意select、poll、epoll本质上都是阻塞同步IO(这里的阻塞是指没有事件发生时, 会阻塞在select、poll、epoll_wait这里),只不过epoll采用回调的方式实现; epoll_wait返回时，实际上是把内核中的就绪链表拷贝给应用层，应用层处理事件的同时，内核更新并继续维护当前的就绪链表，如此反复; kernel中发现readylist正在被使用时，会把就绪事件放在ovflist中当处理完readylist后，会检查ovflist是否有事件。

### 文件事件
- 虽然是单线程运行,但是*文件事件处理器*通过IO复用达到监听多个套接字;
- redis对select、epoll、evport、kqueue等IO多路复用函数都封装了同样的接口,在编译时会选择当前平台下性能最高的函数;
- 监听多个socket的读事件和写事件;

### reference
> - linux 高性能服务器编程 第九章IO复用
> - [epoll源码详解](https://www.nowcoder.com/discuss/26226)
> - [epoll(2) 使用及源码分析](https://www.cnblogs.com/shuqin/p/11743567.html) https://www.cnblogs.com/shuqin/p/11772651.html

## 其他功能
- 通知: (>2.8)订阅给定频道,可以知道某个键进行了哪些操作(订阅key),或者知道某个操作被什么键执行了(订阅del、get之类的命令);
- 发布与订阅: 可以订阅/退订频道(`subscribe`/`unsubscribe`),或者订阅/退订模式(`psubscribe`/`punsubscribe`);`publish`往频道推消息;`pubsub`查阅消息;
- 事务与事务的ACID:  
    - `MULTI`开启事务,`EXEC`提交事务,`WATCH`监控某个key,如果发生改动则修改`REDIS_DIRTY_CAS`标识值(乐观锁),redis会拒绝执行标识打开的事务;
    - A原子性, redis事务是通过打包命令批量入队执行,所以符合原子性;
    - C一致性: 入队时就会检查命令错误的问题, 其他错误在执行时出现也不会影响事务的执行,所以符合一致性;
    - I隔离型: 因为是单线程串行执行,所以具有隔离型;
    - D持久性: 如果开启了RDB和AOF,则具有持久性

- Lua脚本: 
    - 通过`EVAL`执行脚本;`SCRIPT LOAD`载入lua脚本, `SCRIPT FLUSH`清空脚本;`SCRIPT KILL`停止正在执行的脚本(超时情况);
    - redis会给lua脚本计算SHA1校验和, `EVALSHA`可以使用SHA1值执行以前执行过的脚本; 主从同步时,如果不能确保从库有对应的SHA1值, redis会将`evalsha`转换成对应的`eval`语句;

- 排序: `sort <key>`,默认升序; `ALPHA`参数可以实现字符串排序; `DESC`参数执行降序排列;`LIMIT <offset> <count>`分页;
- bitarray:  使用SDS来保存位数组, 逆序保存简化`setbit`操作; 
- 慢查询日志: `slowlog get`, `slowlog len`, `slowlog reset`;
- 监视器, 实时接受服务器当前执行的命令: `MONITOR`


### 客户端与服务器的实现
// todo

### redis新版本特性


## reference
- [Redis开发运维实践指南](https://www.w3cschool.cn/redis_all_about/)
