

# Clickhouse

## 入门

### 特点

- 列式存储数据库（DBMS)
- 在线分析处理查询（OLAP），分析型
- C++ 实现
- SQL化操作
- 数据分区与线程级并行



## 安装

### 准备工作

1. 确定防火墙处于关闭状态

   ```shell
   systemctl status firewalld 				#查看防火墙状态
   systemctl stop firewalld 					#关闭防火墙
   systemctl disable firewalld				#关闭防火墙开机启动
   ```

   

2. CentOS取消打开文件数限制

   ```shell
   vim /etc/security/limits.conf
   vim /etc/security/limits.d/20-nproc.conf
   ```

   添加内容

   ```tex
   * soft nofile 65536
   * hard nofile 65536
   * soft nproc 131072
   * hard nproc 131072
   ```

   

3. CentOS 取消 SELINUX  

   ```shell
    vim /etc/selinux/config
    SELINUX=disabled
   ```

   

4. 安装依赖

   ```shell
   sudo yum install -y libtool	
   sudo yum install -y *unixODBC
   ```

### 安装

1. 上传rpm包

   ![image-20210814080717706](imges/image-20210814080717706.png)

2. 安装并检查

   ```shell
    rpm -ivh *.rpm 									#安装
    rpm -qa|grep clickhouse 					#查看安装情况
   ```

   

3. 修改配置文件

   ```
    vim /etc/clickhouse-server/config.xml
    打开注释 <listen_host>::</listen_host>
   ```

   

4. 启动

   ```
   systemctl start clickhouse-server 				#启动
   systemctl disabled clickhouse-server			#关闭开机自启
   ```

   

## 数据类型

### 整型

#### 有符号

范围：（-2n-1~2n-1-1）

Int8 - [-128 : 127]

Int16 - [-32768 : 32767]

Int32 - [-2147483648 : 2147483647]

Int64 - [-9223372036854775808 : 9223372036854775807]

#### 无符号

范围：（0~2n-1）

UInt8 - [0 : 255]

UInt16 - [0 : 65535]

UInt32 - [0 : 4294967295]

UInt64 - [0 : 18446744073709551615]

### 浮点型

Float32 - float

Float64 – double

### 布尔型

没有单独的类型来存储布尔指，可以使用UInt8类型，取值限制为0或1

### Decimal型

- Decimal32(s)，相当于 Decimal(9-s,s)，有效位数为 1~9
- Decimal64(s)，相当于 Decimal(18-s,s)，有效位数为 1~18
- Decimal128(s)，相当于 Decimal(38-s,s)，有效位数为 1~38

### 字符串

#### String

字符串可以任意长度。可以包含任意的字节集，包含空字节。

#### FixedString(N)

固定长度N的字符串，N必须是严格的正自然数。当服务端读区长度小于N的字符串的时候，通过在末尾添加空字节来达到N字节长度。当服务端读取长度大于N的字符串的时候，将返回错误消息。

### 枚举型

Enum8 用 'String'= Int8 对描述。

Enum16 用 'String'= Int16 对描述。

CASE:

​	建表语句

```sql
 create table t_enum( x Enum8('hello' = 1, 'word' =2 ) ) ENGINE = TinyLog;
```

​	数据写入语句：

```sql
INSERT INTO t_enum VALUES ('hello'),('word'),('hello');
```

​	查询语句：

```sql
select * from t_enum;
select cast(x,'Int8') from t_enum;
```

### 时间类型

Date: 接受年-月-日字符串，比如'2021-08-14'

Datetime: 接受年-月-日 时:分:秒 字符串，比如'2021-08-14 10:00:00'

Datetime64: 接受年-月-日 时：分：秒.亚秒的字符串，比如 '2021-08-14 10:00:00.00'

### 数组

Array(T):由T类型元素组成的数组。T 可以是任意类型，包含数组类型。 但不推荐使用多维数组，ClickHouse 对多维数组

的支持有限。例如，不能在 MergeTree 表中存储多维数组。

创建数组：

```sql
select array(1,2) as x ,toTypeName(x);
select [1,2] as x , toTypeName(x);
```

## 表引擎

### 作用

- 数据的存储方式和位置，写到哪里以及从哪里读取数据
- 支持那些查询以及如何支持
- 并发数据访问
- 索引的使用（如果存在）
- 是否可以执行多线程的请求
- 数据复制参数

**特别注意：引擎名称大小写敏感**

### TinyLog

​	以列文件的形式保存在磁盘上，不支持索引，没有并发控制。一般保存少量数据的小表，生产环境上作用有限。

```sql
create table t_tinylog (id String,name String ) engine = TinyLog;
```

### Memory

​	内存引擎，数据以未压缩的原始形式直接保存在内存中，服务器重启数据就会消失。读写操作不会相互阻塞，不支持索引，简单查询下性能非常高（10G/s)。一般用户测试，且数据量不大。

### MergeTree

​	ClickHouse 中最强大的表引擎当属 MergeTree（合并树）引擎及该系列（*MergeTree）中的其他引擎，支持索引和分区，地位可以相当于 innodb 之于 Mysql。

建表语句：

```sql
create table t_order_mt(
id UInt32,
sku_id String,
total_amount Decimal(16,2),
create_time Datetime
) engine =MergeTree
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id,sku_id);


insert into t_order_mt values
(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

#### Partition by 分区（可选）

##### 作用

分区的目的主要是降低扫描范围，优化查询速度。如果不填只会使用一个分区。

##### 分区目录

​	MergeTree 是以列文件+ 索引文件+表定义文件组成的，如果设定了分区，那么这些文件就会保存到不同的分区目录中。

##### 并行

​	分区后，面对涉及跨分区的查询统计，ClickHouse 会以分区为单位并行处理。

##### 数据写入与分区合并

​	任何一个批次的数据写入都会产生一个临时分区，不会纳入任何一个已有的分区。写入

后的某个时刻（大概 10-15 分钟后），ClickHouse 会自动执行合并操作（等不及也可以手动

通过 optimize 执行），把临时分区的数据，合并到已有分区中。

```
optimize table xxxx final;
```

#### Primary key 主键（可选）

​	ClickHouse 中的主键，和其他数据库不太一样，**它只提供了数据的一级索引，但是却不是唯一约束。**这就意味着是可以存在相同 primary key 的数据的。

​	主键的设定主要依据是查询语句中的 where 条件。

​	根据条件通过对主键进行某种形式的二分查找，能够定位到对应的 index granularity,避免了全表扫描。

​	index granularity： 直接翻译的话就是索引粒度，指在稀疏索引中两个相邻索引对应数据的间隔。ClickHouse 中的 MergeTree 默认是 8192。官方不建议修改这个值，除非该列存在大量重复值，比如在一个分区中几万行才有一个不同数据。

#### order by (必选)

​	order by 设定了分区内的数据按照哪些字段顺序进行有序保存。

​	order by 是 MergeTree 中唯一一个必填项，甚至比 primary key 还重要，因为当用户不设置主键的情况，很多处理会依照 order by 的字段进行处理（比如后面会讲的去重和汇总）。

​	**要求：主键必须是 order by 字段的前缀字段。**

#### 二级索引

​	目前在 ClickHouse 的官网上二级索引的功能在 v20.1.2.4 之前是被标注为实验性的，在这个版本之后默认是开启的。 

##### 老版本使用二级索引需要增加设置

​	是否允许使用实验性的二级索引（v20.1.2.4 开始，这个参数已被删除，默认开启）

```
set allow_experimental_data_skipping_indices=1;
```

##### 创建测试表

```sql
create table t_order_mt2(
id UInt32,
sku_id String,
total_amount Decimal(16,2),
create_time Datetime,
INDEX a total_amount TYPE minmax GRANULARITY 5
) engine =MergeTree
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id);


insert into t_order_mt2 values
(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

#### 数据TTL

​	TTL 即 Time To Live，MergeTree 提供了可以管理数据表或者列的生命周期的功能。

##### 列级别TTL

###### 	创建测试表

```
create table t_order_mt3(
id UInt32,
sku_id String,
total_amount Decimal(16,2) TTL create_time+interval 10 SECOND,
create_time Datetime 
) engine =MergeTree
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id);


insert into t_order_mt3 values
(106,'sku_001',1000.00,'2020-06-12 22:52:30'),
(107,'sku_002',2000.00,'2020-06-12 22:52:30'),
(110,'sku_003',600.00,'2020-06-13 12:00:00');
```

##### 表级别TTL

```sql
alter table t_order_mt3 MODIFY TTL create_time + INTERVAL 10 SECOND;
```

能够使用的时间周期：

\- SECOND

\- MINUTE

\- HOUR

\- DAY

\- WEEK

\- MONTH

\- QUARTER

\- YEAR

### ReplacingMergeTree

​	ReplacingMergeTree 是MergeTree的一个变种，它存储特性完全继承 MergeTree，只是多了一个去重的功能。 尽管 MergeTree 可以设置主键，但是 primary key 其实没有唯一约束的功能。如果你想处理掉重复的数据，可以借助这个 ReplacingMergeTree。 

##### 去重时机

​	数据的去重只会在合并的过程中出现。合并会在未知的时间在后台进行，所以你无法预先作出计划。有一些数据可能仍未被处理。

##### 去重范围

​	如果表经过了分区，去重只会在分区内部进行去重，不能执行跨分区的去重。所以ReplacingMergeTree 能力有限，ReplacingMergeTree 适用于在后台清除重复的数据以节省空间，但他不保证没有重复的数据出现。

##### 创建表

```sql
create table t_order_rmt(
id UInt32,
sku_id String,
total_amount Decimal(16,2) ,
create_time Datetime 
) engine =ReplacingMergeTree(create_time)
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id);

insert into t_order_rmt values
(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

##### 结论

- 实际上是使用 order by 字段作为唯一键
- 去重不能夸分区
- 只用同一批插入（新版本）或合并分区时才会进行去重
- 认定重复的数据保留，版本字段值最大的
- 如果版本字段相同则按插入顺序保留最后一笔

### SummingMergetree

​	对于不查询明细，只关心以维度急性汇总聚合结果的场景。如果只使用普通的MergeTree的话，无论是存储空间的开销，还是查询时临时聚合的开销都比较大。ClickHouse 为了这种场景，提供了一种能够“预聚合”的引擎，SummingMergeTree。

##### 创建表

```sql
create table t_order_smt(
id UInt32,
sku_id String,
total_amount Decimal(16,2) ,
create_time Datetime 
) engine =SummingMergeTree(total_amount)
partition by toYYYYMMDD(create_time)
primary key (id)一次
order by (id,sku_id );

insert into t_order_smt values
(101,'sku_001',1000.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

##### 结论

- SummingMergeTree() 中指定的列作为汇总的数据列
- 可以填写多列必须数字列，如果不填，以所有非维度列且为数字列的字段为汇总数据列
- 以 order by 的列为准，作为维度列
- 其他的列按插入顺序保留第一行
- 不在一个分区的数据不会被聚合
- 只有在同一批次插入(新版本)或分片合并时才会进行聚合

## SQL 操作

### Inster

标准语法：

```sql
insert into [table_name] values(…),(….)
```

表到表的插入语法：

```sql
insert into [table_name] select a,b,c from [table_name_2]
```

### Update 和Delete

ClickHouse 提供了 Delete 和 Update 的能力，这类操作被称为 Mutation 查询，它可以看做 Alter 的一种。虽然可以实现修改和删除，但是和一般的 OLTP 数据库不一样，**Mutation** **语句是一种很“重”的操作，而且不支持事务。**“重”的原因主要是每次修改或者删除都会导致放弃目标数据的原有分区，重建新分区。所以尽量做批量的变更，不要进行频繁小数据的操作。

删除语法：

```sql
alter table t_order_smt delete where sku_id ='sku_001';
```

修改操作：

```sql
alter table t_order_smt update total_amount=toDecimal32(2000.00,2) where id =102;
```

### 查询操作

- 支持子查询
- 支持CTE(Common Table Expression 公用表表达式 with 子句)
- 支持各种JOIN，但是JOIN操作无法使用缓存，所以即使时两次相同的JOIN语句，Clickhouse也会视为两条新的SQL
- 函数窗口（官方真正测试中）
- 不支持自定义函数
- Group By 操作增加了 with rollup \ with cube \ with total 用来计算小计和总和

#### 使用案例

```sql
-- with rollup ：从右至左去掉纬度进行小计 
select id , sku_id,sum(total_amount) from t_order_mt group by id,sku_id with rollup;
-- with cube ：从右至左去掉维度进行小计，再从左至右去掉维度进行小计
select id , sku_id,sum(total_amount) from t_order_mt group by id,sku_id with cube;
-- with totals: 只计算合计
select id , sku_id,sum(total_amount) from t_order_mt group by id,sku_id with totals;
```

### alter 操作

#### 新增字段

```sql
alter table tableName add column newcolname String after col1;
```

#### 修改字段类型

```sql
alter table tableName modify column newcolname String;
```

#### 删除字段

```sql
alter table tableName drop column newcolname;
```

