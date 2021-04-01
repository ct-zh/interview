# mysql知识体系与常见问题粗谈

dml
ddl

行扫描为什么比只读索引扫描要快呢？

OLAP/OLTP



## 回表
1. innodb的主键索引是聚簇索引,使用B+树实现的,在叶子结点上保存了整行的数据;
2. 普通索引也是B树的实现, 不同的是叶子结点上保存的是主键的值;
3. 那么走普通索引查到的自然是主键的值,还需要去主键索引查到记录;这个操作就叫回表.

## 索引覆盖(Covering index)
> 只需要在一棵索引树上就能获取SQL所需的所有列数据，无需回表，速度更快。
1. 见sql:`select id,username from user where username="zhangsan"`, 查询字段为`id`与`username`,不包含其他字段;
2. 那么给`username`字段建立索引,对于`username`索引树,可以查到主键的值;
3. 因为获取的内容只有`username`和`id`,此时就不需要回表再去查聚簇索引了,这就叫*索引覆盖*;

### 那如果我要查其他字段呢
1. 见sql`EXPLAIN SELECT id,username,sex from user_test WHERE username="kangkang"`,因为多了sex字段,肯定不会触发索引覆盖了;
2. 将`username`索引升级成联合索引`index(username, sex)`, 此时又符合索引覆盖了;

// todo 联合索引的原理

