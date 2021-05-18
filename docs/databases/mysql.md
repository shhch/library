# mysql

## mysql存储引擎

+ InnoDB 支持事务，支持行级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引；<br>
+ MyISAM 不支持事务，支持表级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引；<br>
+ Memory 不支持事务，支持表级别锁定，支持 B-tree、Hash 等索引，不支持 Full-text 索引；<br>
+ NDB 支持事务，支持行级别锁定，支持 Hash 索引，不支持 B-tree、Full-text 等索引；<br>
+ Archive 不支持事务，支持表级别锁定，不支持 B-tree、Hash、Full-text 等索引；<br>

## 常用命令

+ 创建索引：create (index_type) index index_name on table_name<br>
+ 加表锁：LOCK TABLE tablename locktype, tablename2 locktype<br>
+ locktype：READ只能读表, read LOCAL, write, low priority write<br>
+ 解锁：UNLOCK TABLE[S],解锁不需要表名，会解除当前用户所有的表锁<br>
+ 判断查询语句是否使用到了索引（explain）： 在语句前面加上explain，查询结果就为explain的相关信息；返回结果字段如下：
    + id：id标识符，如果有join或者子查询，那么是多个，按照查询顺序来，id越大执行的优先级越高，相同就从上往下执行
    + select_type：查找类型
        + simple简单查询
        + primary主要的
        + union联合
        + dependent union依赖联合
        + union result联合结果
        + subquery子查询
        + dependent subquery联合子查询
        + derived派生
        + uncacheable subquery无法缓存的
    + table：查找的表名，如果用了简称，则显示简称
    + type：表示表的连接类型，性能重要指标，从上到下，性能由低到高；通常要保证查询的type至少为range，最好达到ref
        + all：full table scan，全盘扫描
        + index：full index scan，扫描全部索引
        + range：检索给定范围内的行，使用索引选择行
        + ref：表示上述表的连接查询条件，既哪些列和常量用于查找索引上的值
        + eq_ref：类似ref，区别就是使用唯一索引，例如unique key或者primary key作为关联条件
        + const、system：当mysql对某部分查询进行优化，并转化为一个常量时，为此类型，例如，where中使用主键；system为const的特例，查询的表只有一条数据时，为system
        + NULL：mysql查询时优化语句，查询时甚至不用查询表或者访问索引
    + possible_keys：查询时可能使用的索引
    + key：查询时具体使用的key，可通过`use index`或者`force index`或者`ignore index`来强制使用或者忽略某些索引
    + key_len：使用到的索引的长度，不损失精度的情况下，越短越好
    + ref：列与索引的比较
    + rows：mysql认为获得结果需要扫描的行数
    + extra：执行情况的描述和说明，避免文件排序和临时表
        + using where：不读取所有信息，仅通过索引就可以获取所需数据
        + using temporary：使用临时表，比如group by，order by
        + using filesort：使用文件排序，order by时，无法使用索引来完成
        + using join buffer：强调在获取连接查询条件时没有使用索引，而使用了缓存，意味着这里可能需要加索引了
        + impossible where：不存在的where，强调了where会导致没有符合条件的行
        + select table optimized away：仅通过使用索引，优化器可能仅从聚合函数结果中返回一行
        + no table used：使用from dual 或不含任何from子句

## 索引
索引正常而言是优化查询速度的利器，但是并不是越多越好，索引多了，需要存储的索引文件就多，同时在插入时，需要更新的索引文件变多，影响插入性能，在查询时，需要读取更多的索引文件，需要更多的IO，同样影响性能；

+ 联合索引

+ 覆盖索引与回表查询
通过普通索引查询到主键id（聚集索引的键），然后通过id去聚集索引查询，这种叫做回表查询，先定位主键，然后再定位id，要两遍，性能较低；

而要达到不回表查询，则要建立覆盖索引，那就是联合索引

## 数据库连接池的连接数

通用公式：cpu * 2 + 有效磁盘数（ssd， hhd或其他）

## 行锁与表锁

通过lock table table_name即可加表锁；
对于UPDATE、DELETE和INSERT语句，InnoDB会自动给对应数据集加排它锁；或者显示的加行锁在语句后加`for update`；

但是行锁会在批量修改时，变成表锁，这是因为InnoDB的行锁是针对索引加的锁，不是针对记录，同时如果索引失效，行锁也会变表锁；
    

## mysql性能调优

1. 拆分复杂查询，是一个大的复杂查询还是多个简单查询好；
2. 分解关联查询，将join的on拆解为多个表的简单where查询，然后组合；优点：让缓存效率更高；查询分解后，单个查询可以减少锁的竞争；
3. 排序，尽量通过索引进行排序，要避免对大量数据进行排序，数据量大，则需要使用磁盘，否则会使用内存；limit时尽量保证使用索引
4. 子查询，union的时候如果需要排序或者limit，那么在union的各个子查询里面使用；子查询尽量使用关联查询代替；
5. 使用辅助索引时，尽量减少范围查询的范围；尽可能选择唯一性较强的字段建立索引
    

## MyISAM与InnoDB的区别

1. InnoDB支持事物ACID(Atomicity原子性、Isolation隔离性、Consistency一致性、Durability持久性;事务并发会引起脏读（读到未提交的数据，如果回滚或再次修改提交，则读到的是假数据）、不可重复读（A读，B改，A在读，不一样）、幻读（A读没读到，B加，A在读，读到了，删除也是一样）问题;事务隔离级别，读未提交read uncommitted（均不能避免），读提交read committed（避免脏读），可重复读repeatable read（默认，避免脏读和不可重复读），串行序列化serializable（均可阻止）)，而MyISAM不支持事物；mysql默认是采用auto coomit自动提交的，不手动的开启事务，那么每个sql都会当做一个事务提交，
2. InnoDB支持行级锁，而MyISAM只支持表级锁
3. InnoDB支持MVCC(多版本控制), 而MyISAM不支持
4. InnoDB支持外键，而MyISAM不支持
5. InnoDB不支持全文索引，而MyISAM支持

2者selectcount(*)哪个更快？<br>
myisam更快，因为myisam内部维护了一个计数器，可以直接调取。

mysql的日志：错误日志、查询日志、慢查询日志、binlog、中继日志（也是二进制日志）、事务日志（redo和undo）
    

## InnoDB的MVCC多版本控制原理与流程

什么是MVCC？MVCC的意思用简单的话讲就是对数据库的任何修改的提交都不会直接覆盖之前的数据，而是产生一个新的版本与老版本共存，使得读取时可以完全不加锁。这样读某一个数据时，事务可以根据隔离级别选择要读取哪个版本的数据。过程中完全不需要加锁。

在高并发的场景中，要保证数据的正确性，一般进行加锁处理，而大量使用的锁带来了很差的性能，MVCC在大多数场景下，可以代替行级锁，降低锁所带来的性能开销；

在InnoDB中，支持行级锁，而MyISAM只支持表锁，（Mysql的）InnoDB中默认的隔离级别是可重复度repeatable read，他要求同一时间不同事物之间不能相互影响，同时还能够并发处理，使用悲观锁肯定是无法实现的，repeatable read采用的就是乐观锁，而这个乐观锁的实现就是MVCC，MVCC主要使用场景为RR（repeatable read）和RC（read commit）两种隔离级别中；
Read Committed：一个事务读取数据时总是读这个数据最近一次被commit的版本
Repeatable Read：一个事务读取数据时总是读取当前事务开始之前最后一次被commit的版本（所以底层实现时需要比较当前事务和数据被commit的版本号）。

MVCC流程：
1. 事务开启，获得一个事务版本号，有mysql分发，自增
2. 获得一个read view
3. 查询到数据，进行read view事务版本号进行匹配
4. 不符合read view规则的，则从undo log中获取历史版本数据
5. 返回符合规则的数据

他有以下几个核心：
1. 事务版本号；
2. 表的隐藏列；
3. undo log；
4. read view；

+ 表隐藏列：<br>
    DB_TRX_ID: 记录操作该数据事务的事务ID；<br>
    DB_ROLL_PTR：指向上一个版本数据在undo log 里的位置指针；<br>
    DB_ROW_ID: 隐藏ID ，当创建表没有合适的索引作为聚集索引时，会用该隐藏ID创建聚集索引;

+ undo log:<br>
    undo log 主要用于记录数据被修改之前的日志，在表信息修改之前先会把数据拷贝到undo log 里，当事务进行回滚时可以通过undo log里的日志进行数据还原。用于MVCC快照读的数据，在MVCC多版本控制中，通过读取undo log的历史版本数据可以实现不同事务版本号都拥有自己独立的快照数据版本;

+ read view:<br>
    在innodb 中每个SQL语句执行前都会得到一个read_view。副本主要保存了当前数据库系统中正处于活跃（没有commit）的事务的ID号，其实简单的说这个副本中保存的是系统中当前不应该被本事务看到的其他事务id列表；<br>
    
    他的四个属性与匹配规则：
    + trx_ids: 当前系统活跃(未提交)事务版本号集合。
    + low_limit_id: 创建当前read view 时“当前系统最大事务版本号##1“；当前事务id<up_limit_id时显示；如果满足，可以肯定该数据是在当前事务启之前就已经存在了的,所以可以显示。
    + up_limit_id: 创建当前read view 时“系统正处于活跃事务最小版本号”；数据事务ID>low_limit_id则不显示；如果满足，则说明该数据是在当前read view创建之后才产生的，所以数据不予显示;
    + creator_trx_id: 创建当前read view的事务版本号；
    当处于最大和最小之间时，这个数据有可能是在当前事务开始的时候还没有提交的；这时需结合trx_ids进行匹配；<br>1.如果事务ID不存在于trx_ids 集合（则说明read view产生的时候事务已经commit了），这种情况数据则可以显示；<br>2.如果当前事务id存在集合中，且等于creator_trx_id，说明是自己产生的，肯定就可见了；否则产生rw的时候，数据没提交，也不是自己的，就不可见；不满足，则从undo log取历史数据，然后数据历史版本事务号回头再来和read view 条件匹配 ，直到找到一条满足条件的历史数据，或者找不到则返回空结果；

mysql读取时分两种：
快照读，读undo log，无读写时加锁的问题；当前读，读当前数据库最新的数据，很明显需要加锁；
    

## InnoDB的存储结构
  
InnoDB分为聚集索引和辅助索引；他有页的概念，页管理磁盘的一些块；每张表只能有一个聚合索引，它使用此结构来实现的：

BTree是一种平衡树，他的每个节点包含多个kv组，k为键，v为值；索引定义一个大于1的值为他的'度'，h为它的高度，每个非叶子节点由n-1个k与n个指针，其中n在[d,2d]之间；每个叶子节点至少包含一个k与两个指针，指针均为null；所有的叶子节点高度相同，为h；k与指针相互间隔，两端为指针；树满足中序遍历，节点内，k递增；
新插入的数据会影响当前树的结构，所以会涉及分裂、合并、转移等操作；

B+Tree为BTree的优化变种，变化为非叶子节点不存储v，只存储k；每个叶子节点都有指向下一个和上一个相邻节点的指针；通常在B+Tree上有两个头指针，一个指向根节点，一个指向k最小的叶子节点，这样可以方便主键的分页、范围查找、或者从根节点的随机查找；

辅助索引也是用的B+Tree，但是他不存储整行的数据，只存储哪个字段的，但是他有一个标签bookmark字段，会存储当前行所在的页的位置；
  
## 分表分库设计

中大型项目中，难免会遇到数据量比较大的情况，这个时候就需要对数据库、表进行拆分，提升性能，增加可用性；
分库与分表解决的是不同的问题：
+ 分库：数据库读写QPS过高，连接数不足，需要分库分担连接压力；
+ 分表：mysql单表推荐的数量级在百万级别，太大会导致性能下降；

一般分为垂直和水平两种；
1. 垂直切分：一般是按照业务，将经常变更的字段与不变更的字段进行拆分，保证不变更的字段的查询效率；
2. 水平切分：把数据按照一定的规则划分到对应的数据库、表中；

问题主要存在于水平切分，水平切分后，数据划分到不同的表中，需要保证热点数据的分散与扩容带来的问题；

### 水平切分

水平切分有多种方法，不同的切分有对应的优缺点

+ id取模
    
