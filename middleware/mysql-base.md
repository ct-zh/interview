# mysql知识体系与常见问题粗谈
> 适用范围程序员而不是DBA, 只是很粗浅的谈一谈,受限于笔者水平,不保证正确性与时效性;

对于程序员来说, 日常开发比较重要的mysql知识有: 1. sql与索引; 2. 事务和锁;

## 名词解释
### 顺序扫描与随机扫描
什么时候随机扫描什么时候顺序扫描??

在生产环境中，由于数据的 type 值可能是随机的，因此这些数据记录通常也不会按顺序排列，而是随机的散布在整个表空间中。在访问索引记录时就需要大量的随机扫描，其性能可能比全表顺序扫描更差。

随机扫描在超过某个阈值时（大概 20%），MySQL 数据库就开始转向全表扫描;

### DDL与DML、DCL
- DML(data manipulation language) 数据操纵语言,也就是select,update,insert,delete;
- DDL(data definition language)数据定义语言: CREATE,ALTER,DROP等用于*定义表或者改变表结构、数据类型、表之间连接等操作*
- DCL(Data Control Language)数据控制语言: 用来设置或者更改用户的数据库权限,如grant,revoke;

### OLAP/OLTP
- 业务类系统主要供基层人员使用，进行一线业务操作，通常被称为OLTP（On-Line Transaction Processing，联机事务处理）;
- 数据分析的目标则是探索并挖掘数据价值，作为企业高层进行决策的参考，通常被称为OLAP（On-Line Analytical Processing，联机分析处理）;
- 从功能角度来看，OLTP负责基本业务的正常运转，而业务数据积累时所产生的价值信息则被OLAP不断呈现，企业高层通过参考这些信息会不断调整经营方针，也会促进基础业务的不断优化，这是OLTP与OLAP最根本的区别

### 回表
1. innodb的主键索引是聚簇索引,使用B+树实现的,在叶子结点上保存了整行的数据;
2. 普通索引也是B树的实现, 不同的是叶子结点上保存的是主键的值;
3. 那么走普通索引查到的自然是主键的值,还需要去主键索引查到记录;这个操作就叫回表.

### 索引覆盖(Covering index)
> 只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。
1. 见sql:`select id,username from user where username="zhangsan"`, 查询字段为`id`与`username`,不包含其他字段;
2. 那么给`username`字段建立索引,对于`username`索引树,可以查到主键的值;
3. 因为获取的内容只有`username`和`id`,此时就不需要回表再去查聚簇索引了,这就叫*索引覆盖*;

### 那如果我要查其他字段呢
1. 见sql`EXPLAIN SELECT id,username,sex from user_test WHERE username="kangkang"`,因为多了sex字段,肯定不会触发索引覆盖了;
2. 将`username`索引升级成联合索引`index(username, sex)`, 此时又符合索引覆盖了;


## 事务与锁
### ACID
> see [MySQL 是如何实现四大隔离级别的](https://www.zhihu.com/question/263820564/answer/289269082)
>
> [数据库宕机以后恢复的过程？如何保证事务的ACID特性？](https://www.zhihu.com/question/59729819/answer/284180647)


innodb通过锁来实现隔离性;而原子性、持久性、一致性通过redo log、undo log、bin log来保证;

- 原子性（Atomicity）: 

    一个事务对状态的改变是原子的。要么都发生，要么都不发生。这些改变包括数据库的改变、消息以及对转换器的操作; 

- 一致性（Consistency）: 

    事务执行的结果必须是使数据库从一个正确的状态迁移到另一个正确的状态。(比如对于唯一索引,必须维护新增的数据不与现有的数据冲突);

- 隔离性（Isolation）: 

    事务之间不相互影响;

- 持久性（Durability）: 

    持久性是指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。即使发生宕机、崩溃等故障，数据库也可以将数据恢复; *这个持久性只在mysql层面实现,硬件故障等原因不包括在内*

    redo、undo、binlog等用来加强持久性;

### MVCC与 undo log、redo log
MVCC多版本并发控制,特点是不加锁的前提下使读写不冲突,大幅提升事务的并发性;
- innodb有个版本号,每启动一次事务版本号都会增加;
- 在每行数据后面增加了两个字段: 创建时间与删除时间;
- insert时创建时间设置为当前版本号;
- delete时删除时间设置为当前版本号;
- select时,只可查询到: 创建时间小于等于当前版本号的,并且删除时间为空或者大于当前版本号;
- update时,插入一条新记录,并且原来行的删除时间记录为当前版本号;

undo:
- 是逻辑日志;
- 将之前的操作都记录下来，在发生错误时回滚;
- 做相反的工作，比如一条INSERT对应一条DELETE;对每个UPDATE,对应一条相反的UPDATE,将修改后的行修改回去。undo日志用于事务的回滚操作进而保障了事务的原子性。
- 写入时机:1. DML操作修改聚簇索引前，记录undo日志; 2. 二级索引记录的修改，不记录undo日志;

redo:
- 重做日志(redo log)用于数据库的崩溃恢复, 用来保证事务的持久性;
- redo分为redo log buffer(保存在内存)和redo log file(保存在磁盘),是物理日志;
- 流程为: 1. 原始数据从磁盘读入内存,执行相关操作;2. 写redo log buffer，记录数据被修改后的值;3. 事务commit时，将redo log buffer中的内容刷新到 redo log file;
- 聚集索引、二级索引、undo页面的修改，均需要记录Redo日志;

### MVCC的具体实现原理
在InnoDB中MVCC的实现通过两个重要的字段进行连接：DB_TRX_ID和DB_ROLL_PT，在多个事务并行操作某行数据的情况下，不同事务对该行数据的UPDATE会产生多个版本，数据库通过DB_TRX_ID来标记版本，然后用DB_ROLL_PT回滚指针将这些版本以先后顺序连接成一条 Undo Log 链。

- DB_TRX_ID: 事务id，6byte，每处理一个事务，值自动加一。
- DB_ROLL_PT: 回滚指针，7byte，指向当前记录的ROLLBACK SEGMENT 的undolog记录，通过这个指针获得之前版本的数据。该行记录上所有旧版本在 undolog 中都通过链表的形式组织。

UPDATE
1. 事务A给对应行加排它锁;
2. 将修改行原本的值拷贝到Undo log中;
3. 修改目标值产生一个新版本，将DB_TRX_ID设为当前事务ID，将DB_ROLL_PT指向拷贝到Undo log中的旧版本记录;(这样就形成了一个链表,每个结点有旧版本结点的指针)
4. 记录redo log，binlog;

INSERT: 产生一条新的记录，该记录的DB_TRX_ID为当前事务ID;

DELETE: 特殊的UPDATE，在DB_TRX_ID上记录下当前事务的ID，同时将delete_flag设为true，在执行commit时才进行删除操作;

select使用ReadView来实现;

#### ReadView一致性视图实现高并发下的隔离性
对于read uncommitted，直接读取最新值即可，而serializable采用加锁的策略通过牺牲并发能力而保证数据安全，因此只有RC和RR这两个级别需要在MVCC机制下通过ReadView来实现。(MVCC只在innodb的RC、RR两个隔离级别下使用)

在read committed级别下，readview会在事务中的每一个SELECT语句查询发送前生成;因此每次SELECT都可以获取到当前已提交事务和自己修改的最新版本。而在repeatable read级别下，每个事务只会在第一个SELECT语句查询发送前或显式声明处生成;

> ReadView是与SQL绑定的，而并不是事务，所以对于RC,即使在同一个事务中，每次SQL启动时构造的ReadView的up_trx_id和low_trx_id也都是不一样的;

ReadView创建一个数组,保存当前所有活跃事务(开始但未提交事务)的id,将数组中事务ID最小值记为低水位`m_up_limit_id`,当前系统中已创建事务ID最大值+1记为高水位`m_low_limit_id`;(根据事务ID去遍历undo log)

查询流程:
1. 如果DB_TRX_ID小于m_up_limit_id,说明该版本在ReadView生成前就已经完成提交，该版本可以被当前事务访问;
2. 若被访问版本的DB_TRX_ID大于等于m_low_limit_id，说明该版本在ReadView生成之后才生成，因此该版本不能被访问;
3. 若被访问版本的DB_TRX_ID在[m_up_limit_id, m_low_limit_id)区间内，则判断DB_TRX_ID是否等于当前事务ID，等于则证明是当前事务做的修改，可以被访问，否则不可被访问, 继续向上寻找;
4. 最后，还要确保满足以上要求的可访问版本的数据的delete_flag不为true，否则查询到的就会是删除的数据。

##### update的问题
情况: 
1. 事务A将值m从10更新成m+1;
2. 事务B查询m的值,得到10;
3. 事务B将m的值更新成m+1,阻塞,等待A提交;
4. 事务A查询m,结果是11,提交;
5. 事务B查询,结果是12,提交;

问题在于按照上面的流程,事务A对于事务B应该属于未提交状态,是读不到对应版本的,最后事务B update结果应该是11才是?

上面的问题在于UPDATE操作都是读取当前读(current read)数据进行更新的，而不是一致性视图ReadView，因为如果读取的是ReadView，那么事务B的操作会丢失。当前读会读取记录中的最新数据，从而解决上面情形下的并发更新丢失问题。


#### reference
> 高性能mysql第一章,多版本并发控制;
>
> [MVCC机制总结](https://www.jianshu.com/p/d67f0329d3bf)
> 
> [MVCC 机制的原理及实现](https://chenjiayang.me/2019/06/22/mysql-innodb-mvcc/)
> 
> [MVCC实现](https://zhuanlan.zhihu.com/p/40208895)


### 行锁
#### 记录锁 Record Key
- InnoDB的行级锁是通过给索引项加锁来实现的。因此，在使用InnoDB存储引擎时，只有通过索引检索时，才会使用行级锁，否则会直接升级为表锁;
- 如果索引选择性差导致需要锁住相当多行,或者是大量随机扫描(数据不按照顺序排列). 随机扫描在超过某个阈值时（大概 20%），MySQL数据库会开始转向全表扫描;
- 在执行 SQL 语句时，如果能使用到主键、唯一索引或者区分度非常好的索引，记录锁的性能表现还是非常好的;

#### 间隙锁 Gap Lock
> see https://www.imooc.com/read/88/article/2354
两个索引记录之间的间隔就叫做索引的间隙(根据b树原理); 间隙锁没有锁定具体数据,所以没有共享锁和排他锁区分;

间隙锁在 InnoDB 中的作用是保证某个间隙的数据，在锁定的情况下不会发生任何变化，以此来防止*幻读*的产生。

#### 临键锁 Next-key Locks
- 临建锁是记录锁+间隙锁组合
- 当查询的索引含有唯一属性时，InnoDB 会对 Next-Key 锁进行优化，降级为记录锁;
- 在 “REPEATABLE READ” 隔离级别下，MySQL 使用 Next-Key 锁进行加锁；但是在 READ-COMMIT 隔离级别下，除了外键约束和唯一性检查仍然会加间隙锁之外，其他的情况均只有记录锁。

### 显式、隐式锁
可以用`select * from information_schema.innodb_locks`查询到的是显式锁,例如`select ... for update`或者`select ... lock in share mode`这种用户手动加上的就是显式锁;

隐式锁是mysql程序逻辑设计里应该加锁的部分,当其他事务访问到加锁行时,其他事务会将隐式锁转换称显式锁,并将自己放置到等待队列里面;

### 死锁
> see https://www.imooc.com/read/88/article/2355

产生原因:
1. 互斥使用：资源不能被共享，只能由一个进程在同一时刻使用。
2. 持有和等待：进程已持有部分资源并等待得到另外的资源，而这些资源又被其他进程所占用还没释放。
3. 非抢占分配：已经分配的资源不能从相应的进程中被强行剥夺。
4. 部分分配或循环等待：至少存在包含两个进程的一个循环链，链中一个进程等待被链中另外一个进程占有的资源。

常见情况:
1. session1对A加锁, session2对B加锁; 然后session1更新B,session2更新A, 产生死锁;
2. 多个并发操作一个唯一索引,也会导致死锁;

innodb有两种死锁解决办法: 
1. 锁等待超时时间,默认50s; 
2. 等待图: 正在运行的事务抽象为图,查询到环说明发生死锁。对比权重和加锁时间，回滚权重小和更晚持有锁的事物。

innodb 等待超时和等待图两种算法同时进行检测;根据死锁的类型不同，根据等待图能马上检测到的死锁就会马上处理，其他场景产生的锁等待会在超时后处理。(一般情况下等待图算法会优先于等待超时)

查看死锁信息: `show engine innodb status`, `LATEST DETECTED DEADLOCK`行

#### 如何避免死锁
1. 合理的索引设计：可以防止表锁或锁住大量的数据行，降低死锁的概率;(更新删除全走主键索引，否则高并发下容易产生死锁)
2. 尽可能小事务;
3. 合理的事务逻辑：死锁本质上是产生了环路，在业务开发中控制并发事务访问同一资源的请求逻辑，也可以降低死锁的概率;

### 表锁
显式加锁:
```sql
lock tables <tablename> <lock type> -- 加锁
unlock table    -- 解除所有锁
```

lock type:
- READ: 所有的用户只能读取被锁表，不能对表进行修改（包括执行 LOCK 的用户），当表不存在 WRITE 写锁时 READ 读锁被执行;
- READ LOCAL: 除了允许 INSERT 命令以外执行的锁与 READ 相同;
- WRITE: 除了当前用户被允许读取和修改被锁表外，其他用户的所有访问被完全阻止。一个 WRITE 写锁被执行仅当所有其他锁取消时;
- LOW PRIORITY WRITE: 低优先级的读锁，在等待时间内（等待其他锁取消），其他用户的访问将被认为是执行了 READ 读锁，因此将增加等待时间;

### MDL元数据锁
元数据锁是server层的锁，表级锁，主要用于隔离DML和DDL操作之间的干扰。每执行一条DML、DDL语句时都会申请MDL锁，DML操作需要MDL读锁，DDL操作需要MDL写锁;

常见场景是某个事务对表进行读写操作, 另外有session想对表结构进行修改,此时session就会被阻塞;

> see https://blog.csdn.net/finalkof1983/article/details/88063328
>
> [有了MDL锁视图，业务死锁从此一目了然](https://zhuanlan.zhihu.com/p/200755169)


## sql与索引
### explain的 type字段分析
速度从快变慢:
- system最快：不进行磁盘IO; 通常在查询mysql系统值,或者查询tmp表时出现;
- const：PK或者unique上的等值查询; 
- eq_ref：PK或者unique上的join查询，等值匹配，对于前表的每一行(row)，后表只有一行命中
- ref：非唯一索引，等值匹配，可能有多行命中
- range：索引上的范围扫描，例如：between/in/>
- index：索引上的全集扫描，例如：InnoDB的count
- ALL最慢：全表扫描(full table scan)


### explain的 extra字段分析
> from [如何利用工具，迅猛定位低效SQL](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651962587&idx=1&sn=d197aea0090ce93b156e0774c6dc3019&chksm=bd2d09078a5a801138922fb5f2b9bb7fdaace7e594d55f45ce4b3fc25cbb973bbc9b2deb2c31&scene=21#wechat_redirect)

- Using where: 代表SQL使用了where条件过滤了部分数据; *扫表常见的优化方法为，在where过滤属性上添加索引*
- Using index: 无需回表,性能最好;
- Using index condition: 多一个字段需要回表,这类SQL语句性能也较高，但不如Using index。

- Using filesort: filesort代表需要对所有记录进行文件排序,性能极差; *典型的，在一个没有建立索引的列上进行了order by，就会触发filesort，常见的优化方案是，在order by的列上添加索引，避免每次查询都全量排序。*

- Using temporary: 代表需要建立临时表(temporary table)来暂存中间结果;性能较差; group by和order by同时存在，且作用于不同的字段时，就会建立临时表;

- Using join buffer (Block Nested Loop): 说明需要进行嵌套循环计算;(笛卡尔积, 如果内层sql外层sql的type都为all, 代表两个扫表,那么计算量是总 rows x rows);这种sql性能巨差无比, 两个关联表join，关联字段均未建立索引，就会出现这种情况;*常见的优化方案是，在关联字段上添加索引，避免每次嵌套循环计算。*


### 常见索引的实现方式
- 哈希表: 查找速度接近O(1),缺点是无序,无法进行范围查找;
- b+tree: innodb最常用的索引,查找速度O(logN);
- 跳跃表: redis的zset通过跳表实现分数范围查询,速度平均O(logN),最差O(N)
- 字典树(Trie树): 多叉树, 利用公共前缀来查找;
- LSM tree: 常见于NoSql,最大的特点就是写入速度快，主要利用了磁盘的顺序写，pk掉了需要随机写入的B-tree;

B+Tree索引和Hash索引区别: 哈希索引适合等值查询，但是无法进行范围查询; 哈希索引没办法利用索引完成排序; 哈希索引不支持多列联合索引的最左匹配规则; 如果有大量重复键值得情况下，哈希索引的效率会很低，因为存在哈希碰撞问题;

用hash索引: 业务只做大量单列的等值查询,并且重复率很低,不会发生碰撞;此时查找效率O(1)>b树的O(logN)


### b+树一般有几层
- inndb的默认页大小是16k,单行长度最大不能超过65355(see: [VARCHAR(50)中的50到底是能存50个字还是50个字节？](https://www.imooc.com/read/88/article/2357)),假设单行大小是1kb,那么一个叶子结点能保存16条数据; 
- 假设主键为bigint类型，长度为8字节，而指针大小在InnoDB源码中设置为6字节，这样一共14字节,那么非叶子结点可以存16k/14=1170个数据,那么两层高的B+树叶子结点有1170个,每个保存16个数据,也就是`1170*16=18720`; 如果是3层高的B+树,第二层有`1170*1170=1368900`个结点,每个结点对应的第三层叶子结点能保存16个数据,最终能保存`1368900*16`约等于两千万条记录;如果是四层B+树:`1170^3*16=256亿`条数据;

因为 InnoDB 的数据页默认是 16K，每个页中至少存放 2 行数据，因此建议 VARCHAR 字段的总长度不要超过8K,不然可能会行溢出;(会创建一个溢出指针额外指向一个空间)

### 类型的长度
- int是int32也就是4字节,无符号型最多保存42亿多(主键id上限); 
- bigint长度是8字节;
- int系列因为是定长了,所以对于int(10)这样的字段,仍然是占4字节,只是会在右侧填充不同数量的0而已(see [为什么会有程序员使用INT(20)或INT(1)这样的设计，这样是否合理？](https://www.imooc.com/read/88/article/2359))
- varchar是变长,最多保存65532字节(最大65535,再减去部分额外开销),根据编码不同实际长度不同,utf8里每个字符占3字节,所以最大长度为21844(65532/3);

### 如何优化超大的分页查询
> see: [如何优化超大的分页查询](https://www.imooc.com/read/88/article/2362)
MySQL数据库采用了基于代价的查询优化器，而查询代价的估算是基于CPU代价和IO代价。如果MySQL在查询代价估算中，认为采取顺序扫描方式比局部随机扫描的效率更高的话，就会放弃索引，转向顺序扫描的方式。

sql改写优化:
1. 索引覆盖: 只能拿到索引列和id;
2. 子查询优化:`select * from t where id>=(覆盖索引查询) limit 100`: 缺点是不能加其他where条件, 而且这个是按照id排序的, 可能需要按照其他来排序;
3. 延迟关联:`select a.* from t a inner join (select id from t order by 索引 limit xx) b on a.id=b.id`; 这是最常用的sql改写,子查询优化版,速度极快;
4. 记录书签:`select * from t where id >=xxx order by x limit 0, 100` 速度最快,缺点是需要书签, 书签不一定是主键,但是如果不是主键,要确保书签字段不存在大量重复值; 
5. 反向查找: 只适合查最后几页,asc改成desc,`select * from t order by x desc limit 0, 100`

### count(*)、count(1)、count(id)哪个性能更好？
> see [count(*)、count(1)、count(id)哪个性能更好？](https://www.imooc.com/read/88/article/2364)
由于每一行数据都需要判断自己是否对这个事务可见，所以 InnoDB 就必须把数据一行一行的全部读出，然后进行判断。版本号小于当前事务的，或者没有行删除标记的，才会被纳入统计的表的总行数。

在mysql5.7里, COUNT (*) 和 COUNT (1) 是最快的，其次是 COUNT (id)，最慢的就是 COUNT 使用了强制主键的情况:

**InnoDB 在处理`COUNT (*)`的时候，不一定使用我们平时认为最快的主键索引。**,例如某个区分度很低的字段建了索引,那么对于count(*)来说就会被优化器优化到走这个索引;

如果可以保证MySQL的自增值是完全连续的,可以用max查索引最大值记录书签，查看最后一页数据;

### 数据库索引应该如何设计
> see [数据库索引应该如何设计](https://www.imooc.com/read/88/article/2365)
>
> see 高性能mysql //todo

三星索引通俗版:
1. 一星：where通过索引取出数据;
2. 二星：`Order by`列加入索引中;
3. 三星：不用回表;

mysql是基于成本的优化器:`总代价 = IO 代价 + CPU 代价`;具体到 SQL 子句中，MySQL 数据库会认为 GROUP BY、ORDER BY 的操作不走索引的代价，会高于 WHERE 子句不走索引的代价。

>  有一个常见的优化方法：如果 create_time 的顺序和主键 ID 的顺序是一致的，将 ORDER BY create_time 修改成 ORDER BY id。

进行 SQL 调优的时候，要按照 SQL 语句执行的顺序进行优化，重点处理执行成本比较高的部分：
1. 如果是多表 JOIN，先看 JOIN 的条件是否合理，列上是否有索引，避免笛卡尔积的产生。
2. 检查 WHERE 条件的索引是否合理，尽最大可能缩小结果集的大小。
3. 检查 GROUP BY 条件上是否有索引，如果没法使用索引，MySQL 会通过临时表完成 GROUP BY 的操作。
4. 检查 ORDER BY 条件是否利用到了索引，如果没有索引，使用排序算法将结果集放入临时表中进行排序。

不一定不能给基数低的列建立索引;如果数据的倾斜度高,经常查询,可以考虑创建索引;(举例: status字段只存在1、2、3三个值,大部分值都是3,此时如果想查1和2,可以考虑建立索引,查3和全表扫描就差不多了)



### 为什么我明明创建了索引，SQL却无法用到
> see [为什么我明明创建了索引，SQL却无法用到](https://www.imooc.com/read/88/article/2366)

以下情况适用于where, 也适用于 JOIN ON 和 HAVING 子句
1. where + 索引A orderby+索引B/groupby+索引B, mysql优先使用orderby或者groupby的索引;
2. 使用了困难谓词, 如 NOT、LIKE ‘%xx’、OR、IN;
    - LIKE: 如果值是`"xx%"`，可以正常使用索引;值是’% xx’或’% xx%’，则通常情况下无法使用索引;
    - OR:
        1. OR 条件的两边都是同一个索引列的情况下，如果 WHERE 条件是主键，完全能使用索引。
        2. OR 条件的两边都是同一个索引列的情况下，如果 WHERE 条件不是主键，则是否使用索引取决于 MySQL 查询优化器的代价估算。
        3. OR 条件的两边是不同的索引列，是否使用索引也取决于 MySQL 查询优化器的代价估算。
        4. 如果能使用索引，MySQL 会使用索引合并技术合并计算结果。如果代价太高仍然会走全表扫描。
        5. 如果多个 OR 条件中有其中一个条件没有索引，则必须进行全表扫描。

3. 隐性转换: 查询的类型写错了,如查varchar类型的数据,传入的是int型的数据;(如果索引列是 INT 型的，隐性转换可以使用到索引。但如果索引列是字符型的，隐性转换无法使用索引。)
4. 在索引列上使用函数或者运算符;(MySQL 8.0 开始引入了函数索引，解决了一小部分问题)
5. 列对象参与了运算,如`col1 + col2 = 2`;
6. 使用了不等于表达式:`!=`或者`<>`;(这种 SQL 的写法不是完全使用不到索引，也要看索引的基数等情况。)

> 这种情况下，SQL 的执行性能比较差：查询数据表，WHERE 条件中不包含索引列，但是 GROUP BY 子句的条件中包含索引列。

实际上只要给 SQL 语句中的 WHERE 子句和 ORDER BY/GROUP BY 子句加上一个复合索引就可以解决全表扫描的问题;

在复合索引的使用中无法触发索引使用的几种情况：
1. 没有使用到索引前缀
2. 使用了复合索引中的全部列，但索引键不是 AND 操作


### SQL优化的一些思路
> see [SQL优化的一些思路](https://www.imooc.com/read/88/article/2369)

1. 首先是减少硬盘访问，这一点对应的就是尽量减少大规模的数据和索引扫描。
2. 其次是减少网络传输和交互次数，在客户端和 server 端交互时尽可能的返回更少的数据，减少连接次数。
3. 再次就是节省内存空间，避免一次读取大批量的数据到内存中。
4. 最后是 CPU 的使用，避免在 MySQL 端做业务逻辑处理，避免调用到 MySQL 的排序、分组算法等等。

从查询优化的维度来看，单表扫描的算法有如下几种, 性能从高到低：
1. 行扫描: 只操作主键;
2. 只读索引扫描: 不用回表,覆盖索引;
3. 多个索引扫描: 索引合并优化,在一个表中使用多个索引，然后对结果合并;
4. 索引扫描
5. 顺序扫描: 一般在全表扫描，或者索引的选择度差时出现

如何建立索引:
1. WHERE 条件中频繁使用的字段应该建立索引;
2. 选择率太低的字段不适合建立索引;
3. 更新非常频繁的字段不适合建立索引;
4. 避免建立冗余索引;
5. 为经常做排序、分组的字段设计复合索引;
6. 对长字符串索引,应该定制一个前缀长度，可以节省大量的索引空间;

> 选择率可以参考: `show index from <tablename>;`的`Cardinality`字段;


### 怎么建立索引
#### 联合索引
联合索引必须注意:
1. 联合索引的顺序*不是*指where的顺序, where顺序不重要,优化器会自动优化成索引的顺序;
2. 出现范围查询则停止匹配,(只索引当前范围查询的字段, 联合索引后面的字段不再走索引);
3. 不要重复建立索引, 索引(a,b,c)包含了索引(a),索引(a,b);
4. 不能参与任何运算;

测试需要注意:
1. explain: `using index`未必代表命中索引(a,b,c),可能只是命中了索引(a);检测方法是看`key_len`字段的长度(命中索引越多索引字段越长);大部分完全命中联合索引的explain type应该是ref;
2. 可能出现ab索引a做范围查询, b仍然可以走索引的情况, 这是走了mysql5.6以后的*索引下推*, 关闭索引下推:`SET optimizer_switch = 'index_condition_pushdown=off';`

- `A=? AND B=? AND C=?`: 并不一定要(A,B,C),只要把列选择性高的排在前面就行了;abc/acb/bac/bca/cba/cab

- `A > ? AND B=?`: (B,A), 

    - 因为如果建立的是(a,b)索引，那么只有a字段能用得上索引,而且如果A的随机查询范围够大,可能会被优化成扫表;
    - ba索引,a范围查询仍然可以用到a索引;

- `A > ? AND B=? AND C>?`: (B,A)或者(B,C); 看A、C的数据比较

- `A=? and B=? AND C>?`: abc/bac能走全索引;

- `A=? order by B`: (A,B)先索引到A, 此时B已经有序了; 与`a=? and b>?`不同的是这里只能用到a索引;

- `A>? order by B`: 对A建索引; 因为a的值是一个范围，这个范围内b值是无序的，没有必要对(a,b)建立索引;(b,a)也用不到索引;

- `A=? and b=? and c>? order by c`: abc/bac; 这里c在where条件里面,能走到索引里;如果c不在where条件,就只会用到ab索引;

- `A in (? ... ?) and b>?`: 具体问题具体分析,如果a的范围小,可以建立ab索引,mysql5.6以后会触发索引下推,可以走完ab索引,查询type是range范围查询;如果a范围大,可能会转成扫表; (通常我们都限制in查询数量不超过50个,一般来说是可以建索引的); 补充测试: `a in () AND b in ()`会走全表扫描;

- `A=? and B in (...) AND C>? order by c`: 建ab索引,c用不到索引; type=range, extra=Using where; Using MRR; Using filesort


### 索引下推
> 索引下推是5.6的新特性
索引下推(index condition pushdown, ICP),索引条件下推优化可以减少存储引擎回表次数，也可以减少MySQL服务器从存储引擎接收数据的次数。

- 例如ab联合索引, `a > ? and b = ?` 按照最左匹配原则,b字段就走不了索引了;
- 5.6以前会将a拿到的数据做回表,每条记录回表一次;
- 5.6后会在索引内部再走一次b字段的判断,筛选掉一部分数据,再拿剩下的数据去回表;

## mysql监控与观测
Zabbix + Percona Monitoring Plugins
> see https://www.imooc.com/read/88/article/2348


## mysql大表的导出导入
> from [mysql的insert与update效率提高上万倍的经历](https://blog.csdn.net/cleanfield/article/details/6415596)

- 从表A导出数据的一条条insert+update进入表B, 效率极低根本跑不完;
- 将sql整理成10份, 类似并行写,效率还是很低,几个小时都跑不完;
- 按照db和table进行分类,再按照key分成N块，起N个进程并行执行; 1分钟内跑完;

关键点在于 让乱序执行的sql,变成了顺序执行. 从而减少锁表,甚至不锁表;
1.任务的队列化，如果任务的执行会涉及到大范围的随机跳转操作，而这种跳转还会引起资源竞争，那么最好的办法就是将任务队列化，按照跳转最少，资源竞争最少的原则进行排序。
2.在任务队列化的基础上，map/reduce

// todo 测试


## reference
> [浅析MySQL事务中的redo与undo](https://www.jianshu.com/p/20e10ed721d0)
> 