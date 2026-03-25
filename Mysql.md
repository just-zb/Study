MySQL 在「可重复读」隔离级别下，可以很大程度上避免幻读现象的发生,为什么,以及哪些不能避免?

> 因为快照读使用mvcc 一个事务只能读到第一次读的时候读到的快照.,当前读:使用next-key锁,锁定区间所以避免了幻读.
>



Read View的字段作用,聚簇索引中的两个隐藏字段?

> Read View为每次执行快照读时记录数据库事务状态,可以根据Read View判断数据页的快照对当前事务是否可见.隐藏字段分别是修改数据页的事务ID以及回滚指针roll_ptr

mysql执行delete之后,数据会立刻删除吗? 

> 不会,因为删除操作开启日志,被删除的record会打上delete tag,由后台线程purge进行删除.

buffer pool的作用?

> buffer pool提升读写效率,会用来存储缓存页,自适应哈希索引,undo日志等. 缓存页负责与磁盘中的页进行沟通,最小单位是页. 查到一条记录,也会加载整个页.
>



## Mysql学习

怎么跟mysql交互?

![查询语句执行流程](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/sql%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B/mysql%E6%9F%A5%E8%AF%A2%E6%B5%81%E7%A8%8B.png)



sql过来了怎么处理? 分为server层和存储引擎,server是公用的,存储引擎有innodb,myisam,memo,用来具体存储数据.

myisam和innodb的区别?

> 事务,锁粒度,外键,日志支持,MVCC,聚簇索引,count(*)性能
>
> 聚簇索引: 数据和索引放在一起,而MyISAM只支持二级索引,叶子节点存放的是数据的物理地址.

如何选择存储引擎?

> 大多数情况下，使用默认的 InnoDB,MyISAM 适合读多写少的场景。MEMORY 适合临时表，数据量不大的情况。

观察innodb, 想一个表是怎么在引擎中存储的,引出索引与聚簇索引,表的存储与聚簇索引相关, 这种索引结构广为出现. 所以索引的种类?

为什么用B+树不用B树?

> B 树的每个节点既存储键值，又存储数据和指针，导致单节点存储的键值数量较少
>
> B+ 树的叶子节点通过双向链表顺序连接，范围查询只需定位起始点后顺序遍历链表即可
>
> B树查询时间波动较大,而且删除时容易引起频繁表结构变更.

思考到联合索引时,自然想到这种索引的结构与最左匹配,想一下最左匹配出现范围查询时,为什么>=能用到索引,>用不到. 联合索引中什么叫做索引下推? 把判断过程改成在联合索引进行而不是回表主键索引,是一种优化方式.

然后是什么时候需要索引,什么样的索引才好? 

查询一个sql可能用到索引可能用不到,如何避免索引失效?

> 比如索引列使用了函数、使用了通配符开头的模糊查询、联合索引不满足最左前缀原则，或者使用 or 的时候部分字段无索引等.使用 `!=` 或 `<>` 不等值查询会导致索引失效

索引的物理结构使用B+树,处于什么考虑

索引之所以重要是因为它跟经常用到的sql查询息息相关,能极大提高查询效率.



几个sql一起执行会相互影响,并且sql执行过程中如果数据库崩溃必须要恢复,数据表也要保证一致性. 所以引入事务概念. 事务的并发执行导致了哪些问题? 脏读,不可重复读,幻读, 定义事务隔离级别, 决定多大程度上事务能相互影响.

事务本身有ACID特性,保证事务之间互不影响. 如何保证这些特性? 原子性引入了undo log, 隔离性引入MVCC和锁,持久性引入redo log. 事务的特性需要日志支持,日志不像索引那样直观,而是在暗处提供保障. 日志还有binlog处于server层面. 主要为了数据库备份以及主从复制.

### 可重复读与不可重复读的区别

根据这两个级别的定义,引入MVCC的快照读和当前读. 可重复读只会在第一次读的时候生成一次快照Read View,而不可重复读每次都会生成.

select普通会归类到快照读,通过Read View实现,而当前读包含加锁的select(for update和LOCK IN SHARE MODE),update,delete,insert. 

快照读不加锁,每次读取某个可见版本,当前读会加锁,通过 next-key lock实现. 

为什么InnoDB 引擎的可重复读很大程度上避免了幻读? MVCC避免了幻读,通过快照读和当前读的实现方式

那么Read View是干嘛的? 如何实现的快照读? 

Read View的四个核心字段,以及 数据表(聚簇索引)中的两个隐藏字段trx_id和roll_ptr. 快照读必须能够获得不同的版本,通过roll_ptr指向不同的版本组成的链表, 这涉及到undo log记录每一个版本,并加入到roll_ptr指向的链表中. 这样就实现了不同的快照,快照读会读取对应的快照,通过比较Read View的字段和trx_id, 决定读那个快照.

什么是MVCC?

多版本并发控制，每次修改数据时，都会生成一个新的版本,它的快照读根本不加锁,所以不会阻塞.底层实现主要依赖于 Undo Log 和 Read View。

**通过「版本链」来控制并发事务访问同一个记录** 就是MVCC,

### 如何保证原子性

事务没提交后中断,需要进行回滚,用到undo 日志.undo log的主要功能就是事务回滚和MVCC,undo log记录回滚需要的信息. 每一次产生一个版本的undo log,都会在roll_pointer中加入一条链表节点.

对于undo log的刷盘,首先刷新到buffer pool,然后刷新到磁盘,会跟redo log进行结合,用redo log保证持久化

### 如何保证隔离性

通过MVCC和锁实现,

### 如何保证持久性

保证持久性用到redo log, 同时redo log还能提供对buffer pool和undo log的持久性支持. 我们知道对数据页的修改是修改在buffer pool缓存中,产生脏页.必须要及时进行刷盘. redo log实现了刷盘的部分功能,通过WAL,也就是刷盘前先记录日志,能够实现顺序记录redo log,随后后台进行进行脏页刷盘的异步操作. 把脏页的随机刷盘变成了redo log的顺序刷盘,

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/how_update/wal.png?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

redo log本质上是物理日志,事务提交就不用等待脏页刷盘,只需将redo log和binlog刷盘. undo log会写到buffer pool中undo log页面,也要利用redo log. redo log记录的是更新之后的值,而undo log记录持久前的状态.

redo log会直接写到磁盘吗? 不会,在特定时机会进行刷盘, 这里有个redo log的刷盘策略.



在实现了事务之后,我们知道MVCC使用了锁,对于特殊的情况如修改表结构会加表锁,数据库备份加全局锁.

所以数据库中的锁是帮助实现某些机制,分为全局锁,表锁和行锁.

全局锁在全数据库逻辑备份中用到,让数据库处于只读.备份数据用到mysqldump,加上 `–single-transaction` 参数能够开启事务,因为MVCC会创建快照,所以在备份过程中可以继续执行事务.

表锁是一个术语,表示修改表的可读和可写状态,常见于MyISAM,可以加读锁和写锁.读锁允许多个事务读表,不允许写.排他锁只允许一个事务进行写操作，其他事务不能读也不能写。

更改表结构时加MDL元数据锁,CRUD加MDL读锁,对表结构变更时,加MDL写锁.

在对表中一行记录加共享锁和独占锁时,会分别加表级意向共享锁和表级意向独占锁,目的是**提高加表锁时的判断效率**。这俩种锁不会跟行级锁冲突,这两个意向锁之间也不会冲突,加行级S锁之前，先加表级**IS锁**（意向共享锁）加行级X锁之前，先加表级**IX锁**（意向排他锁）

意向锁的本质是一个表级别的"快速标记"，让加表锁的操作不需要逐行检查是否存在行锁，把冲突检测的复杂度从O(N)降到O(1)。它只在行锁和表锁共存的场景下才有意义. 如果有意向写锁,就不能加表级读锁,有意向读锁可以加表级读锁.

行级锁为记录锁,间隙锁,和next-key lock, 记录锁分为行级读锁,写锁,记录锁+gap lock就是next-key lock,它会锁定一个范围,并且锁定记录. 所以当前读实现了next-key lock

间隙锁锁定一个区间,而插入意向锁会在试图在间隙锁范围内插入时出现,并且处于阻塞等待状态,插入意向锁锁住一个点, 间隙锁释放之后才会获得插入意向锁.



讨论完了锁和事务,来看看server层的binlog日志,感觉它比较孤单. 事务执行过程中,先写binlog cache,事务提交之后才会把cache写入binlog,一个线程同时只能执行一个事务,所以一个事务的binlog不能拆开,如果拆开在从库里就会识别为两个事务.

看看binlog的特点,首先是server层的,任何存储引擎都可以使用,它分为三种格式: statement(逻辑日志),row(记录数据行最终修改结果),mixed(包含statement和row). binlog是追加写,保存全量日志. 相比之下redo log就比较单一,功能就是为了保证掉电恢复的持久性,并且是循环写的物理日志.

保证binlog与redo log的一致性, 用到两阶段提交,因为他们是独立的逻辑,半成功会导致逻辑不一致. 两阶段分为准备和提交阶段.

![两阶段提交](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/how_update/%E4%B8%A4%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A4.drawio.png?image_process=watermark,text_5YWs5LyX5Y-377ya5bCP5p6XY29kaW5n,type_ZnpsdHpoaw,x_10,y_10,g_se,size_20,color_0000CD,t_70,fill_0)

mysql重启后扫描redo log,发现状态为prepare,然后拿到xid,到binlog里,看binlog有无xid,事务是否提交就看binlog是否有xid



## 高可用

为了保障高可用,会进行读写分离和主从复制,以及分库分表.

主从复制的过程:

主库执行事务提交时，将数据变更以事件形式记录到 Binlog,从库通过 I/O 线程从主库的 Binlog 中读取变更事件，并将这些事件写入到本地的中继日志文件中.SQL 线程会实时监控中继日志的内容，按顺序读取并执行这些事件，从而保证从库与主库数据一致。

主库在提交事务写入binlog之后,会把binlog同步到所有从库,每个从库把binlog写入到暂存日志delaylog中,然后从库回放binlog,更新innodb数据. 从库会创建IO线程链接主库log dump线程,接收主库的binlog存放到暂存日志,然后再用一个线程回放binlog的数据.

为了避免网络导致的主从同步延迟解决:

对一致性要求高的查询（如支付结果查询）可以直接走主库。非关键业务允许短暂数据不一致，可以提示用户“数据同步中，请稍后刷新”，然后借助异步通知机制替代实时查询。

采用半同步复制，主库在事务提交时，要等至少一个从库确认收到 binlog（但不要求执行完成），才算提交成功。

怎么分库?

垂直分库：按照业务模块将不同的表拆分到不同的库中，比如说用户、登录、权限等表放在用户库中，商品、分类、库存放在商品库中

水平分库：按照一定的策略将一个表中的数据拆分到多个库中，比如哈希分片和范围分片，对用户 id 进行取模运算或者范围划分，将数据分散到不同的库中。

怎么分表?

也分为水平拆分和垂直拆分

如何避免分库分表带来的问题:

跨库事务无法依赖单机 MySQL 的 ACID 特性.用分布式事务.

跨库后无法使用 JOIN 联表查询。可以在业务层进行拼接

自增 ID 在分片场景下容易冲突，需要使用全局唯一方案。数据库表被切分后，不能再依赖数据库自身的主键生成机制,保证主键全局唯一用雪花算法.

雪花算法怎么实现?

使用一个 64 位的数字来作为全局唯一 ID。符号位+时间戳+工作机器ID+12 位的序列号.

### 数据表与sql语句

sql语句首先能写对,然后对慢sql进行优化.

sql涉及到数据库表设计,用到的范式理论, 第一范式原子性，确保表的每一列都是不可分割的基本数据单元,后面的两个范式讲的是表的普通字段与主键的关系,第二范式消除部分依赖:每一个列都跟主键直接相关,非主键字段必须完全依赖于整个主键,第三范式消除传递依赖:非主键列应该只依赖于主键列,非主键字段不能依赖于另一个非主键字段

第二范式（2NF）解决的是部分依赖问题：在满足1NF的基础上，要求所有非主属性必须完全依赖于整个主键，不能只依赖主键的一部分。这个问题只在联合主键的表中才会出现。第三范式（3NF）解决的是传递依赖问题：在满足2NF的基础上，要求非主属性之间不能存在依赖关系，即非主属性必须直接依赖于主键，不能通过另一个非主属性间接依赖。

表定义有很多字段类型,讨论下他们. 

varchar与char的区别, char(10)固定长度10,不够的用空格填充. blob和text区别,blob存储二进制文件,text存储文本数据, 

datetime和timestamp的区别.DATETIME 直接存储日期和时间的完整值，与时区无关。 TIMESTAMP存储1970-01-01 00:00:01 UTC 以来的秒数,为unix时间戳,受到时区影响. datetime默认值null,timestamp默认值为current_timestamp

in和exists, in适合结果集小的情况,exists适合结果集大,因为exists只需要判断只有一个行存在

记录货币用DECIMAL(19,4)更好,不能用浮点数存在误差.

存储emoji需要使用 utf8mb4 字符集,因为emoji是4字节的utf-8字符,MySQL 的 utf8 字符集只支持最多 3 个字节的 UTF-8 字符.

drop、delete 与 truncate 的区别？ DROP 是物理删除，用来删除整张表，包括表结构，且不能回滚。DELETE 支持行级删除，可以带 WHERE 条件，可以回滚。TRUNCATE 用于清空表中的所有数据，但会保留表结构，不能回滚。

count(1),count(*),count(列名)区别,count(1)被优化为count(\*),count(\*)会走索引加速计算. count(列名)会剔除值为null的行.

sql执行顺序?

使用 limit 语句，结合偏移量和行数来实现。查询表第3-10行数据.

一些mysql函数: 

- `CONCAT()`: 用于连接两个或多个字符串。
- `LENGTH()`: 用于返回字符串的长度。
- `SUBSTRING()`: 从字符串中提取子字符串。
- `REPLACE()`: 替换字符串中的某部分。
- `TRIM()`: 去除字符串两侧的空格或其他指定字符。
- `ABS()`: 返回一个数的绝对值。
- `ROUND()`: 四舍五入到指定的小数位数。
- `MOD()`: 返回除法操作的余数。
- `NOW()`: 返回当前的日期和时间。
- `CURDATE()`: 返回当前的日期。
- `SUM()`: 计算数值列的总和。
- `AVG()`: 计算数值列的平均值。
- `COUNT()`: 计算某列的行数。
- `IF()`: 如果条件为真，则返回一个值；否则返回另一个值。
- `CASE`: 根据一系列条件返回值。

```sql
-- IF函数
SELECT IF(1 > 0, 'True', 'False') AS simple_if;

-- CASE表达式
SELECT CASE WHEN 1 > 0 THEN 'True' ELSE 'False' END AS case_expression;
```

一些sql隐式类型转换:SELECT 1 + 1.0; -- 结果为 2.0 . 当一个字符串和一个整数相加时，字符串会被转换为整数。



SELECT中的各列在逻辑上是同时计算的，不存在先后顺序,所以**同一个SELECT里的列不能引用同层定义的别名**。

`DISTINCT` 是修饰整个 `SELECT` 的，不是修饰某一列。它对**所有选出的列的组合**去重。`DISTINCT` 要么跟在 `SELECT` 后面对整行去重，要么放在聚合函数括号里对单列去重。



一条update语句的执行过程:

如果当前没有显式事务，InnoDB会自动开启一个事务（autocommit=1时隐式事务），分配事务ID. 然后加锁,update会加排他锁,是主键则加record锁,如果走的不是唯一索引或者是范围查询,则加上临键锁,防止幻读.

读取旧数据到buffer pool,然后记录undo log.更新buffer pool,同时更新隐藏列 `trx_id` 为当前事务ID。此时这个数据页变成了**脏页**.

把这次修改的物理变更写入 **Redo Log Buffer**，然后刷入磁盘的redo log文件，状态标记为 **prepare**。如果MySQL宕机，重启后可以通过redo log把已提交但还没刷盘的脏页数据恢复。这就是WAL（Write-Ahead Logging）机制——先写日志再写数据。

如果数据库开启了binlog,执行器将这条UPDATE语句写入 **Binlog Buffer**，然后刷入磁盘的binlog文件。

Binlog写成功后，再把redo log的状态从prepare改为 **commit**。这就是**两阶段提交**，目的是保证redo log和binlog的一致性

事务提交后,释放锁,然后对脏页异步刷盘.

事务开始前,记录undo log,然后将更新操作写入redo log,状态标记为prepare,开启事务



sql优化部分: 什么是慢sql?

long_query_time 的参数,执行时间超过这个参数就是慢sql,记录到慢查询日志中. 优化慢sql: 先找到那些慢sql,可以开启慢查询日志, 设置 slow_query_log 参数为 1.然后使用 EXPLAIN 查看慢 SQL 的执行计划,检查是否用索引.

优化慢sql方式?

先EXPLAIN 查看慢 SQL 的执行计划.

根据分析结果，通过索引优化、sql优化,表结构优化,和架构等方式进行优化。

架构:**读写分离**，读请求打到从库，降低主库压力。热点数据加**Redis缓存**，减少数据库访问

如何进行分页优化？避免深度偏移带来的全表扫描,如 LIMIT 1000,20. 分页会导致变慢,由于 OFFSET 的存在，OFFSET 会导致 MySQL 必须扫描和跳过 offset + limit 条数据，这个过程是非常耗时的。

用延迟关联和添加书签.

```sql
SELECT e.id, e.name, d.details
FROM (
    SELECT id
    FROM employees
    ORDER BY id
    LIMIT 1000, 20
) AS sub # 延迟关联,先获取id,适用于从多个表中获取数据且主表行数较多的情况
JOIN employees e ON sub.id = e.id
JOIN department d ON e.department_id = d.id;
# 添加书签
SELECT id, name
FROM users
WHERE id > last_max_id  -- 假设last_max_id是上一页最后一行的ID
ORDER BY id
LIMIT 20;
```





![沉默王二：SQL 优化](https://cdn.paicoding.com/stutymore/mysql-20240327104050.png)

优化子查询指的是用join代替子查询,JOIN 的 ON 条件能更直接地触发索引，而子查询可能因嵌套导致索引失效。JOIN 的一次连接操作替代了子查询的多次重复执行，尤其在大数据量的情况下性能差异明显

子查询中的标量子查询是什么?

出现在SELECT列表中，语义上要求子查询**必须返回一行一列**（或者零行，零行时返回NULL）,如果返回多行会直接报错.

```sql
SELECT o.order_id, o.amount, 
       (SELECT c.name # 标量,必须返回一行一列
        FROM customers c 
        WHERE c.customer_id = o.customer_id) AS customer_name
FROM orders o;
```

在这个sql中,外层表的每一行orders都会触发一次子查询,而join会把两个表进行关联,连接操作为一次执行,并且可以通过索引快速关联数据



为什么小表驱动大表?

- 如果大表的 JOIN 字段有索引，那么小表的每一行都可以通过索引快速匹配大表。时间复杂度为小表行数 N 乘以大表索引查找复杂度 log(大表行数 M)，总复杂度为 N*log(M)。所以小表的时间复杂度更低.
- 如果大表没有索引,实际上小表驱动仍然更好，原因在于磁盘IO和Buffer Pool的利用效率. 假设小表A有100行，大表B有10000行。小表驱动大表时，大表被反复扫描100次。但因为MySQL有**Join Buffer**（Block Nested Loop Join），它会把驱动表的数据批量加载到内存的Join Buffer中，然后扫描一次被驱动表就和Buffer里的多行同时匹配。小表100行可能一次就全部装进Join Buffer，这样大表实际上**只需要全表扫描一次**。大表驱动小表时，需要把10000行装进Join Buffer，如果装不下就要分多批，每一批都要对小表做一次全表扫描。虽然小表本身不大，但**批次数更多，总的扫描次数更多**。
  - 假设Join Buffer能装500行,小表驱动：100行 ÷ 500 = 1批 → 大表全表扫描 1次,大表驱动：10000行 ÷ 500 = 20批 → 小表全表扫描 20次
  - 小表驱动大表的本质优势不仅仅是索引，而是**让驱动表尽量一次装进Join Buffer，减少被驱动表的扫描次数**。
- 如果大表没有索引，需要将小表的数据加载到内存，再全表扫描大表进行匹配。

- 当使用 join 时，MySQL 会选择数据量比较小的表作为驱动表，大表作为被驱动表。left join时,左表为驱动表. 让小表做驱动表.

为什么要避免使用 JOIN 关联太多的表？

多表 JOIN 需要缓存中间结果集，可能超出 join_buffer_size，这种情况下内存临时表就会转为磁盘临时表，性能也会急剧下降。

如何进行排序优化？

对 ORDER BY 涉及的字段创建索引，避免 filesort。如果是多个字段，联合索引需要保证 ORDER BY 的列是索引的最左前缀。

调整排序参数,让排序在内存中完成.

分析explain的执行.查看 SQL 执行计划.最关注的字段是 type、key、rows 和 Extra。它们判断 SQL 有没有走索引、是否全表扫描、预估扫描行数是否太大，以及是否触发了 filesort 或临时表. 

key如果为 PRIMARY,表示主键索引

type={const(只找到一行),range(索引范围扫描),eq_ref(对主键join匹配),index(全索引扫描),ALL}如果 type=ALL 表示全表扫描,或者 Extra=Using filesort表示排序的字段不是索引,则需要进行优化. 

extra取值三个: index,temporary table,filesort 如果temporary table超过了内存Memory引擎大小则会导致sql显著变慢.

什么是filesort?

当不能使用索引生成排序结果的时候，MySQL 需要自己进行排序，如果数据量比较小，会在内存中进行；如果数据量比较大就需要写临时文件到磁盘再排序，我们将这个过程称为文件排序。

为什么避免使用<>和!=?

因为`!=` 或者 `<>` 操作符会导致 MySQL 无法使用索引，从而导致全表扫描。可以把`column<>'aaa'`，改成`column>'aaa' or column<'aaa'`。



面试过程中还要写sql语句. 记录一下可能的题:

可以做牛客的sql题https://www.nowcoder.com/exam/oj?page=1&tab=SQL%E7%AF%87&topicId=82

claude给出的常考面试题:

- 多表关联是高频考点，常见场景包括：查员工和部门的对应关系、找没有下过订单的用户（LEFT JOIN + IS NULL）、自连接找同部门的同事等。面试官会看你对INNER/LEFT/RIGHT JOIN的理解是否清晰

- 聚合与分组: GROUP BY + 聚合函数（COUNT、SUM、AVG、MAX、MIN）配合HAVING做条件过滤，

- 窗口函数:ROW_NUMBER、RANK、DENSE_RANK做分组排名是最常见的，比如"找每个部门工资最高的前3名员工"。 另外还有求同比和环比时,用到LAG()和LEAD(), 窗口函数一般在select中使用,可以在order by中出现.

- 经典场景题: 连续N天登录（ROW_NUMBER差值法）、第N高的工资（DENSE_RANK或子查询LIMIT）、删除重复数据只保留一条（DELETE + 子查询）、行列转换（CASE WHEN + 聚合）。

  - 第N高的工资,用窗口函数

  - 连续N天登录: 如果登录日期是连续的，那么 `日期 - ROW_NUMBER` 得到的值相同。 见https://www.nowcoder.com/practice/63ac3be0e4b44cce8dd2619d2236c3bf

    ```sql
    with a as(
        select user_id, sales_date, 
        row_number() over(partition by user_id order by sales_date) as rn
        from sales_tb
    )
    select user_id, count(*) days_count
    from a
    group by user_id, date_add(sales_date,INTERVAL -rn day) # 连续的天数,date_add(sales_date,INTERVAL -rn day)相同
    having count(*) >=2
    ```

    

  - 删除重复数据只保留一条:MySQL中DELETE的子查询不能直接引用自身表，所以需要多包一层子查询（那个 `t`），这是个常见的坑，面试爱考。

    ```sql
    DELETE FROM employees
    WHERE id NOT IN (
        SELECT id FROM (
            SELECT MIN(id) AS id
            FROM employees
            GROUP BY emp_no, dept_no
        ) t
    );
    ```

  - 行列转换（行转列）套路就是 `CASE WHEN` 按条件拆列，外面包聚合函数

  - 列转行: 多次select,然后把select的结果union all

