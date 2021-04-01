# mysql锁与事务

## 表锁
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

## 显式、隐式锁
可以用`select * from information_schema.innodb_locks`查询到的是显式锁,例如`select ... for update`或者`select ... lock in share mode`这种用户手动加上的就是显式锁;

隐式锁是mysql程序逻辑设计里应该加锁的部分,当其他事务访问到加锁行时,其他事务会将隐式锁转换称显式锁,并将自己放置到等待队列里面;

## 加锁详细场景分析
### 前置内容
表结构:
```sql
CREATE TABLE `test` (
  `Id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `ExtraId` int(11) DEFAULT NULL COMMENT '外键id,唯一索引',
  `TiId` int(11) DEFAULT NULL COMMENT '外键id,普通索引',
  `Name` varchar(255) DEFAULT NULL,
  `Age` tinyint(4) DEFAULT NULL,
  PRIMARY KEY (`Id`),
  UNIQUE KEY `ExtraId` (`ExtraId`) USING BTREE,
  KEY `TId_index` (`TiId`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

查询锁:
```sql
select * from information_schema.innodb_trx;
select * from information_schema.innodb_locks;
select * from information_schema.innodb_lock_waits;
```

对于`innodb_locks`,查询出来的字段解释:
|列名|	描述|
|---|---|
|LOCK_ID|	一个唯一的锁ID号，内部为 InnoDB。|
|LOCK_TRX_ID|	持有锁的交易的ID|
|LOCK_MODE|	如何请求锁定。允许锁定模式描述符 S，X， IS，IX， GAP，AUTO_INC，和 UNKNOWN。锁定模式描述符可以组合使用以识别特定的锁定模式。|
|LOCK_TYPE|	锁的类型|
|LOCK_TABLE|	已锁定或包含锁定记录的表的名称|
|LOCK_INDEX|	索引的名称，如果LOCK_TYPE是 RECORD; 否则NULL|
|LOCK_SPACE|	锁定记录的表空间ID，如果 LOCK_TYPE是RECORD; 否则NULL|
|LOCK_PAGE|	锁定记录的页码，如果 LOCK_TYPE是RECORD; 否则NULL。|
|LOCK_REC|	页面内锁定记录的堆号，如果 LOCK_TYPE是RECORD; 否则NULL。|
|LOCK_DATA|	与锁相关的数据（如果有）。如果 LOCK_TYPE是RECORD，是锁定的记录的主键值，否则NULL。此列包含锁定行中主键列的值，格式为有效的SQL字符串。如果没有主键，LOCK_DATA则是唯一的InnoDB内部行ID号。如果对键值或范围高于索引中的最大值的间隙锁定，则LOCK_DATA 报告supremum pseudo-record。当包含锁定记录的页面不在缓冲池中时（如果在保持锁定时将其分页到磁盘），InnoDB不从磁盘获取页面，以避免不必要的磁盘操作。相反， LOCK_DATA设置为 NULL。|


`set autocommit=0`关闭自动提交, 相当于开启一次事务;

### 测试流程:
测试方法, `set autocommit=0`关闭自动提交, 并执行一段update语句,不commit:
```sql
set autocommit=0;
update test set name='郭小宝' where age=24;
-- commit; 不提交
```

测试session也需要关闭自动提交或者开启事务;

#### update where不走索引
`UPDATE test SET `Name`='周杰伦' WHERE age=43;`

1. session2,`UPDATE test SET `Name`='周杰伦2' WHERE age=43;`; 可以看到在主键上加了X锁;
2. session2, `INSERT INTO test VALUES(null, 7, 3, "陈克明", 78);`: lock_data是supremum pseudo-record,说明触发了间隙锁;


结论: Repeatable Read+无索引，会对所有数据的主键加X锁，并且在记录的缝隙之间加GAP锁防止新记录插入来解决幻读。


#### update where走普通索引
`UPDATE test SET `Name`="周杰伦" WHERE TiId=2;`

1. session2, `UPDATE test SET `Name`="周杰伦2" WHERE TiId=2;`; 在TiId列加了X锁; lock data是5、2(主键id、索引列);
2. session2根据主键更新, `UPDATE test SET `Name`="周杰伦2" WHERE Id=5;`: 在主键上加了X锁, lockdata是主键id;
3. session2在TiId`(2, 5]`这个范围内插入新记录: `INSERT INTO test VALUES(null, 7, 3, "陈明", 71);`; 可以看到在TiId索引上加了X、GAP锁;

结论：对满足条件的记录加X锁+GAP锁，对应的主键加X锁。防止幻读。

#### 走唯一索引
唯一索引只有主键-唯一索引的X锁、没有GAP锁;

#### 走主键索引
仅对主键加X锁;


### 结论
<img src="https://upload-images.jianshu.io/upload_images/14137179-89d1da35730fb6fd?imageMogr2/auto-orient/strip|imageView2/2/format/webp" width="700px" >



## 死锁
1. session1对A加锁, session2对B加锁; 然后session1更新B,session2更新A, 产生死锁;
2. 多个并发操作一个唯一索引,也会导致死锁;

mysql有锁等待超时时间,默认50s;

50s超时回滚，这种机制缺点不适合并发，对于发生死锁等待50S太久。设置的太短也不行可能有的事务真要50S，所以就有了第二种死锁处理机制。

等待图,使用持有锁列表和等待锁列表,由等待事务指向持有事务，组成的图。采用深度优先遍历算法如果发线回路则说明发生死锁。对比权重和加锁时间，回滚权重小和更晚持有锁的事物。


### 如何避免死锁
1. 尽可能小事务;
2. 多张表更新，不同业务对表操作顺序保持一致;
3. 唯一索引异常尽快回滚事务;
4. 根据自身业务设置等待锁超时时间，默认50S;
5. 批量分批提交;
6. 若业务可以接受幻读，则使用Read Commited的隔离级别;
7. 更新删除全部使用主键，否则高并发下容易产生死锁，因为更新一笔数据要对库、表、页、索引加多个锁。交叉下并发下比较容易产生死锁。


