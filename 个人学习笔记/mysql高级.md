#  MySQL 的架构介绍



## MySQL 逻辑架构介绍

> **mysql 的分层思想**

1. 和其它数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的架构上。
2. 插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离。这种架构可以根据业务的需求和实际需要选择合适的存储引擎。

**mysql 四层架构**

1. **连接层**

2. **服务层**

3. **引擎层**

4. **存储层**



### MySQL 部件

**连接器**

-  连接器负责跟客户端建立连接，获取权限、维持和管理连接
- 用户名密码验证
- 查询权限信息，分配对应的权限
-  可以使用show processlist查看现在的连接
- 如果太长时间没有动静，就会自动断开，通过wait_timeout控制，默认8小时

连接可以分为两类：

– 长连接：推荐使用，但是要周期性的断开长连接

– 短链接



**查询缓存**

- 当执行查询语句的时候，会先去查询缓存中查看结果，之前执行过的sql语句及其结果可能以key-value的形式存储在缓存中，如果能找到则直接返回，如果找不到，就继续执行后续的阶段。

- 但是，不推荐使用查询缓存：

  1. 查询缓存的失效比较频繁，只要表更新，缓存就会清空

  2. 缓存对应新更新的数据命中率比较低

     

**分析器**

- 词法分析：Mysql需要把输入的字符串进行识别每个部分代表什么意思

  - 把字符串 T 识别成 表名 T 

  - 把字符串 ID 识别成 列ID 

    ​	词法分析成语法树 `select id ,name from t_user where status = 'ACTIVE' and age = 18`

    <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210910162956835.png" alt="image-20210910162956835" style="zoom:30%;" />

- 语法分析：
- 根据语法规则判断这个sql语句是否满足mysql的语法，如果不符合就会报错“You have an error in your SQL synta”

**优化器**

- 在具体执行SQL语句之前，要先经过优化器的处理

- 当表中有多个索引的时候，决定用哪个索引

- 当sql语句需要做多表关联的时候，决定表的连接顺序

  等等。不同的执行方式对SQL语句的执行效率影响很大

  -  RBO:基于规则的优化
  - CBO:基于成本的优化



### Mysql的执行流程



**mysql 的查询流程：**

1. mysql 客户端通过协议与 mysql 服务器建连接， 发送查询语句， 先检查查询缓存， 如果命中， 直接返回结果。

   如果没命中缓存进行如下操作

2. 解析sql：词法解析和语法解析，解析sql的关键词并检查语法。

3. 优化sql： 一条查询可以有很多种执行方式，最后都会返回相同的结果。 优化器会选择最优的方式去执行。

4. 执行sql：进行语句的执行，查询出结果

   
   
   
   
   mysql整体执行流程架构如下：
   
   
   
   ![image-20211009141630161](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211009141630161.png)
   
   
   
   
   
   
   
   
   
   

## MySQL 存储引擎

> **查看 mysql 存储引擎**

- 查看 mysql 支持的存储引擎

```mysql
show engines;
```

![image-20200803211155030](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODAzMjExMTU1MDMwLnBuZw?x-oss-process=image/format,png)

- 查看 mysql 默认的存储引擎

```mysql
show variables like '%storage_engine%';
```

![image-20200803211257403](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODAzMjExMjU3NDAzLnBuZw?x-oss-process=image/format,png)

**MyISAM 引擎和 InnoDb 引擎的对比**

![image-20200803211313682](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODAzMjExMzEzNjgyLnBuZw?x-oss-process=image/format,png)





**Innodb和myisam的文件存储格式**

innodb

- xxx.frm: 保存了每个表的表结构以及元数据。

- xxx.ibd :保存索引和表数据

   在innodb中，ibd文件的索引和真实数据是存储在一起的（聚簇索引）

myisam

- xxx.frm：保存了每个表的表结构以及元数据。

- xxx.MYD :表的数据文件

- xxx.MYI： 表的索引文件

  在myisam中索引和真实数据是分开存储的（非聚簇索引）



## mysql事务

### 事务复习



**事务（Transation）及其ACID属性**

事务是由一组SQL语句组成的逻辑处理单元，事务具有以下4个属性，通常简称为事务的ACID属性。

1. **原子性**（Atomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
2. **一致性**（Consistent）：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。
3. **隔离性**（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。
4. **持久性**（Durability）：事务院成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

------

**并发事务处理带来的问题**

1. 更新丢失

   （Lost Update）：

   - 当两个或多个事务选择同一行，然后基于最初选定的值更新该行时，由于每个事务都不知道其他事务的存在，就会发生丢失更新问题一一最后的更新覆盖了由其他事务所做的更新。
   - 例如，两个程序员修改同一java文件。每程序员独立地更改其副本，然后保存更改后的副本，这样就覆盖了原始文档。最后保存其更改副本的编辑人员覆盖前一个程序员所做的更改。
   - 如果在一个程序员完成并提交事务之前，另一个程序员不能访问同一文件，则可避免此问题。

2. 脏读

   （Dirty Reads）：

   - 一个事务正在对一条记录做修改，在这个事务完成并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做”脏读”。
   - 一句话：事务A读取到了事务B已修改但尚未提交的的数据，还在这个数据基础上做了操作。此时，如果B事务回滚，A读取的数据无效，不符合一致性要求。

3. 不可重复读

   （Non-Repeatable Reads）：

   - 一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现其读出的数据已经发生了改变、或某些记录已经被删除了！这种现象就叫做“不可重复读”。
   - 一句话：事务A读取到了事务B已经修改并提交了的数据，不符合隔离性

4. 幻读

   （Phantom Reads）

   - 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读一句话：事务A读取到了事务B体提交的新增数据，不符合隔离性。
   - 多说一句：幻读和脏读有点类似，脏读是事务B里面修改了数据，幻读是事务B里面新增了数据。

5. 脏读”、“不可重复读”和“幻读”，其实都是数据库读一致性问题，必须由数据库提供一定的事务隔离机制来解决。

6. 数据库的事务隔离越严格，并发副作用越小，但付出的代价也就越大，因为事务隔离实质上就是使事务在一定程度上“串行化”进行，这显然与“并发”是矛盾的。

7. 同时，不同的应用对读一致性和事务隔离程度的要求也是不同的，比如许多应用对“不可重复读”和“幻读”并不敏感，可能更关心数据并发访问的能力。

8. 查看当前数据库的事务隔离级别：`show variables like 'tx_isolation';` mysql 默认是可重复读

| 隔离级别          | 脏读 | 不可重复  读 | 幻读 |
| ----------------- | ---- | ------------ | ---- |
| READ- UNCOMMITTED | √    | √            | √    |
| READ-COMMITTED    | ×    | √            | √    |
| REPEATABLE- READ  | ×    | ×            | √    |
| SERIALIZABLE      | ×    | ×            | ×    |



------

**事物的隔离级别**

- READ-UNCOMMITTED(读取未提交)： 事务的修改，即使没有提交，对其他事务也都是可见的。事务能够读取未提交的数据，这种情况称为脏读。
- READ-COMMITTED(读取已提交)： 事务读取已提交的数据，大多数数据库的默认隔离级别。当一个事务在执行过程中，数据被另外一个事务修改，造成本次事务前后读取的信息不一样，这种情况称为不可重复读。
- REPEATABLE-READ(可重复读)： 这个级别是MySQL的默认隔离级别，它解决了脏读的问题，同时也保证了同一个事务多次读取同样的记录是一致的，但这个级别还是会出现幻读的情况。幻读是指当一个事务A读取某一个范围的数据时，另一个事务B在这个范围插入行，A事务再次读取这个范围的数据时，会产生幻读
- SERIALIZABLE(可串行化)： 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

事务的隔离级别由**MVCC + 锁** 实现。

MVCC 由隐藏字段、read-view 、undolog实现。

详见MVCC......

# 索引优化分析

## 1、慢 SQL

> **性能下降、 SQL 慢、执行时间长、等待时间长的原因分析**

1. 查询语句写的烂
2. 索引失效：
   - 单值索引：在user表中给name属性建个索引，`create index idx_user_name on user(name)`
   - 组合索引：在user表中给name、email属性建个索引，`create index idx_user_nameEmail on user(name,email)`
3. 关联查询太多join(设计缺陷或不得已的需求)
4. 服务器调优及各个参数设置(缓冲、线程数等)

## 2、join 查询

### 2.1、SQL 执行顺序

> **我们手写的 SQL 顺序**

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODAzMjEyNzMxMTgzLnBuZw?x-oss-process=image/format,png" alt="image-20200803212731183" style="zoom:60%;" />

> **MySQL 实际执行 SQL 顺序**

1. mysql 执行的顺序：随着 Mysql 版本的更新换代， 其优化器也在不断的升级， 优化器会分析不同执行顺序产生的性能消耗不同而动态调整执行顺序。
2. 下面是经常出现的查询顺序：

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODAzMjEyODI2NDI5LnBuZw?x-oss-process=image/format,png" alt="image-20200803212826429" style="zoom:67%;" />

- 总结：mysql 从 FROM 开始执行~

![image-20200803212936111](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODAzMjEyOTM2MTExLnBuZw?x-oss-process=image/format,png)

为什么select后执行的原因：

mysql的执行流程就是连接后解析sql，优化sql、执行sql，在解析sql后，将语句解析生成语法解析树，然后才会执行。因此select在后面

而group by 和limit是要生成结果后经过分组、过滤后的，因此这俩在select后面

### 2.2、JOIN 连接查询

> **常见的 JOIN 查询图**

<img src="https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODAzMjEzMTA2NDM0LnBuZw?x-oss-process=image/format,png" alt="image-20200803213106434" style="zoom: 67%;" />

> **建表 SQL**

```mysql
CREATE TABLE tbl_dept(
	id INT(11) NOT NULL AUTO_INCREMENT,
	deptName VARCHAR(30) DEFAULT NULL,
	locAdd VARCHAR(40) DEFAULT NULL,
	PRIMARY KEY(id)
)ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

CREATE TABLE tbl_emp (
	id INT(11) NOT NULL AUTO_INCREMENT,
	NAME VARCHAR(20) DEFAULT NULL,
	deptId INT(11) DEFAULT NULL,
	PRIMARY KEY (id),
	KEY fk_dept_Id (deptId)
	#CONSTRAINT 'fk_dept_Id' foreign key ('deptId') references 'tbl_dept'('Id')
)ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO tbl_dept(deptName,locAdd) VALUES('RD',11);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('HR',12);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('MK',13);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('MIS',14);
INSERT INTO tbl_dept(deptName,locAdd) VALUES('FD',15);

INSERT INTO tbl_emp(NAME,deptId) VALUES('z3',1);
INSERT INTO tbl_emp(NAME,deptId) VALUES('z4',1);
INSERT INTO tbl_emp(NAME,deptId) VALUES('z5',1);
INSERT INTO tbl_emp(NAME,deptId) VALUES('w5',2);
INSERT INTO tbl_emp(NAME,deptId) VALUES('w6',2);
INSERT INTO tbl_emp(NAME,deptId) VALUES('s7',3);
INSERT INTO tbl_emp(NAME,deptId) VALUES('s8',4);
INSERT INTO tbl_emp(NAME,deptId) VALUES('s9',51);
```

**7 种 JOIN 示例**

**笛卡尔积**

1. tbl_emp 表和 tbl_dept 表的笛卡尔乘积：`select * from tbl_emp, tbl_dept;`
2. 其结果集的个数为：5 * 8 = 40

```sql
mysql> select * from tbl_emp, tbl_dept;
40 rows in set (0.00 sec)
```



------

**inner join**

1. tbl_emp 表和 tbl_dept 的交集部分（公共部分）
2. `select * from tbl_emp e inner join tbl_dept d on e.deptId = d.id;`

```SQL
mysql> select * from tbl_emp e inner join tbl_dept d on e.deptId = d.id;
```

------

**left join**

1. tbl_emp 与 tbl_dept 的公共部分 + tbl_emp 表的独有部分
2. left join：取左表独有部分 + 两表公共部分
3. `select * from tbl_emp e left join tbl_dept d on e.deptId = d.id;`

```SQL
mysql> select * from tbl_emp e left join tbl_dept d on e.deptId = d.id;
```



------

**right join**

1. tbl_emp 与 tbl_dept 的公共部分 + tbl_dept表的独有部分
2. right join：取右表独有部分 + 两表公共部分
3. `select * from tbl_emp e right join tbl_dept d on e.deptId = d.id;`

```SQL
mysql> select * from tbl_emp e right join tbl_dept d on e.deptId = d.id;
```





![image-20210711172506961](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210711172506961.png)

LEFT JOIN: 左边表是主表，会显示左边表的所有部分，且左边表对应的右边表虽然不存在但是也会显示，此时显示为null

RIGHT JOIN: 右边表是主表

INNER JOIN: 取两表的公共部分

------

**left join without common part**

1. tbl_emp 表的独有部分：将 left join 结果集中的两表公共部分去掉即可：`where d.id is null`
2. `select * from tbl_emp e left join tbl_dept d on e.deptId = d.id where d.id is null;`

```
mysql> select * from tbl_emp e left join tbl_dept d on e.deptId = d.id where d.id is null;
+----+------+--------+------+----------+--------+
| id | NAME | deptId | id   | deptName | locAdd |
+----+------+--------+------+----------+--------+
|  8 | s9   |     51 | NULL | NULL     | NULL   |
+----+------+--------+------+----------+--------+
1 row in set (0.00 sec)
```

------

**right join without common part**

1. tbl_dept表的独有部分：将 right join 结果集中的两表公共部分去掉即可：`where e.id is null`
2. `select * from tbl_emp e right join tbl_dept d on e.deptId = d.id where e.id is null;`

```
mysql> select * from tbl_emp e right join tbl_dept d on e.deptId = d.id where e.id is null;
+------+------+--------+----+----------+--------+
| id   | NAME | deptId | id | deptName | locAdd |
+------+------+--------+----+----------+--------+
| NULL | NULL |   NULL |  5 | FD       | 15     |
+------+------+--------+----+----------+--------+
1 row in set (0.00 sec)
```

------

**full join** mysql 不支持 full join ，但是可以通过union 合并

```sql
select * from tbl_emp e left join tbl_dept d on e.deptId = d.id
union 
select * from tbl_emp e right join tbl_dept d on e.deptId = d.id;
```



------

**full join without common part**

1. tbl_emp 表的独有部分 + tbl_dept表的独有部分


```sql
mysql> select * from tbl_emp e left join tbl_dept d on e.deptId = d.id where d.id is null
    -> union
    -> select * from tbl_emp e right join tbl_dept d on e.deptId = d.id where e.id is null;
+------+------+--------+------+----------+--------+
| id   | NAME | deptId | id   | deptName | locAdd |
+------+------+--------+------+----------+--------+
|    8 | s9   |     51 | NULL | NULL     | NULL   |
| NULL | NULL |   NULL |    5 | FD       | 15     |
+------+------+--------+------+----------+--------+
2 rows in set (0.00 sec)
```



## 3、索引简介

### 3.1、索引是什么

1. MySQL官方对索引的定义为：索引(Index)是帮助MySQL高效获取数据的数据结构。可以得到索引的本质：**索引是数据结构**

2. 你可以简单理解为"**排好序的快速查找数据结构**"，即**索引 = 排序 + 查找**

3. 一般来说索引本身占用内存空间也很大，不可能全部存储在内存中，因此**索引往往以文件形式存储在硬盘上**

4. 我们平时所说的索引，如果没有特别指明，都是指B树(多路搜索树，并不一定是二叉树)结构组织的索引。

5. 聚集索引，次要索引，覆盖索引，组合索引，前缀索引，唯一索引默认都是使用B+树索引，统称索引。当然，**除了B+树这种类型的索引之外，还有哈希索引(hash index)等**。

   

### 3.2、索引原理

> **将索引理解为"排好序的快速查找数据结构"**

1. 在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构**以某种方式引用（指向）数据**，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。
2. 下图就是一种可能的索引方式示例：
   - 左边是数据表，一共有两列七条记录，最左边的十六进制数字是数据记录的物理地址
   - 为了加快col2的查找，可以维护一个右边所示的二叉查找树，**每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针**，这样就可以运用二叉查找在一定的复杂度内获取到相应数据，从而快速的检索出符合条件的记录。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712165551587.png" alt="image-20210712165551587" style="zoom:50%;" />

举例：如果我们想要找23这个数据，没有索引的情况下需要遍历整张表才能找到对应的数据，有索引的情况下只需要两次即可找到23，起到快速定位和查找对应数据的作用



索引的结构：磁盘块（数据页）+ 数据块+ 指针

### 3.3、索引优劣势

> **索引的优势**

1. 提高数据检索效率，**通过减少磁盘IO次数来达到降低数据库的IO成本的目的**
2. 通过索引列对数据进行排序，**降低数据排序成本**，降低了CPU的消耗
3. **加快了查询速度**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210720153740519.png" alt="image-20210720153740519" style="zoom:50%;" />

> **索引的劣势**

1. 实际上索引也是一张表，该表保存了主键和索引字段，并指向实体表的记录，所以**索引也占用空间**
2. **虽然索引大大提高了查询速度，同时却会降低更新表的速度**（增删改）。因为更新表时，Mysql不仅要更新数据，还会更新索引
3. 索引易失效需要花时间研究建立优秀的索引，或优化查询语句

### 3.4、MySQL 索引分类

1. 普通索引：是最基本的索引，即一个索引只包含单个列，一个表可以有多个单列索引

2. 唯一索引：索引的每个值都是唯一的，但允许有空值。

3. 主键索引：设置主键时数据库会自动创建

4. 组合索引：即一个索引包含多个列

5. 全文索引：主要用来查找文本中的关键字，而不是直接与索引中的值相比较。fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配

   ```sql
   `ALTER TABLE tbl_name ADD FULLTEXT index_name(column_list)`： # 该语句指定了索引为FULLTEXT，用于全文索引。
   ```




#### 覆盖索引

表结构如下：

```sql
CREATE TABLE `tuser` (
  `id` int(11) NOT NULL,
  `id_card` varchar(32) DEFAULT NULL,
  `name` varchar(32) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `ismale` tinyint(1) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `id_card` (`id_card`),
  KEY `name_age` (`name`,`age`)
) ENGINE=InnoDB
```

如果现在有一个高频请求，要根据市民的身份证号查询他的姓名，这个联合索引就有意义了。

创建一个身份证号和姓名的联合索引，此时通过身份证号查询姓名就是覆盖索引，**不再需要回表查整行记录**，减少语句的执行时间。

```sql
select name from tuser where id_card =  230227200308082328;
```





#### 索引下推

还是以市民表的联合索引（name, age）为例。如果现在有一个需求：检索出表中“名字第一个字是张，而且年龄是 10 岁的所有男孩”。那么，SQL 语句是这么写的：

```sql
select * from tuser where name like '张 %' and age=10 and ismale=1;
```

在 MySQL 5.6 之前，只能从 ID3 开始一个个回表。到主键索引上找出数据行，再对比字段值。

而 MySQL 5.6 引入的索引下推优化（index condition pushdown)， 可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数

无索引下推执行流程

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210724182028070.png" alt="image-20210724182028070" style="zoom: 33%;" />

有索引下推执行流程

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210724182052094.png" alt="image-20210724182052094" style="zoom: 33%;" />

可见，在无索引下推中直接通过第一个索引name找到符合条件的记录后就回表了，有索引下推通过第一个索引name找到符合条件的记录后发现后面的条件是联合索引中的下一个索引，因此接着通过索引过滤另一个结果。将所有结果过滤完后再回表，明显减少回表次数



### 3.5、MySQL 索引语法



查看索引（`\G`表示将查询到的横向表格纵向输出，方便阅读）

```mysql
SHOW INDEX FROM table_name\G 
```



------

**带你看看 mysql 索引：Index_type 为 BTREE**

​	show index from t_emp;![image-20210712122808095](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712122808095.png)



### 3.6、MySQL 索引结构

#### 3.6.1、Btree 索引

> **Btree 索引搜索过程**

【初始化介绍】

1. 一颗 b 树， **蓝色的块我们称之为一个磁盘块**， 可以看到每个磁盘块包含几个**数据项 和指针（黄色所示）**
2. 如磁盘块 1 包含数据项 17 和 35， 包含指针 P1、 P2、 P3
3. P1 表示小于 17 的磁盘块， P2 表示在 17 和 35 之间的磁盘块， P3 表示大于 35 的磁盘块
4. **真实的数据存在于叶子节点和非叶子节点中**

【查找过程】

1. 如果要查找数据项 29， 那么首先会把磁盘块 1 由磁盘加载到内存， 此时发生一次 IO， 在内存中用二分查找确定 29在 17 和 35 之间， 锁定磁盘块 1 的 P2 指针， 内存时间因为非常短（相比磁盘的 IO） 可以忽略不计
2. 通过磁盘块 1的 P2 指针的磁盘地址把磁盘块 3 由磁盘加载到内存， 发生第二次 IO， 29 在 26 和 30 之间， 锁定磁盘块 3 的 P2 指针
3. 通过指针加载磁盘块 8 到内存， 发生第三次 IO， 同时内存中做二分查找找到 29， 结束查询， 总计三次 IO。

![image-20200804093236612](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MDkzMjM2NjEyLnBuZw?x-oss-process=image/format,png)

#### 3.6.2、B+tree 索引

原理图：![image-20210712142521318](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712142521318.png)

【B+Tree 与 BTree 的区别】

B树的关键字（数据项）和记录是放在一起的； B+树的非叶子节点中只有关键字和指向下一个节点的索引， 记录只放在叶子节点中。







真实的情况是， 3 层的 B+ 树可以表示上百万的数据， 如果上百万的数据查找只需要三次 IO， 性能提高将是巨大的，如果没有索引， 每个数据项都要发生一次 IO， 那么总共需要百万次的 IO， 显然成本非常非常高。

【思考： 为什么说 B+树比 B-树更适合实际应用中操作系统的文件索引和数据库索引？】

1. B树中非叶子节点和叶子节点上都存在数据，B+树上只有叶子节点存在数据，**因此B+树的性能更稳定**。
2.   B+树每个节点可容纳的元素个数更多，B+树中访问一个磁盘块中会有大量数据，B树中一个磁盘块只有一个数据。 树高也比B树更小， **直接带来的好处就是减少IO次数**。
3.  **B+树的叶子节点使用指针连接， 方便顺序遍历（范围搜索）**。





### 3.7、何时需要/不需要创建索引

> **哪些情况下适合建立索引**

2. **频繁作为查询的条件的字段**应该创建索引
3. **查询中与其他表关联的字段**，外键关系建立索引
7. **查询中排序、统计、或者分组字段**

> **哪些情况不要创建索引**

1. 表记录太少
2. **经常增删改的表**，**字段频繁的更新**：因为每次增删改的同时也会修改数据的索引，从而降低增删改的效率，但这也视情况而定
3. **数据重复且分布平均的表字段**： 因此应该只为经常查询和经常排序的数据列建立索引。注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。



------

**案例分析：**

1. 假如一个表有10万行记录，有一个字段A只有T和F两种值，且每个值的分布概率大约为50%，那么对这种表A字段建索引一般不会提高数据库的查询速度。
2. 索引的选择性是指索引列中不同值的数目与表中记录数的比。如果一个表中有2000条记录，表索引列有1980个不同的值，那么这个索引的选择性就是1980/2000=0.99。
3. 一个索引的选择性越接近于1，这个索引的效率就越高。



### 3.8、聚簇索引和非聚簇索引

### 非聚簇索引（辅助索引）

MyIsam的B+树实现示意图：

![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MDc1NDI1,size_16,color_FFFFFF,t_70.png)

可以看出，MyIsam的叶子节点只存储了数据对应的引用地址，并指向了真实数据，此为**非聚簇索引**

### 聚簇索引

InnoDB的主键索引的B+树实现示意图：

![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MDc1NDI1,size_16,color_FFFFFF,t_70-20210714215059455.png)

InnoDB的叶子节点存储的就是真实数据本身，此为**聚簇索引**。

**聚簇索引就是按照每张表的主键构造一颗B+树，只存在于Innodb存储引擎上**，同时叶子节点中存放的就是整张表的行记录数据，也将聚集索引的叶子节点称为数据页。这个特性决定了索引组织表中数据也是索引的一部分，**每张表只能拥有一个聚簇索引**。

　　Innodb通过主键聚集数据，**如果没有定义主键，innodb会选择非空的唯一索引代替**。**如果没有这样的索引，innodb会隐式的定义一个主键来作为聚簇索引。**因此**innodb上必有且仅有一个聚簇索引**



**对比**

**MyISM主键索引使用的是非聚簇索引**，在myisam中主键索引和非主键索引的节点结构完全一致，只是存储的内容不同而已，主键索引B+树的节点存储了主键的值，非主键索引B+树存储了非主键的值。表数据存储在独立的地方，这两颗B+树的叶子节点都使用一个地址指向真正的表数据，对于表数据来说，这两个键没有任何差别。由于索引树是独立的，通过辅助键检索无需访问主键的索引树。因此在myisam不存在回表的现象

**Innodb主键索引使用的是聚簇索引**，在Innodb中主键索引和非主键索引结构上是不同的，主键索引上存的是真实数据，非主键索引对应的键上存的是其对应的主键id值，因此通过非主键索引查询数据会产生回表（注：通过非主键索引查询也有不回表的情况：例如 `select id, coloum from table where coloum = ''` 此时就直接通过非主键索引查到数据)

![image-20210724162828850](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210724162828850.png)

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210724163635172.png" alt="image-20210724163635172" style="zoom:30%;" />



**总结：聚簇索引和非聚簇索引的区别：索引字段和真实数据是否在一起，换句话说就是数据文件和索引文件是否放在了一起。**

**文件存放形式**

innodb

- xxx.frm: 保存了每个表的表结构以及元数据。

- xxx.ibd :保存索引和表数据

   在innodb中，ibd文件的索引和真实数据是存储在一起的（聚簇索引）

myisam

- xxx.frm：保存了每个表的表结构以及元数据。

- xxx.MYD :表的数据文件

- xxx.MYI： 表的索引文件

  在myisam中索引和真实数据是分开存储的（非聚簇索引）

### 3.9、索引模型

1. 哈希索引

    **哈希表这种结构只适用于等值查询精确匹配的场景，不适合使用范围查询**，哈希索引用在mysql的memory引擎中或者Memcached 及其他一些 NoSQL 引擎。

哈希索引自身只需存储对应的hash值，所以索引的结构十分紧凑，这让哈希索引查找的速度非常快



 哈希索引的缺点

1. 利用hash存储的话**需要将所有的数据文件添加到内存，耗费内存空间**

2. 如果搜有的查询时等只查询就会很快，但是实际开发中使用范围查找的sql更多，因此hash索引就不合适

   



     案例
    
     当需要存储大量的URL，并且根据URL进行搜索查找，如果使用B+树，存储的内容就会很大 select id from url where url="" 也可以利用将url使用CRC32做哈希，可以使用以下查询方式： select id fom url where url="" and url_crc=CRC32("") 此查询性能较高原因是使用体积很小的索引来完成查找

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210718165040266.png" alt="image-20210718165040266" style="zoom:50%;" />

2. 有序数组索引

   **有序数组在等值查询和范围查询场景中的性能就都非常优秀**，但是**有序数组索引只适用于静态存储引擎**，比如你要保存的是 2021 年某个城市的所有人口信息，这类不会再修改的数据。因为一旦添加和删除数据，就会造成数据的挪动，成本太高

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210718165113340.png" alt="image-20210718165113340" style="zoom:50%;" />

3. 树形索引

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210718165242768.png" alt="image-20210718165242768" style="zoom:50%;" />

## 4、Mysql执行计划分析



### 4.1、性能优化概述



**MySQL 常见瓶颈**

1. CPU 瓶颈：CPU在饱和的时候一般发生在数据装入在内存或从磁盘上读取数据时候
2. IO 瓶颈：磁盘I/O瓶颈发生在装入数据远大于内存容量时
3. 服务器硬件的性能瓶颈：top、free、iostat和vmstat来查看系统的性能状态

### 4.2、Explain 概述

> **Explain**

**是什么？Explain 是查看执行计划**

1. 使用EXPLAIN关键字可以模拟优化器执行SQL语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是结构的性能瓶颈
2. 官网地址：https://dev.mysql.com/doc/refman/8.0/en/explain-output.html

------

**能干嘛？**

1. 分析表的读取顺序（id 字段）
2. 分析数据读取操作的操作类型（select_type 字段）
3. 分析哪些索引可以使用（possible_keys 字段）
4. 分析哪些索引被实际使用（keys 字段）
5. 分析表之间的引用（ref 字段）
6. 分析每张表有多少行被优化器查询（rows 字段）

------

**怎么玩？**

- Explain + SQL语句



### 4.3、Explain 详解

#### ①. id

**select查询的序列号，包含一组数字，表示查询中执行select子句或操作表的顺序**

这里的id并不是所谓的主键

id 取值的三种情况：

1. **id相同**，执行顺序由上至下执行

   ![image-20200804101016101](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTAxMDE2MTAxLnBuZw?x-oss-process=image/format,png)

2. **id不同**，如果是子查询，id的序号会递增，**id值越大优先级越高，越先被执行**

   ![image-20200804101005684](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTAxMDA1Njg0LnBuZw?x-oss-process=image/format,png)

3. **即存在id相同和不同**：先执行id优先级高的，优先级相同的从上至下看

   ![image-20200804101502048](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTAxNTAyMDQ4LnBuZw?x-oss-process=image/format,png)





#### ②select_type

**查询的类型，主要用于区别普通查询、联合查询、子查询等复杂查询**

参数讲解:

> 1. SIMPLE：简单的select查询，查询中不包含子查询或者UNION
> 2. PRIMARY：主要的查询，一般在查询的最外层
> 3. SUBQUERY：见名知义，包含**在 select 中的子查询（不在 from 子句中）**
> 4. DERIVED：**在 from 子句中子查询**，MySQL 会将结果存放在一个临时表中，也称为**派生表（derived 的英文含义）**
> 5. UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
> 6. UNION RESULT：从UNION表获取结果的SELECT

**举栗**

```sql
explain select (select 1 from actor where id = 1) from (select * from film where id = 1) alias;
```

![image-20210713124208231](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210713124208231.png)



**UNION 和 UNION RESULT举例**

```sql
EXPLAIN
select * from t_emp e LEFT JOIN t_dept d on e.deptId = d.id
union
select * from t_emp e RIGHT JOIN t_dept d on e.deptId = d.id;
```

![image-20210712154243952](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712154243952.png)

#### ③table

表示 explain 的那一行访问的表是哪个



#### ④type

![image-20210715224923504](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715224923504.png)

访问类型排列，显示查询使用了何种类型

从最好到最差依次是：**system>const>eq_ref>ref>range>index>ALL**一般来说，得保证查询至少达到range级别，最好能达到ref。

1. system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，这个可以忽略不计

   

2. const：该表至多有一个匹配行，const 用于在和 primary key 或 unique 索引中有固定值比较的情形。

   ```sql
   EXPLAIN select * from t_emp where id = 1;
   ```

   ![image-20210712154934978](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712154934978.png)

   

3. eq_ref：**最多只返回一条符合条件的记录**。在使用唯一性索引或主键查找时会出现该值，非常高效。

   ```sql
   EXPLAIN select * from t_emp e LEFT JOIN t_dept d on e.deptId = d.id where e.deptId is NULL;
   # 查询e表的独有值，表中只有一个记录
   ```
   
4. ref：**索引查找**，不使用唯一索引，使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行。

   ![image-20200804103959011](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTAzOTU5MDExLnBuZw?x-oss-process=image/format,png)

   **注：eq_ref 和ref的区别是都是使用到索引，前者是通过指定条件（如 where col = '')检索索引并只返回一条符合条件的记录，后者通过指定条件检索索引并返回多条符合条件的记录**

5. **range**：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引一般就是在你的where语句中出现了`between`、`<`、`>`、`in`等的查询这种范围扫描索引扫描比全表扫描要好，因为他**只需要开始索引的某一点，而结束于另一点，不用扫描全部索引**

   ![image-20200804104130086](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTA0MTMwMDg2LnBuZw?x-oss-process=image/format,png)

   

6. **index**：Full Index Scan，index与ALL区别为index类型**只遍历索引树**。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说**虽然all和index都是读全表，但index是从索引中读取的，而all是从硬盘数据库文件中读的**）

   ```sql
   create index idx_emp_name on tbl_emp(name);  # 对t_emp表中的name字段创建了索引
   explain select name from tbl_emp;
   ```

   

7. **all**：FullTable Scan，**将遍历全表以找到匹配的行（全表扫描）**

   ```sql
   explain select * from t_emp;
   ```

   

   备注：一般来说，得保证查询只是达到range级别，最好达到ref



#### ⑤possible_keys 

1. 显示可能会在这个表中使用的一个或多个索引

2. 若查询涉及的字段上存在索引，则该索引将被列出，但不一定被查询实际使用

   

#### ⑥key

1. **实际**使用的索引，如果为null，则没有使用索引

2. **若查询中使用了覆盖索引，则该索引仅出现在key列表中**

   ![image-20200804105225100](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTA1MjI1MTAwLnBuZw?x-oss-process=image/format,png)



#### ⑦key_len

1. **表示在索引中使用的字节数**，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好
2. key_len显示的值为**索引最大可能长度，并非实际使用长度**，即key_len是根据表定义计算而得，不是通过表内检索出的

![image-20200804105833936](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTA1ODMzOTM2LnBuZw?x-oss-process=image/format,png)



#### ⑧ref 

1. **显示索引哪一列被使用了**，如果可能的话，最好是一个常数。哪些列或常量被用于查找索引列上的值
2. 由key_len可知t1表的索引idx_col1_col2被充分使用，t1表的col1匹配t2表的col1，t1表的col2匹配了一个常量，即ac。



注意区分type参数中的ref非唯一索引扫描



#### ⑧rows

rows列显示Mysql执行查询时认为它执行查询时必须检查的行数 

通俗的讲：就是显示查过了多少行！

![image-20200804111249713](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTExMjQ5NzEzLnBuZw?x-oss-process=image/format,png)





#### ⑨Extra

包含不适合在其他列中显示但十分重要的额外信息

1. **Using filesort（文件排序）：**

   - MySQL中无法利用索引完成排序操作成为“文件排序”
   - 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取
   - **出现 Using filesort 是不好的情况(九死一生)！，需要尽快优化 SQL！**
   - 示例中第一个查询只使用了 col1 和 col3，原有索引派不上用场，所以进行了外部文件排序
   - 示例中第二个查询使用了 col1、col2 和 col3，原有索引派上用场，无需进行文件排序

   ![image-20200804111738931](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTExNzM4OTMxLnBuZw?x-oss-process=image/format,png)

2. **Using temporary（创建临时表）：**

   - 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by
   - **出现 Using temporary 超级不好！（十死无生），需要立即优化 SQL！**
   - 示例中第一个查询只使用了 col1，原有索引派不上用场，所以创建了临时表进行分组
   - 示例中第二个查询使用了 col1、col2，原有索引派上用场，无需创建临时表

   ![image-20200804112558915](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTEyNTU4OTE1LnBuZw?x-oss-process=image/format,png)

3. **Using index（覆盖索引）：**

   表示相应的select操作中使用了覆盖索引（Coveing Index），避免访问了表的数据行，效率不错！

   - 如果同时出现using where，表明索引被用来执行索引键值的查找

   - 如果没有同时出现using where，表明索引用来读取数据而非执行查找动作

     ![image-20200804113450147](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTEzNDUwMTQ3LnBuZw?x-oss-process=image/format,png)

   - **覆盖索引（Covering Index）**

     查询的所有字段中都用到了索引，查询到的字段中如果出现索引以外的字段就会产生回表。

4. **Using where**：表明使用了where过滤

5. **Using join buffer**：表明使用了连接缓存

6. impossible where：where子句的值总是false，不能用来获取任何元组

   只有错误的查询才会出现此情况！

   ```sql
   explain select id,name,age from t_emp where id = 1 and id = 2; 
   ```

7. select tables optimized away：在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化`COUNT(*)`操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

8. distinct：优化distinct，在找到第一匹配的元组后即停止找同样值的工作






### 4.4 Explain 热身 Case

![image-20200804114203012](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTE0MjAzMDEyLnBuZw?x-oss-process=image/format,png)



## 5、SQL JOIN索引优化

### 5.1、单表索引优化分析

**创建表**

- 建表 SQL

```mysql
CREATE TABLE IF NOT EXISTS article(
	id INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
	author_id INT(10) UNSIGNED NOT NULL,
	category_id INT(10) UNSIGNED NOT NULL,
	views INT(10) UNSIGNED NOT NULL,
	comments INT(10) UNSIGNED NOT NULL,
	title VARCHAR(255) NOT NULL,
	content TEXT NOT NULL
);

INSERT INTO article(author_id,category_id,views,comments,title,content)
VALUES
(1,1,1,1,'1','1'),
(2,2,2,2,'2','2'),
(1,1,3,3,'3','3');
```

------

**查询案例**

- 查询category_id为1且comments 大于1的情况下，views最多的article_id。

```sql
mysql> SELECT id, author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
```

- 此时 article 表中只有一个主键索引

```sql
SHOW INDEX FROM article;
```

![image-20210712153308003](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712153308003.png)

- 使用 explain 分析 SQL 语句的执行效率

```sql
EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
```

![image-20210712153324454](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712153324454.png)

- 结论：
  - 很显然，type是ALL，即最坏的情况。
  - Extra 里还出现了Using filesort，也是最坏的情况。
  - 优化是必须的。

------

**开始优化：新建索引**

- 在 category_id 列、comments 列和 views 列上建立联合索引

```sql
create index idx_article_ccv on article(category_id, views, comments);

SHOW INDEX FROM article;
```

![image-20210712153736366](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712153736366.png)

- 再次执行查询：

测试1：

```sql
EXPLAIN SELECT id, author_id FROM article WHERE category_id > 1 AND views > 1 ORDER BY comments DESC LIMIT 1;
```

![image-20210712153802114](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712153802114.png)

测试2：

```sql
EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND views > 1 ORDER BY comments DESC LIMIT 1;
```

![image-20210712153909742](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712153909742.png)

测试3：

```sql
EXPLAIN SELECT id, author_id FROM article WHERE category_id > 1 AND views = 1 ORDER BY comments DESC LIMIT 1;
```

![image-20210712153941543](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712153941543.png)

- 分析：
  - 在测试中我们已经建立了索引啊！为啥没用呢？
  - 这是因为按照B+Tree索引的工作原理，先排序 category_id，再排序 views,再排序comments。
  - 当comments字段在联合索引里处于中间位置时，**因为`comments>1`条件是一个范围值**（所谓 range），MySQL 无法利用索引再对后面的views部分进行检索，即 **范围查询字段后面的索引无效**。
- 将查询条件中改为category_id = 1 ,views = 1 ，发现 Use filesort 神奇地消失了，从这点可以验证：范围后的索引会导致索引失效

```sql
EXPLAIN SELECT id, author_id FROM article WHERE category_id = 1 AND views = 1 ORDER BY comments DESC LIMIT 1;
```

![image-20210712154140186](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712154140186.png)

------

**删除索引**

- 删除刚才创建的 idx_article_ccv 索引

```sql
DROP INDEX idx_article_ccv ON article;
```

------

**再次创建索引**

- 由于 range 后（category_id > 1）的索引会失效，这次我们建立索引时，直接抛弃 category_id 列，先利用 views 和comments 的组合索引查询所需要的数据，再从其中取出 category_id > 1的数据（我觉着应该是这样的）

```sql
create index idx_article_ccv on article(views,comments);
```



- 对比测试2执行查询：可以看到，type变为了ref，Extra中的Using filesort也消失了，结果非常理想

  

```sql
EXPLAIN SELECT id, author_id FROM article WHERE category_id > 1 AND views = 1 ORDER BY comments DESC LIMIT 1;
```

![image-20210713234359730](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210713234359730.png)

### 5.2、两表索引优化分析



**创建表**

- 建表 SQL

```mysql
CREATE TABLE IF NOT EXISTS class(
	id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
	card INT(10) UNSIGNED NOT NULL,
	PRIMARY KEY(id)
);

CREATE TABLE IF NOT EXISTS book(
	bookid INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
	card INT(10) UNSIGNED NOT NULL,
	PRIMARY KEY(bookid)
);

INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));

INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));

```



------

**查询案例**

- 实现两表的连接，连接条件是 class.card = book.card

```sql
SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210713234445581.png" alt="image-20210713234445581" style="zoom:50%;" />

- 使用 explain 分析 SQL 语句的性能，可以看到：驱动表是左表 class 表

```sql
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

![image-20210713234610750](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210713234610750.png)

- 结论：
  - **type 有 All ，rows 为表中数据总行数，说明 class 和 book 进行了全表检索**
  - 即每次 class 表对 book 表进行左外连接时，都需要在 book 表中进行一次全表检索

------

**添加索引：在右表添加索引**



- 在 book 的 card 字段上添加索引

```sql
CREATE INDEX idx_card on book(`card`);
```

- 测试结果：可以看到第二行的type变为了ref，rows也变成了1，优化比较明显。

```sql
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

![image-20210713235630120](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210713235630120.png)

- 分析：
  - 这是由左连接特性决定的。LEFT JOIN条件用于确定如何从右表搜索行，左边一定都有，所以右边是我们的关键点，一定需要建立索引。
  - **左表连接右表，则需要拿着左表的数据去右表里面查，索引需要在右表中建立**

------

**添加索引：在右表添加索引**

- 删除之前右边 book 表中的索引

```mysql
DROP INDEX Y ON book;
```

- 在左边 class 表的 card 字段上建立索引

```mysql
ALTER TABLE class ADD INDEX X(card);
```

- 再次执行左连接，发现执行查询的行数共为41行，比之前的22行多了将近一倍

```sql
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
```

![image-20210713235812043](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210713235812043.png)

因此我们可以发现左连接需要在右表的连接字段建立索引

- 别怕，我们来执行右连接：可以看到第二行的type变为了ref，rows也变成了1优化比较明显。

```sql
EXPLAIN SELECT * FROM class RIGHT JOIN book ON class.card = book.card;
```

![image-20210713235847393](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210713235847393.png)

- 分析：
  - 这是因为RIGHT JOIN条件用于确定如何从左表搜索行，使用RIGHT JOIN右边是主表一定都有，所以左边是我们的关键点，一定需要建立索引。
  - RIGHT JOIN book ：book 里面的数据一定存在于结果集中，我们需要拿着 book 表中的数据，去 class 表中搜索，所以索引需要建立在 class 表中

**结论：左外链接索引建右表，右外连接索引建左表！ （索引建在副表，因为主表一定全有）**



### 5.3、三表索引优化分析



**创建表**

- 建表 SQL

```mysql
CREATE TABLE IF NOT EXISTS phone(
	phoneid INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
	card INT(10) UNSIGNED NOT NULL,
	PRIMARY KEY(phoneid)
)ENGINE=INNODB;

INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO phone(card) VALUES(FLOOR(1+(RAND()*20)));
```



------

**查询案例**

- 实现三表的连接查询：

  ```sql
  select * from book LEFT JOIN class on book.card = class.card LEFT JOIN phone on class.card = phone.card;
  ```




- 使用 explain 分析 SQL 指令：

  ```sql
  explain select * from book LEFT JOIN class on book.card = class.card LEFT JOIN phone on class.card = phone.card;
  ```
  
  ![image-20210715213855136](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715213855136.png)
  
- 结论：
  - **type 有All 、rows 为表数据总行数，说明 class、 book 和 phone 表都进行了全表扫描！**
  - Extra 中的Using join buffer ，表明连接过程中使用了 join 缓冲区

------

**优化：**

**创建索引**

- 创建索引,在两个副表上建了索引，此sql中主表book不需要建立索引

```sql
alter table class add index idx_card (card) ;
alter table phone add index idx_card (card);
show index from class;
show index from phone;
```

- **进行 LEFT JOIN ，永远都在右表的字段上建立索引**



- 执行查询：后2行的type都是ref，且总rows优化很好，效果不错。因此索引最好设置在需要经常查询的字段中。

```sql
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card=book.card LEFT JOIN phone ON book.card = phone.card;
```

![image-20210715214142954](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715214142954.png)



**解释：将两个 left join 看作是两层嵌套 for 循环**

1. 尽可能减少Join语句中的NestedLoop的循环总次数；
2. **永远都要小表驱动大表**。原因：驱动表为主表，被驱动表为副表，主表一定会遍历全表，因此要用小表作为驱动，大表作为被驱动表建立索引
3. 优先优化NestedLoop的内层循环；
4. 保证Join语句中被驱动表上Join条件字段已经被索引；
5. 当无法保证被驱动表的Join条件字段被索引且内存资源充足的前提下，不要太吝惜JoinBuffer的设置；



### sql join原理?

​		MySQL是只支持一种Join算法Nested-Loop Join(嵌套循环连接)，并不支持哈希连接和合并连接，不过在mysql中包含了多种变种，能够帮助MySQL提高join执行的效率。

​		**1、Simple Nested-Loop Join **

​		比如两张表t1、t2，t1表中取出所有数据和t2表中所有数据进行关联，t1表中如果有m个数据，t2表中如果有n个数据，就会匹配m*n次（使用的是笛卡尔乘积）

虽然简单，但是相对来说开销还是太大了。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210911000010788.png" alt="image-20210911000010788" style="zoom:30%;" />

​		**2、Index Nested-Loop Join**

​		索引嵌套联系由于非驱动表上有索引，所以比较的时候不再需要一条条记录进行比较，而可以通过索引来减少比较，从而加速查询。这也就是平时我们在做关联查询的时候必须要求关联字段有索引的一个主要原因。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210911000636750.png" alt="image-20210911000636750" style="zoom:33%;" />

​	这种算法在链接查询的时候，驱动表会根据关联字段的索引进行查找，当在索引上找到了符合的值，再回表进行查询，也就是只有当匹配到索引以后才会进行回表。至于驱动表的选择，MySQL优化器一般情况下是会选择记录数少的作为驱动表，但是当SQL特别复杂的时候不排除会出现错误选择。

​		在索引嵌套链接的方式下，如果非驱动表的关联键是主键的话，这样来说性能就会非常的高，如果不是主键的话，关联起来如果返回的行数很多的话，效率就会特别的低，因为要多次的回表操作。先关联索引，然后根据二级索引的主键ID进行回表的操作。这样来说的话性能相对就会很差。

​		**3、Block（块） Nested-Loop Join**

​		当连接字段之间没有索引，且join_buffer_size足够的时候会用到该算法

​			

​		Block Nested-Loop Join对比Simple Nested-Loop Join多了一个中间处理的过程，也就是join buffer，使用join buffer将驱动表的查询JOIN相关列都给缓冲到了JOIN BUFFER当中，然后批量与非驱动表进行比较，这也来实现的话，可以将多次比较合并到一次，降低了非驱动表的访问频率。也就是只需要访问一次S表。这样来说的话，就不会出现多次访问非驱动表的情况了，也只有这种情况下才会访问join buffer。

​		在MySQL当中，我们可以通过参数join_buffer_size来设置join buffer的值，然后再进行操作。默认情况下join_buffer_size=256K，在查找的时候MySQL会将所有的需要的列缓存到join buffer当中，包括select的列，而不是仅仅只缓存关联列。在一个有N个JOIN关联的SQL当中会在执行时候分配N-1个join buffer。

效率介于1和2之间。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210911000931913.png" alt="image-20210911000931913" style="zoom:33%;" />

## 6、索引失效

> **索引失效（应该避免）**

**创建表**

- 建表 SQL

```mysql
CREATE TABLE staffs(
	id INT PRIMARY KEY AUTO_INCREMENT,
	`name` VARCHAR(24)NOT NULL DEFAULT'' COMMENT'姓名',
	`age` INT NOT NULL DEFAULT 0 COMMENT'年龄',
	`pos` VARCHAR(20) NOT NULL DEFAULT'' COMMENT'职位',
	`add_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT'入职时间'
)CHARSET utf8 COMMENT'员工记录表';

INSERT INTO staffs(`name`,`age`,`pos`,`add_time`)VALUES('z3',22,'manager',NOW());
INSERT INTO staffs(`name`,`age`,`pos`,`add_time`) VALUES('July',23,'dev',NOW());
INSERT INTO staffs(`name`,`age`,`pos`,`add_time`) VALUES('2000',23,'dev',NOW());

ALTER TABLE staffs ADD INDEX index_staffs_nameAgePos(`name`,`age`,`pos`);
```

- staffs 表中的测试数据

```sql
select * from staffs;
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715215845718.png" alt="image-20210715215845718" style="zoom:50%;" />

- staffs 表中的组合索引：name、age、pos

```sql
SHOW INDEX FROM staffs;
```

![image-20210715215910810](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715215910810.png)



### 6.1、索引失效准则判断

1. 全值匹配我最爱，即where后面的字段全都要精确匹配

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210718164658080.png" alt="image-20210718164658080" style="zoom:30%;" />

   **全值匹配利用到的是什么特性？索引下推！**

   

2. 最佳左前缀法则：如果索引了多例，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。（带头大哥不能没）

   注：索引左前缀原则。对于组合索引（多个字段中使用同一个索引），我们在查询过程中要使用左前缀原则，即：查询中要用到组合索引中的最左字段

   ![image-20210712184222945](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210712184222945.png)

   

3. 在索引列上做任何操作，（计算、函数、（自动or手动）类型转换）都会导致索引失效而转向全表扫描

4. 存储引擎不能使用索引中范围条件右边的列

5. **尽量使用覆盖索引**（只访问索引的查询（索引列和查询列一致）），减少`select *`

6. mysql在使用不等于（!=或者<>）的时候无法使用索引会导致全表扫描

7. `is null`，`is not null` 也无法使用索引（早期版本不能走索引，后续版本应该优化过，可以走索引）

8. like以通配符开头（’%abc…’）mysql索引失效会变成全表扫描操作

9. **字符串类型的字段不加单引号索引会失效**

10. 少用or，用它连接时会索引失效，建议使用union代替



> **最佳左前缀法则：带头大哥不能死，中间兄弟不能断**

- 只有带头大哥 name 时
  - key = index_staffs_nameAgePos 表明索引生效
  - ref = const ：这个常量就是查询时的 ‘July’ 字符串常量

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July';
```

![image-20210715222749092](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715222749092.png)

- 带头大哥 name 带上小弟 age
  - key = index_staffs_nameAgePos 表明索引生效
  - ref = const,const：两个常量分别为 ‘July’ 和 23

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July'AND age = 23;
```

![image-20210715222905951](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715222905951.png)

这叫索引下推



- 带头大哥 name 带上小弟 age ，小弟 age 带上小小弟 pos
  - key = index_staffs_nameAgePos 表明索引生效
  - ref = const,const,const ：三个常量分别为 ‘July’、23 和 ‘dev’

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July'AND age = 23 AND pos ='dev' 
```

![image-20210715223011981](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715223011981.png)

- 带头大哥 name 挂了
  - key = NULL 说明索引失效
  - ref = null 表示 ref 也失效
  - type=All 说明使用了全表扫描

```sql
EXPLAIN SELECT * FROM staffs WHERE age = 23 AND pos = 'dev';
```

![image-20210715223035168](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715223035168.png)

- 带头大哥 name 没挂，中间的小弟 age 跑了
  - key = index_staffs_nameAgePos 说明索引没有失效
  - ref = const 表明只使用了一个常量，即第二个常量（pos = ‘dev’）没有生效

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July'AND pos = 'dev';
```

![image-20210715223107167](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715223107167.png)



> **在索引列上进行计算，会导致索引失效，进而转向全表扫描**

- 不对带头大哥 name 进行任何操作：key = index_staffs_nameAgePos 表明索引生效

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July';
```

![image-20210715223143824](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715223143824.png)

- 对带头大哥 name 进行操作：使用 LEFT 函数截取子串
  - key = NULL 表明索引生效
  - type = ALL 表明进行了全表扫描

```sql
EXPLAIN SELECT * FROM staffs WHERE LEFT(name,4) = 'July';
```

![image-20210715223740045](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715223740045.png)



> **范围之后全失效**

- 精确匹配
  - type = ref 表示非唯一索引扫描，SQL 语句将返回匹配某个单独值的所有行。
  - key_len = 140 表明表示索引中使用的字节数

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July'AND age = 23 AND pos = 'dev';
```

![image-20210715224008782](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715224008782.png)

- 将 age 改为范围匹配
  - type = range 表示范围扫描
  - key = index_staffs_nameAgePos 表示索引并没有失效
  - key_len = 78 ，ref = NULL 均表明范围搜索使其后面的索引均失效

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July'AND age > 23 AND pos = 'dev';
```

![image-20210715224027868](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715224027868.png)



> **尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）），减少 `select *`**

- `SELECT *` 的写法

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July'AND age > 23 AND pos = 'dev';
```



- 覆盖索引的写法：Extra = Using where; Using index ，Using index 表示使用索引列进行查询，将大大提高查询的效率

```sql
EXPLAIN SELECT name, age, pos FROM staffs WHERE name = 'July'AND age = 23 AND pos = 'dev';
```

- 覆盖索引中包含 range 条件：type = ref 并且 Extra = Using where; Using index ，虽然在查询条件中使用了 范围搜索，但是由于我们只需要查找索引列，所以无需进行全表扫描

**一句话：用什么查什么，尽量少查，用不到的不要查！**





> **mysql在使用不等于（!=或者<>）的时候无法使用索引会导致全表扫描**

- 在使用 != 和 <> 时会导致索引失效：
  - key = null 表示索引失效
  - rows = 3 表示进行了全表扫描

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July';
EXPLAIN SELECT * FROM staffs WHERE name != 'July';
```

![image-20210715225246578](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715225246578.png)

![image-20210715225256256](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715225256256.png)

> 
>
> 
>
> **is null，is not null 也无法使用索引**

- is null，is not null 会导致索引失效：key = null 表示索引失效

```sql
EXPLAIN SELECT * FROM staffs WHERE name is null;
EXPLAIN SELECT * FROM staffs WHERE name is not null;	
```

![image-20210715225452022](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715225452022.png)

![image-20210715225508864](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715225508864.png)



> **like % 要写在最右**

- like % 写在左边的情况
  - type = All ，rows = 3 表示进行了全表扫描
  - key = null 表示索引失效

```sql
EXPLAIN SELECT * FROM staffs WHERE name like '%July';
EXPLAIN SELECT * FROM staffs WHERE name like '%July%';
EXPLAIN SELECT * FROM staffs WHERE name like 'July%';
```

![image-20210715225708504](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715225708504.png)

**可见，将like % 写在右边效果最佳**



**如果业务需求非得要将like左右两边都加%，那就得用到覆盖索引了！**





**创建表**

- 建表 SQL

```sql
CREATE TABLE `tbl_user`(
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(20) DEFAULT NULL,
	`age`INT(11) DEFAULT NULL,
	`email` VARCHAR(20) DEFAULT NULL,
	PRIMARY KEY(`id`)
)ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

INSERT INTO tbl_user(`name`,`age`,`email`)VALUES('1aa1',21,'a@163.com');
INSERT INTO tbl_user(`name`,`age`,`email`)VALUES('2bb2',23,'b@163.com');
INSERT INTO tbl_user(`name`,`age`,`email`)VALUES('3cc3',24,'c@163.com');
INSERT INTO tbl_user(`name`,`age`,`email`)VALUES('4dd4',26,'d@163.com');
```

- tbl_user 表中的测试数据

```sql
select * from tbl_user;
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715225845563.png" alt="image-20210715225845563" style="zoom:50%;" />

------

创建索引

- 在 tbl_user 表的 name 字段和 age 字段创建联合索引

```mysql
CREATE INDEX idx_user_nameAge ON tbl_user(name, age);
SHOW INDEX FROM tbl_user;
```

![image-20210715225907421](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715225907421.png)

------

**测试覆盖索引**

- 如下 SQL 的索引均不会失效：
  - 只要查询的字段能和覆盖索引扯得上关系，并且没有多余字段，覆盖索引就不会失效
  - id是主键，本身也是一种索引！

```mysql
EXPLAIN SELECT name, age FROM tbl_user WHERE NAME LIKE '%aa%';

EXPLAIN SELECT id FROM tbl_user WHERE NAME LIKE '%aa%';  #id是主键索引
EXPLAIN SELECT name FROM tbl_user WHERE NAME LIKE '%aa%';
EXPLAIN SELECT age FROM tbl_user WHERE NAME LIKE '%aa%';

EXPLAIN SELECT id, name FROM tbl_user WHERE NAME LIKE '%aa%';
EXPLAIN SELECT id, age FROM tbl_user WHERE NAME LIKE '%aa%';
EXPLAIN SELECT id, name, age FROM tbl_user WHERE NAME LIKE '%aa%';
```

- 如下 SQL 的索引均会失效：但凡有多余字段，覆盖索引就会失效

```mysql
EXPLAIN SELECT * FROM tbl_user WHERE NAME LIKE '%aa%';
EXPLAIN SELECT id, name, age, email FROM tbl_user WHERE NAME LIKE '%aa%';

EXPLAIN SELECT * FROM tbl_user WHERE NAME LIKE '%aa%';

EXPLAIN SELECT id, name, age, email FROM tbl_user WHERE NAME LIKE '%aa%';
```





> **字符串不加单引号索引失效**

- 正常操作，索引没有失效

```mysql
explain select * from staffs where name='2000';
```



- 如果字符串忘记写 ‘’ ，那么 mysql 会为我们进行隐式的类型转换，但凡进行了类型转换，索引都会失效

```mysql
explain select * from staffs where name=2000;
```



> **使用or导致索引失效**

- 使用 or 连接，会导致索引失效

```mysql
explain select * from staffs where name='z3' or name = 'July';
```

![image-20210715230914784](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715230914784.png)



### 6.2、索引优化面试题

> **索引优化面试题**

**创建表**

- 建表 SQL

```mysql
create table test03(
    id int primary key not null auto_increment,
    c1 char(10),
    c2 char(10),
    c3 char(10),
    c4 char(10),
    c5 char(10)
);

insert into test03(c1,c2,c3,c4,c5) values ('a1','a2','a3','a4','a5');
insert into test03(c1,c2,c3,c4,c5) values ('b1','b2','b3','b4','b5');
insert into test03(c1,c2,c3,c4,c5) values ('c1','c2','c3','c4','c5');
insert into test03(c1,c2,c3,c4,c5) values ('d1','d2','d3','d4','d5');
insert into test03(c1,c2,c3,c4,c5) values ('e1','e2','e3','e4','e5');

create index idx_test03_c1234 on test03(c1,c2,c3,c4);
```



> **问题：我们创建了组合索引idx_test03_c1234，根据以下SQL分析下索引使用情况？**



- 全值匹配

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' AND c3='a3' AND c4='a4';
```

![image-20210715231801080](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715231801080.png)



- mysql 优化器进行了优化，所以我们的索引都生效了

```mysql
EXPLAIN SELECT * FROM test03 WHERE c4='a4' AND c3='a3' AND c2='a2' AND c1='a1';
```

![image-20210715231840112](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715231840112.png)





- c3 列使用了索引进行排序，并没有进行查找，导致 c4 无法用索引进行查找,即验证了索引之后全失效

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' AND c3>'a3' AND c4='a4'; 
```

![image-20210715231916777](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715231916777.png)





- mysql 优化器进行了优化，所以我们的索引都生效了，在 c4 时进行了范围搜索

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' AND c4>'a4' AND c3='a3'; 
```

![image-20210715232756900](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715232756900.png)





- c3 列将索引用于排序，而不是查找，c4 列没有用到索引

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' AND c4='a4' ORDER BY c3; 
```

![image-20210715232900592](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715232900592.png)

​	因为去掉c4项key_len 的长度相同，没有用到c4索引！

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' ORDER BY c3; 
```

![image-20210715233031454](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715233031454.png)







- 妈耶，因为索引建立的顺序和使用的顺序不一致，导致 mysql 动用了文件排序
- 看到 Using filesort 就要知道：此句 SQL 必须优化

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' ORDER BY c4; 
```

![image-20210715233103431](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715233103431.png)



- 只用 c1 一个字段索引，但是c2、c3用于排序，无filesort

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c5='a5' ORDER BY c2, c3; 
```

![image-20210715233418101](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715233418101.png)





- 出现了filesort，我们建的索引是1234，因为它没有按照顺序来，32颠倒了

```sql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c5='a5' ORDER BY c3, c2; 
```

![image-20210715233639334](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715233639334.png)



- 用c1、c2两个字段索引，但是c2、c3用于排序，无filesort

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' ORDER BY c2, c3; 
```

![image-20210715233716144](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715233716144.png)



- 和 c5 这个坑爹货没啥关系

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' AND c5='a5' ORDER BY c2, c3; 
```

![image-20210715233800901](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715233800901.png)



- 注意查询条件 c2=‘a2’ ，我都把 c2 查出来了（c2 为常量），我还给它排序作甚，所以没有产生 filesort

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2='a2' AND c5='a5' ORDER BY c3, c2; 
```

![image-20210715233833537](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715233833537.png)



- **group by 分组之前必排序，group by 和 order by 在索引上的问题基本是一样的**

  

- 结论：
  - group by 基本上都需要进行排序，但凡使用不当，**就会产生临时表！**
  - 定值为常量、范围之后失效，最终看排序的顺序



### 6.3、索引失效总结

> **一般性建议**

1. 对于单键索引，尽量选择针对当前查询过滤性更好的索引
2. 在选择组合索引的时候，当前query中过滤性最好的字段在索引字段顺序中，位置越靠前越好。
3. 在选择组合索引的时候，尽量选择可以能包含当前查询中的where子句中更多字段的索引**（where中要多用索引）**
4. 尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的

> **索引优化的总结**

- like 后面以常量开头，比如 like ‘kk%’ 和 like ‘k%kk%’ ，可以理解为就是常量，因为上来就已经选择好了

![image-20200804162716329](https://imgconvert.csdnimg.cn/aHR0cDovL2hleWdvLm9zcy1jbi1zaGFuZ2hhaS5hbGl5dW5jcy5jb20vaW1hZ2VzL2ltYWdlLTIwMjAwODA0MTYyNzE2MzI5LnBuZw?x-oss-process=image/format,png)

------

**like SQL 实测**

- = ‘kk’ ：key_len = 93 ，请记住此参数的值，后面有用

- like ‘kk%’：
  - key_len = 93 ，和上面一样，说明 c1 c2 c3 都用到了索引
  - type = range 表明这是一个范围搜索

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2 like 'kk%' AND c3='a3';
```

![image-20210715234441631](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210715234441631.png)



- like ‘%kk’ 和 like ‘%kk%’ ：key_len = 31 ，表示只有 c1 用到了索引

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2 like '%kk' AND c3='a3';

EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2 like '%kk%' AND c3='a3';
```





- like ‘k%kk%’ ：key_len = 93 ，表示 c1 c2 c3 都用到了索引

```mysql
EXPLAIN SELECT * FROM test03 WHERE c1='a1' AND c2 like 'k%kk%' AND c3='a3';
```





> **索引优化的总结**

- 全值匹配我最爱， 最左前缀要遵守；
- 带头大哥不能死， 中间兄弟不能断；
- 索引列上少计算， 范围之后全失效；
- LIKE 百分写最右， 覆盖索引不写 *；
- 不等空值还有OR， 索引影响要注意；
- VAR的引号不可丢， SQL 优化有诀窍。

# 第 3 章 查询截取分析

## 查询优化

### 1.1、MySQL 优化原则

> **mysql 的调优大纲**

1. 慢查询的开启并捕获

2. explain+慢SQL分析

3. show profile查询指定SQL语句在Mysql服务器里面的执行细节和生命周期情况

   ```sql
   show profiles
   show PROFILE for QUERY id
   ```

4. SQL数据库服务器的参数调优


> **永远小表驱动大表，类似嵌套循环 Nested Loop**

1. EXISTS 语法：
   - `SELECT ... FROM table WHERE EXISTS(subquery)`
   - 该语法可以理解为：将查询的数据，放到子查询中做条件验证，根据验证结果（TRUE或FALSE）来决定主查询的数据结果是否得以保留。
2. EXISTS(subquery) 只返回TRUE或FALSE，因此子查询中使用`SELECT *`也可使用以SELECT 1`或其他，官方说法是实际执行时会忽略SELECT清单，因此没有区别
3. EXISTS子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比，如果担忧效率问题，可进行实际检验以确定是否有效率问题。
4. EXISTS子查询往往也可以用条件表达式、其他子查询或者JOIN来替代，何种最优需要具体问题具体分析

<img src="https://img-blog.csdnimg.cn/img_convert/ead448201cff67f2a5e05b17ee8a6f4b.png" alt="image-20200805101726632" style="zoom: 67%;" />

------

**结论：**

1. 永远记住小表驱动大表
2. 当 A 表数据集大于 B 表数据集时，使用 in
3. 当 A 表数据集小于 B 表数据集时，使用 exist

------

**in 和 exists 的用法**

- in 的写法

```mysql
select * from tbl_emp e where e.deptId in (select id from tbl_dept);
```

- exists 的写法

```mysql
select * from tbl_emp e where exists (select 1 from tbl_dept d where e.deptId = d.id);
```



### in和exist的区别

　　首先，查询中涉及到的两个表，一个user和一个order表，具体表的内容如下：

　　　　user表：

　　　　![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/1104082-20170226015714054-85509796.png)

　　　　order表：

　　　　![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/1104082-20170226020236195-1260049812.png)

**in**

　　　　确定给定的值是否与子查询或列表中的值相匹配。in在查询的时候，首先查询子查询的表，然后将内表和外表做一个笛卡尔积，然后按照条件进行筛选。所以相对内表比较小的时候，in的速度较快。



```
 1 SELECT
 2     *
 3 FROM
 4     `user`
 5 WHERE
 6     user`.id IN (
 7         SELECT
 8             `order`.user_id
 9         FROM
10             `order`
11     )
```

这条语句很简单，通过子查询查到的user_id 的数据，去匹配user表中的id然后得到结果

它的执行流程：

- 首先，在数据库内部，执行子查询， 
- 然后，将查询到的结果和原有的user表做一个笛卡尔积，结果如下：

　　　　![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/1104082-20170226022145538-205031549.png)

　　　　此时，再根据我们的user.id IN order.user_id的条件，将结果进行筛选（既比较id列和user_id 列的值是否相等，将不相等的删除）。最后，得到两条符合条件的数据。
　　　　![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/1104082-20170226022446179-827444722.png)



**exists**

先遍历循环外表，然后看外表中的记录有没有和内表的数据一样的。匹配上就将结果放入结果集中



```
 1 SELECT
 2     `user`.*
 3 FROM
 4     `user`
 5 WHERE
 6     EXISTS (
 7         SELECT
 8             `order`.user_id
 9         FROM
10             `order`
11         WHERE
12             `user`.id = `order`.user_id
13     )
```



　　　　这条sql语句的执行结果和上面的in的执行结果是一样的。

　　　　![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/1104082-20170226022911710-1188849130.png)

　　　　但是，不一样的是它们的执行流程完全不一样：

　　　　使用exists关键字进行查询的时候，首先，我们先查询的不是子查询的内容，而是查我们的主查询的表，也就是说，我们先执行的sql语句是：

　　　　 SELECT `user`.* FROM `user` 

　　　　得到的结果如下：

　　　　![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/1104082-20170226023222945-1965432269.png)

　　　　然后，根据表的每一条记录，执行以下语句，依次去判断where后面的条件是否成立：



```
EXISTS (
        SELECT
            `order`.user_id
        FROM
            `order`
        WHERE
            `user`.id = `order`.user_id
    )
```

　　　　如果成立则返回true不成立则返回false。如果返回的是true的话，则该行结果保留，如果返回的是false的话，则删除该行，最后将得到的结果返回。



**区别及应用场景**

驱动表的不同，in在内层是驱动表，exist驱动表在外层

因此，里层的表是大表的时候建议用in，里层的表是小表的时候建议用exist









### 1.2、ORDER BY 优化

> **ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序**

**创建表**

- 建表 SQL

```mysql
create table tblA(
    #id int primary key not null auto_increment,
    age int,
    birth timestamp not null
);

insert into tblA(age, birth) values(22, now());
insert into tblA(age, birth) values(23, now());
insert into tblA(age, birth) values(24, now());

create index idx_A_ageBirth on tblA(age, birth);
```

----

**CASE1：能使用索引进行排序的情况**

- 只有带头大哥 age

```mysql
EXPLAIN SELECT * FROM tblA where age>20 order by age;

EXPLAIN SELECT * FROM tblA where birth>'2016-01-28 00:00:00' order by age;
```

![image-20210716101733914](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716101733914.png)

![image-20210716101746948](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716101746948.png)

- 带头大哥 age + 小弟 birth

```mysql
EXPLAIN SELECT * FROM tblA where age>20 order by age,birth;
```

![image-20210716101835175](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716101835175.png)



- mysql 默认升序排列，全升序或者全降序，都扛得住

```mysql
EXPLAIN SELECT * FROM tblA ORDER BY age ASC, birth ASC;

EXPLAIN SELECT * FROM tblA ORDER BY age DESC, birth DESC;
```



------

**CASE2：不能使用索引进行排序的情况**

- 带头大哥 age 挂了

```mysql
EXPLAIN SELECT * FROM tblA where age>20 order by birth;
```

![image-20210716101948086](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716101948086.png)



- 小弟 birth 居然敢在带头大哥 age 前面

```mysql
EXPLAIN SELECT * FROM tblA where age>20 order by birth,age;
```

![image-20210716102014531](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716102014531.png)

- mysql 默认升序排列，如果全升序或者全降序，都 ok ，但是一升一降就不行！

```mysql
EXPLAIN SELECT * FROM tblA ORDER BY age ASC, birth DESC;
```

![image-20210716102051203](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716102051203.png)

---



**结论**

1. MySQL支持二种方式的排序，FileSort和Index，**Index效率高，它指MySQL扫描索引本身完成排序**，FileSort方式效率较低。

2. ORDER BY满足两情况（最佳左前缀原则），会使用Index方式排序
   - ORDER BY语句使用索引最左前列
   - 使用where子句与OrderBy子句条件列组合满足索引最左前列
   
3. 尽可能在索引列上完成排序操作，遵照索引建的**最佳左前缀**

   

> **如果未在索引列上完成排序，mysql 会启动 filesort 的两种算法：双路排序和单路排序**



**filesort的两种算法**

1. 双路排序

   ​	 扫描两次磁盘

   读取行指针和将要进行orderby操作的列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据传输

   - 从磁盘取排序字段，在buffer进行排序，再从磁盘取其他字段。

     总结：先将要排序的字段取出来order by，再将排好序的结果按照需要去读取数据行

2. 单路排序

   ​	扫描一次磁盘

   - 从磁盘读取查询需要的所有列，按照将要进行orderby的列，在sort buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据，并且把随机IO变成顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了。

     总结：先读取查询所需要的所有列，然后再根据给定列进行排序，最后直接返回排序结果
     
     

**结论及引申出的问题**：

由于单路是改进的算法，总体而言好过双路但是单路排序存在问题



**更深层次的优化策略**：

- 增大sort_buffer_size参数的设置

- 增大max_length_for_sort_data参数的设置

- 减少select后面的查询字段

  ![image-20210716105924847](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716105924847.png)
  
  

------



> **Order By 排序索引优化的总结**![image-20210716110633805](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716110633805.png)

### 1.3、GROUP BY 优化

> **group by关键字优化**

![image-20210716110745412](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716110745412.png)	

​	group by使用索引的原则几乎跟orderby一致，唯一区别是groupby即使没有过滤条件用到索引，也可以直接使用索引。



## 慢查询日志

### 2.1、慢查询日志介绍

> **慢查询日志是什么？**

1. MySQL的慢查询日志是MySQL提供的一种日志记录，它用来**记录在MySQL中响应时间超过阀值的语句**，具体指运行时间超过**long_query_time**值的SQL，则会被记录到慢查询日志中。
2. long_query_time的默认值为10，意思是运行10秒以上的SQL语句会被记录下来
3. 由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析。

### 2.2、慢查询日志开启

> **怎么玩？**

**说明：**

1. 默认情况下，MySQL数据库没有开启慢查询日志，需要我们手动来设置这个参数。
2. 当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支持将日志记录写入文件

------

**查看是否开启及如何开启**

- 查看慢查询日志是否开启：
  - 默认情况下slow_query_log的值为OFF，表示慢查询日志是禁用的
  - 可以通过设置**slow_query_log**的值来开启
  - 通过`SHOW VARIABLES LIKE '%slow_query_log%';`查看 mysql 的慢查询日志是否开启

```sql
mysql> SHOW VARIABLES LIKE '%slow_query_log%';
+---------------------+-------------------------------+
| Variable_name       | Value                         |
+---------------------+-------------------------------+
| slow_query_log      | OFF                           |
| slow_query_log_file | /var/lib/mysql/Heygo-slow.log |
+---------------------+-------------------------------+
```

- 如何开启开启慢查询日志：
  - `set global slow_query_log = 1;`开启慢查询日志
  - 使用`set global slow_query_log=1`开启了慢查询日志只对当前数据库生效，如果MySQL重启后则会失效。

```sql
mysql> set global slow_query_log = 1;
Query OK, 0 rows affected (0.07 sec)

mysql> SHOW VARIABLES LIKE '%slow_query_log%';
+---------------------+-------------------------------+
| Variable_name       | Value                         |
+---------------------+-------------------------------+
| slow_query_log      | ON                            |
| slow_query_log_file | /var/lib/mysql/Heygo-slow.log |
+---------------------+-------------------------------+	
```

- 如果要永久生效，就必须修改配置文件my.cnf（其它系统变量也是如此）

  - 修改my.cnf文件，[mysqld]下增加或修改参数：slow_query_log和slow_query_log_file后，然后重启MySQL服务器。
  - 也即将如下两行配置进my.cnf文件

  ```
  [mysqld]
  slow_query_log =1
  slow_query_log_file=/var/lib/mysql/Heygo-slow.log
  ```
  
- 关于慢查询的参数slow_query_log_file，它指定慢查询日志文件的存放路径，系统默认会给一个缺省的文件host_name-slow.log（如果没有指定参数slow_query_log_file的话）

------

**那么开启慢查询日志后，什么样的SQL参会记录到慢查询里面？**

- 这个是由参数long_query_time控制，默认情况下long_query_time的值为10秒，命令：`SHOW VARIABLES LIKE 'long_query_time%';`查看慢 SQL 的阈值

```
mysql> SHOW VARIABLES LIKE 'long_query_time%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```

- 可以使用命令修改，也可以在my.cnf参数里面修改。
- 假如运行时间大于long_query_time会被记录，等于不会被记录

### 2.3、慢查询日志示例

> **案例讲解**

- 设置慢 SQL 的阈值时间，我们将其设置为 3s

```
mysql> set global long_query_time=3;
Query OK, 0 rows affected (0.00 sec)
```

**注：需要重新打卡会话才能看到自己重新设置的阈值**

```sql
mysql> set global long_query_time=3;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'long_query_time%';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 3.000000 |
+-----------------+----------+
```

- 记录慢 SQL 以供后续分析

  

  查看慢查询日志中的内容，`SHOW VARIABLES LIKE '%slow_query_log%';`根据自己电脑中的显示的文件地址查

  

- 查询当前系统中有多少条慢查询记录：`show global status like '%Slow_queries%';`

```
mysql> show global status like '%Slow_queries%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 1     |
+---------------+-------+
```

------

**配置版的慢查询日志**

在 /etc/my.cnf 文件的 [mysqld] 节点下配置

```
slow_query_log=1；
slow_query_log_file=/var/lib/mysql/Heygo-slow.log 
long_query_time=3；
log_output=FILE
```

> **日志分析命令 mysqldumpslow**

**mysqldumpslow是什么？**

在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具mysqldumpslow。

------

**查看 mysqldumpslow的帮助信息**

```
[root@Heygo mysql]# mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                al: average lock time
                ar: average rows sent
                at: average query time
                 c: count
                 l: lock time
                 r: rows sent
                 t: query time  
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
```

------

**mysqldumpshow 参数解释**

1. s：是表示按何种方式排序
2. c：访问次数
3. l：锁定时间
4. r：返回记录
5. t：查询时间
6. al：平均锁定时间
7. ar：平均返回记录数
8. at：平均查询时间
9. t：即为返回前面多少条的数据
10. g：后边搭配一个正则匹配模式，大小写不敏感的

------

**常用参数手册**

1. 得到返回记录集最多的10个SQL

   ```mysql
   mysqldumpslow -s r -t 10 /var/lib/mysql/Heygo-slow.log
   ```
   
2. 得到访问次数最多的10个SQL

   ```mysql
   mysqldumpslow -s c- t 10/var/lib/mysql/Heygo-slow.log
   ```
   
3. 得到按照时间排序的前10条里面含有左连接的查询语句

   ```mysql
   mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/Heygo-slow.log
   ```
   
4. 另外建议在使用这些命令时结合 | 和more使用，否则有可能出现爆屏情况

   ```mysql
   mysqldumpslow -s r -t 10 /var/lib/mysql/Heygo-slow.log | more
   ```



## Show Profile

> **Show Profile 是什么？**

1. 是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优测量
2. 官网：http://dev.mysql.com/doc/refman/5.5/en/show-profile.html
3. 默认情况下，参数处于关闭状态，并保存最近15次的运行结果

> **分析步骤**

**查看是当前的SQL版本是否支持Show Profile**

- show variables like ‘profiling%’; 查看 Show Profile 是否开启

```mysql
show variables like 'profiling%';
```

![image-20210716114326955](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716114326955.png)

------

**开启功能 Show Profiles**,我这里已经默认开启了

- `set profiling=on;` 开启 Show Profile



------

**运行SQL**

- 正常 SQL

```mysql
select * from tbl_emp;
select * from tbl_emp e inner join tbl_dept d on e.deptId = d.id;
select * from tbl_emp e left join tbl_dept d on e.deptId = d.id;
```

- 慢 SQL

```mysql
select * from emp group by id%10 limit 150000;
select * from emp group by id%10 limit 150000;
select * from emp group by id%10 order by 5;
```

------

**查看结果**

- 通过 show profiles; 指令查看结果

```mysql
show profiles;
```



------

**诊断SQL**

- `show profile cpu, block io for query SQL编号;` 查看 SQL 语句执行的具体流程以及每个步骤花费的时间

```
mysql> show profile cpu, block io for query 2;
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
| starting             | 0.000055 | 0.000000 |   0.000000 |            0 |             0 |
| checking permissions | 0.000007 | 0.000000 |   0.000000 |            0 |             0 |
| Opening tables       | 0.000011 | 0.000000 |   0.000000 |            0 |             0 |
| init                 | 0.000024 | 0.000000 |   0.000000 |            0 |             0 |
| System lock          | 0.000046 | 0.000000 |   0.000000 |            0 |             0 |
| optimizing           | 0.000018 | 0.000000 |   0.000000 |            0 |             0 |
| statistics           | 0.000008 | 0.000000 |   0.000000 |            0 |             0 |
| preparing            | 0.000019 | 0.000000 |   0.000000 |            0 |             0 |
| executing            | 0.000003 | 0.000000 |   0.000000 |            0 |             0 |
| Sending data         | 0.000089 | 0.000000 |   0.000000 |            0 |             0 |
| end                  | 0.000004 | 0.000000 |   0.000000 |            0 |             0 |
| query end            | 0.000003 | 0.000000 |   0.000000 |            0 |             0 |
| closing tables       | 0.000005 | 0.000000 |   0.000000 |            0 |             0 |
| freeing items        | 0.000006 | 0.000000 |   0.000000 |            0 |             0 |
| cleaning up          | 0.000006 | 0.000000 |   0.000000 |            0 |             0 |
+----------------------+----------+----------+------------+--------------+---------------+
15 rows in set, 1 warning (0.00 sec)
```

- 参数备注：

1. ALL：显示所有的开销信息
2. BLOCK IO：显示块IO相关开销
3. CONTEXT SWITCHES：上下文切换相关开销
4. CPU：显示CPU相关开销信息
5. IPC：显示发送和接收相关开销信息
6. MEMORY：显示内存相关开销信息
7. PAGE FAULTS：显示页面错误相关开销信息
8. SOURCE：显示和Source_function，Source_file，Source_line相关的开销信息
9. SWAPS：显示交换次数相关开销的信息

------

**日常开发需要注意的结论**

1. converting HEAP to MyISAM：查询结果太大，内存都不够用了往磁盘上搬了。
2. Creating tmp table：创建临时表，mysql 先将拷贝数据到临时表，然后用完再将临时表删除
3. Copying to tmp table on disk：把内存中临时表复制到磁盘，危险！！！
4. locked：锁表

------

举例

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716114632075.png" alt="image-20210716114632075" style="zoom:50%;" />

## 全局查询日志

**永远不要在生产环境开启这个功能。**

> **配置启用全局查询日志**

- 在mysql的my.cnf中，设置如下：

```properties
# 开启
general_log=1

# 记录日志文件的路径
general_log_file=/path/logfile

# 输出格式
log_output=FILE
```

> **编码启用全局查询日志**

- 执行如下指令开启全局查询日志

```mysql
set global general_log=1;
set global log_output='TABLE';
```

- 此后，你所执行的sql语句，将会记录到mysql库里的general_log表，可以用下面的命令查看

```mysql
select * from mysql.general_log;
```



# mysql的锁机制



**mysql中的乐观锁和悲观锁**

​	乐观锁并不是数据库自带的，需要手动实现乐观锁，实现的方式和CAS相同，就是通过比较和交换而完成的，一般情况下，我们会在表中新增一个version字段，每次更新数据version+1,在进行提交之前会判断version是否一致。

注：mysql中的绝大部分锁都是悲观锁，按照粒度可以分为行锁和表锁：

**举例：通过自旋方式实现mysql的乐观锁更新**

```xml
<!-- 乐观锁更新 -->
<update id="updateCount">
    update
    goods_sale
    set count = #{record.count}, data_version = data_version + 1
    where goods_sale_id = #{record.goodsSaleId}
    and data_version = #{record.dataVersion}
</update>
```



```java
public void addCount(String goodsId, Integer count) {
    while(true) {
        GoodsSale goodsSale = dao.selectByGoodsId(goodsId);
        if (goodsSale == null) {
            throw new Execption("数据不存在");
        }
        int count = goodsSale.getCount() + count;
        goodsSale.setCount(count);
        int count = dao.updateCount(goodsSale);
        if (count > 0) {
            return;
        }   
    }
}
```



## 

### 1、MySQL锁的基本介绍

​		**锁是计算机协调多个进程或线程并发访问某一资源的机制。**在数据库中，除传统的 计算资源（如CPU、RAM、I/O等）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一 个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

​		相对其他数据库而言，MySQL的锁机制比较简单，其最 显著的特点是不同的**存储引擎**支持不同的锁机制。比如，MyISAM和MEMORY存储引擎采用的是表级锁（table-level locking）；InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁。 

​		**表级锁：**开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。 
​		**行级锁：**开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。  

​		从上述特点可见，很难笼统地说哪种锁更好，只能就具体应用的特点来说哪种锁更合适！仅从锁的角度 来说：表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如Web应用；而行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有 并发查询的应用，如一些在线事务处理（OLTP）系统。 

### 2、MyISAM表锁

MySQL的表级锁有两种模式：**表共享读锁（Table Read Lock）**和**表独占写锁（Table Write Lock）**。  

对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作；MyISAM表的读操作与写操作之间，以及写操作之间是串行的！ 

建表语句：

```sql
CREATE TABLE `mylock` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `NAME` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

INSERT INTO `mylock` (`id`, `NAME`) VALUES ('1', 'a');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('2', 'b');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('3', 'c');
INSERT INTO `mylock` (`id`, `NAME`) VALUES ('4', 'd');
```

**MyISAM写锁阻塞读的案例：**

​		当一个线程获得对一个表的写锁之后，只有持有锁的线程可以对表进行更新操作。其他线程的读写操作都会等待，直到锁释放为止。

| session1                                                     |                         session2                          |
| ------------------------------------------------------------ | :-------------------------------------------------------: |
| 获取表的write锁定<br />lock table mylock write;              |                                                           |
| 当前session对表的查询，插入，更新操作都可以执行<br />select * from mylock;<br />insert into mylock values(5,'e'); | 当前session对表的查询会被阻塞<br />select * from mylock； |
| 释放锁：<br />unlock tables；                                |          当前session能够立刻执行，并返回对应结果          |

**MyISAM读阻塞写的案例：**

​		一个session使用lock table给表加读锁，这个session可以锁定表中的记录，但更新和访问其他表都会提示错误，同时，另一个session可以查询表中的记录，但更新就会出现锁等待。

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|        获得表的read锁定<br />lock table mylock read;         |                                                              |
|   当前session可以查询该表记录：<br />select * from mylock;   |   当前session可以查询该表记录：<br />select * from mylock;   |
| 当前session不能查询没有锁定的表<br />select * from person<br />Table 'person' was not locked with LOCK TABLES | 当前session可以查询或者更新未锁定的表<br />select * from mylock<br />insert into person values(1,'zhangsan'); |
| 当前session插入或者更新表会提示错误<br />insert into mylock values(6,'f')<br />Table 'mylock' was locked with a READ lock and can't be updated<br />update mylock set name='aa' where id = 1;<br />Table 'mylock' was locked with a READ lock and can't be updated | 当前session插入数据会等待获得锁<br />insert into mylock values(6,'f'); |
|                  释放锁<br />unlock tables;                  |                       获得锁，更新成功                       |

### 注意:

**MyISAM在执行查询语句之前，会自动给涉及的所有表加读锁，在执行更新操作前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此用户一般不需要使用命令来显式加锁，上例中的加锁时为了演示效果。**

**MyISAM的并发插入问题**

MyISAM表的读和写是串行的，这是就总体而言的，在一定条件下，MyISAM也支持查询和插入操作的并发执行

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|   获取表的read local锁定<br />lock table mylock read local   |                                                              |
| 当前session不能对表进行更新或者插入操作<br />insert into mylock values(6,'f')<br />Table 'mylock' was locked with a READ lock and can't be updated<br />update mylock set name='aa' where id = 1;<br />Table 'mylock' was locked with a READ lock and can't be updated |    其他session可以查询该表的记录<br />select* from mylock    |
| 当前session不能查询没有锁定的表<br />select * from person<br />Table 'person' was not locked with LOCK TABLES | 其他session可以进行插入操作，但是更新会阻塞<br />update mylock set name = 'aa' where id = 1; |
|          当前session不能访问其他session插入的记录；          |                                                              |
|                  释放锁资源：unlock tables                   |               当前session获取锁，更新操作完成                |
|           当前session可以查看其他session插入的记录           |                                                              |

 可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺： 

```sql
mysql> show status like 'table%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 352   |
| Table_locks_waited    | 2     |
+-----------------------+-------+
--如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。
```

**InnoDB锁**

**1、事务及其ACID属性**

事务是由一组SQL语句组成的逻辑处理单元，事务具有4属性，通常称为事务的ACID属性。

原子性（Actomicity）：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
一致性（Consistent）：在事务开始和完成时，数据都必须保持一致状态。
隔离性（Isolation）：数据库系统提供一定的隔离机制，保证事务在不受外部并发操作影响的“独立”环境执行。
持久性（Durable）：事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。

**2、并发事务带来的问题**

相对于串行处理来说，并发事务处理能大大增加数据库资源的利用率，提高数据库系统的事务吞吐量，从而可以支持更多用户的并发操作，但与此同时，会带来一下问题：

**脏读**： 一个事务正在对一条记录做修改，在这个事务并提交前，这条记录的数据就处于不一致状态；这时，另一个事务也来读取同一条记录，如果不加控制，第二个事务读取了这些“脏”的数据，并据此做进一步的处理，就会产生未提交的数据依赖关系。这种现象被形象地叫做“脏读” 

**不可重复读**：一个事务在读取某些数据已经发生了改变、或某些记录已经被删除了！这种现象叫做“不可重复读”。 

**幻读**： 一个事务按相同的查询条件重新读取以前检索过的数据，却发现其他事务插入了满足其查询条件的新数据，这种现象就称为“幻读” 

上述出现的问题都是数据库读一致性的问题，可以通过事务的隔离机制来进行保证。

数据库的事务隔离越严格，并发副作用就越小，但付出的代价也就越大，因为事务隔离本质上就是使事务在一定程度上串行化，需要根据具体的业务需求来决定使用哪种隔离级别

|                  | 脏读 | 不可重复读 | 幻读 |
| :--------------: | :--: | :--------: | :--: |
| read uncommitted |  √   |     √      |  √   |
|  read committed  |      |     √      |  √   |
| repeatable read  |      |            |  √   |
|   serializable   |      |            |      |

 可以通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况： 

```sql
mysql> show status like 'innodb_row_lock%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 18702 |
| Innodb_row_lock_time_avg      | 18702 |
| Innodb_row_lock_time_max      | 18702 |
| Innodb_row_lock_waits         | 1     |
+-------------------------------+-------+
--如果发现锁争用比较严重，如InnoDB_row_lock_waits和InnoDB_row_lock_time_avg的值比较高
```

**3、InnoDB的行锁模式及加锁方法**

​		**共享锁（s）**：又称读锁。允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
​		**排他锁（x）**：又称写锁。允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。

​		mysql InnoDB引擎默认的修改数据语句：**update,delete,insert都会自动给涉及到的数据加上排他锁，select语句默认不会加任何锁类型**，如果加排他锁可以使用select …for update语句，加共享锁可以使用select … lock in share mode语句。**所以加过排他锁的数据行在其他事务种是不能修改数据的，也不能通过for update和lock in share mode锁的方式查询数据，但可以直接通过select …from…查询数据，因为普通查询没有任何锁机制。** 

**InnoDB行锁实现方式**

​		InnoDB行锁是通过给**索引**上的索引项加锁来实现的，这一点MySQL与Oracle不同，后者是通过在数据块中对相应数据行加锁来实现的。InnoDB这种行锁实现特点意味着：只有通过索引条件检索数据，InnoDB才使用行级锁，**否则，InnoDB将使用表锁！**  

1、在不通过索引条件查询的时候，innodb使用的是表锁而不是行锁

```sql
create table tab_no_index(id int,name varchar(10)) engine=innodb;
insert into tab_no_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| set autocommit=0<br />select * from tab_no_index where id = 1; | set autocommit=0<br />select * from tab_no_index where id =2 |
|      select * from tab_no_index where id = 1 for update      |                                                              |
|                                                              |     select * from tab_no_index where id = 2 for update;      |

session1只给一行加了排他锁，但是session2在请求其他行的排他锁的时候，会出现锁等待。原因是在没有索引的情况下，innodb只能使用表锁。

2、创建带索引的表进行条件查询，innodb使用的是行锁

```sql
create table tab_with_index(id int,name varchar(10)) engine=innodb;
alter table tab_with_index add index id(id);
insert into tab_with_index values(1,'1'),(2,'2'),(3,'3'),(4,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| set autocommit=0<br />select * from tab_with_indexwhere id = 1; | set autocommit=0<br />select * from tab_with_indexwhere id =2 |
|     select * from tab_with_indexwhere id = 1 for update      |                                                              |
|                                                              |     select * from tab_with_indexwhere id = 2 for update;     |

此时没有出现所等待情况。

3、由于mysql的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是依然无法访问到具体的数据

```sql
insert into tab_with_index  values(1,'4');
```

|                           session1                           |                           session2                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                       set autocommit=0                       |                       set autocommit=0                       |
| select * from tab_with_index where id = 1 and name='1' for update |                                                              |
|                                                              | select * from tab_with_index where id = 1 and name='4' for update<br /> |

**虽然session2访问的是和session1不同的记录，但是因为使用了相同的索引，所以需要等待锁**



### 总结

**对于MyISAM的表锁，主要讨论了以下几点：** 
（1）共享读锁（S）之间是兼容的，但共享读锁（S）与排他写锁（X）之间，以及排他写锁（X）之间是互斥的，也就是说读和写是串行的。  
（2）在一定条件下，MyISAM允许查询和插入并发执行，我们可以利用这一点来解决应用中对同一表查询和插入的锁争用问题。 
（3）MyISAM默认的锁调度机制是写优先，这并不一定适合所有应用，用户可以通过设置LOW_PRIORITY_UPDATES参数，或在INSERT、UPDATE、DELETE语句中指定LOW_PRIORITY选项来调节读写锁的争用。 
（4）由于表锁的锁定粒度大，读写之间又是串行的，因此，如果更新操作较多，MyISAM表可能会出现严重的锁等待，可以考虑采用InnoDB表来减少锁冲突。

**对于InnoDB表，本文主要讨论了以下几项内容：** 
（1）InnoDB的行锁是基于索引实现的，如果不通过索引访问数据，InnoDB会使用表锁。 
（2）在不同的隔离级别下，InnoDB的锁机制和一致性读策略不同。

在了解InnoDB锁特性后，用户可以通过设计和SQL调整等措施减少锁冲突和死锁，包括：

- 尽量使用较低的隔离级别； 精心设计索引，并尽量使用索引访问数据，使加锁更精确，从而减少锁冲突的机会；
- 选择合理的事务大小，小事务发生锁冲突的几率也更小；
- 给记录集显式加锁时，最好一次性请求足够级别的锁。比如要修改数据的话，最好直接申请排他锁，而不是先申请共享锁，修改时再请求排他锁，这样容易产生死锁；
- 不同的程序访问一组表时，应尽量约定以相同的顺序访问各表，对一个表而言，尽可能以固定的顺序存取表中的行。这样可以大大减少死锁的机会；
- 尽量用相等条件访问数据，这样可以避免间隙锁对并发插入的影响； 不要申请超过实际需要的锁级别；除非必须，查询时不要显示加锁；
- 对于一些特定的事务，可以使用表锁来提高处理速度或减少死锁的可能。





# 第 5 章 主从复制



## binlog日志文件



### binlog日志的介绍

最开始的mysql没有innodb引擎，innodb引擎是其他公司以插件形式插入mysql的。mysql自带的引擎是myisam，但是我们知道redolog是innodb引擎特有的，其他存储引擎都没有，这就会导致没有cash-safe的能力（cash-safe指即使数据库发生异常重启，之前提交的记录都不会丢失）。

**binlog 是逻辑日志，记录的是这个语句的原始逻辑**，比如 “ 给 ID=2 这一行的 c 字段加 1 ”。

binlog 记录的都是事务操作内容，binlog 有三种模式：

- Statement（基于 SQL 语句的复制）
- Row（基于行的复制） 
- 以及 Mixed（混合模式）

### binlog日志的使用

查看binlog是否开启以及binlog目录

```sql
SHOW GLOBAL VARIABLES like 'log_bin%'
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716153401060.png" alt="image-20210716153401060" style="zoom:50%;" />

**binlog日志格式**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716153623393.png" alt="image-20210716153623393" style="zoom:50%;" />

查看binlog的二进制日志格式

```sql
SHOW GLOBAL VARIABLES LIKE 'binlog_format%'
```

![image-20210716160249517](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716160249517.png)



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723215626458.png" alt="image-20210723215626458" style="zoom:50%;" />

## redolog、undolog日志文件

### redolog的介绍

binlog是mysql的server层特有的日志 （归档日志） 是**逻辑日志**，**而redo log 是InnoDB 引擎特有的日志，也是物理日志**，记录的是 “ 在某个数据页上做了什么修改 ”

### redolog的作用

redolog和binlog都可以做数据的恢复

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723164154685.png" alt="image-20210723164154685" style="zoom:50%;" />



### redolog的两阶段提交

redolog 的写入拆成了两个步骤： **prepare 和commit** ，这就是 **" 两阶段提交 "**。要注意：

**redo log prepare：** 是“ 当前事务提交 ” 的一个阶段，也就是说，在事务 A 提交的时候，我们才会走到事务 A 的 redo log prepare 这个阶段。

由于 redo log 和 binlog 是两个独立的逻辑，如果不用两阶段提交，要么就是先写完 redo log 再写binlog ，或者采用反过来的顺序，会产生日志与数据不一致的问题，例如

- 先写redolog直接提交，然后写binlog，假设写完redolog后，机器挂了，binlog日志没有被写入，那么机器重启后，这台机器本机可以通过redolog恢复数据，但是binlog并没有记录该数据，后续进行数据备份的时候，造成数据不同步的问题。
- 先写binlog，然后写redolog，假设写完了binlog，机器异常重启了，由于没有redolog，本机是无法恢复这条记录的，但是binlog又有记录，后序进行机器备份的时候，造成数据不同步的问题。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210912102858896.png" alt="image-20210912102858896" style="zoom:50%;" />

redolog和binlog是预写日志，redolog和binlog都是顺序io，数据更新的操作是随机io，因此写入redolog和binlog的速度要比数据更新要快

因此当更新数据失败时，需要通过预写日志对其进行恢复



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210909161053475.png" alt="image-20210909161053475" style="zoom:50%;" />

时序上 redo log 先 prepare ， 再写 binlog ，最后再把 redo log commit。

由于两个日志是分开写的，所以很难保证两个日志数据一致，在恢复的时候尽量保证同时参考两个日志文件，如果一致才提交
不一致直接丢弃。

如果写完redolog，此时redolog处于prepare状态，在写binlog前发生异常：判断binlog和redolog是否一致（一定不一致）不一致就会回滚

如果写完binlog，此时redolog处于prepare状态，在事务提交前发生异常：判断binlog和redolog是否一致。如果一致就提交，不一致回滚

**因此通过两阶段提交保证数据的一致性**



### undolog的作用

undolog是一个很牛逼的用来支持事务的日志，undolog又称作**回滚日志**。**它里面记录了每个当前行的历史版本数据。**

在数据修改的时候，不仅记录了redo，还记录了相对应的undo，如果因为某些原因导致事务失败或回滚了，可以借助该undo进行回滚。假如我们的事务中有5条sql，其中有2条执行失败了，3条执行成功了，mysql就会通过undolog将数据回滚到最初状态，保证了事务的原子性。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723165042051.png" alt="image-20210723165042051" style="zoom:39%;" />



## MVCC多版本并发控制

### 1、MVCC

​		MVCC，全称Multi-Version Concurrency Control，即多版本并发控制。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。

 		MVCC在MySQL InnoDB中的实现主要是为了提高数据库并发性能，用更好的方式去处理读写冲突，做到即使有读写冲突时，也能做到不加锁，非阻塞并发读。

### 2、当前读

​		**像select lock in share mode(共享锁), select for update ; update, insert ,delete(排他锁) 修改插入删除这些操作都是一种当前读**，为什么叫当前读？就是它读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。



### 3、快照读（提高数据库的并发查询能力）

​		**像不加锁的select操作就是快照读**，即不加锁的非阻塞读；快照读的前提是隔离级别不是串行级别，串行级别下的快照读会退化成当前读；之所以出现快照读的情况，是基于提高并发性能的考虑，**快照读的实现是基于多版本并发控制，即MVCC**,可以认为MVCC是行锁的一个变种，但它在很多情况下，避免了加锁操作，降低了开销；既然是基于多版本，即快照读可能读到的并不一定是数据的最新版本，而有可能是之前的历史版本

### 4、当前读、快照读、MVCC关系

​		MVCC多版本并发控制指的是维持一个数据的多个版本，使得读写操作没有冲突，快照读是MySQL为实现MVCC的一个非阻塞读功能。MVCC模块在MySQL中的具体实现是**由三个隐式字段，undo日志、read view三个组件来实现的。**

### 5、MVCC解决的问题

​		数据库并发场景有三种，分别为：

​		1、读读：不存在任何问题，也不需要并发控制

​		2、读写：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读、幻读、不可重复读

​		3、写写：有线程安全问题，可能存在更新丢失问题

​		MVCC是一种用来解决读写冲突的无锁并发控制，也就是为事务分配单项增长的时间戳，**为每个修改保存一个版本，版本与事务时间戳关联**，读操作只读该事务开始前的数据库的快照，所以MVCC可以为数据库解决一下问题：

​		1、在并发读写数据库时，可以做到在读操作时不用阻塞写操作，写操作也不用阻塞读操作，提高了数据库并发读写的性能

​		2、解决脏读、幻读、不可重复读等事务隔离问题，但是不能解决更新丢失问题

### 6、MVCC实现原理

mvcc的实现原理主要依赖于记录中的三个隐藏字段，undolog，read view来实现的。

**隐藏字段**

除了我们自定义的字段外还有隐藏字段

- DB_TRX_ID (事务id)：6字节，最近修改事务id，记录创建这条记录或者最后一次修改该记录的事务id
- DB_ROLL_PTR （回滚指针）:	7字节，回滚指针，指向这条记录的上一个版本,用于配合undolog，指向上一个旧版本
- DB_ROW_JD:	6字节，隐藏的主键，如果数据表没有主键，那么innodb会自动生成一个6字节的row_id

​		记录如图所示：

![image-20210225233929554](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/%E6%95%B0%E6%8D%AE%E6%A1%88%E4%BE%8B.png)

​		在上图中，DB_ROW_ID是数据库默认为该行记录生成的唯一隐式主键，DB_TRX_ID是当前操作该记录的事务ID，DB_ROLL_PTR是一个回滚指针，用于配合undo日志，指向上一个旧版本

​		**undo log**

​		undolog被称之为回滚日志，表示在进行insert，delete，update操作的时候产生的方便回滚的日志

​		当进行insert操作的时候，产生的undolog只在事务回滚的时候需要，并且在事务提交之后可以被立刻丢弃

​		当进行update和delete操作的时候，产生的undolog不仅仅在事务回滚的时候需要，在快照读的时候也需要，所以不能随便删除，只有在快照读或事务回滚不涉及该日志时，对应的日志才会被purge线程统一清除（当数据发生更新和删除操作的时候都只是设置一下老记录的deleted_bit，并不是真正的将过时的记录删除，因为为了节省磁盘空间，innodb有专门的purge线程来清除deleted_bit为true的记录，如果某个记录的deleted_id为true，并且DB_TRX_ID相对于purge线程的read view 可见，那么这条记录一定时可以被清除的）

​		**下面我们来看一下undolog生成的记录链**

​		1、假设有一个事务编号为1的事务向表中插入一条记录，那么此时行数据的状态为：

![image-20210225235444975](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/1.png)

​		2、假设有第二个事务编号为2对该记录的name做出修改，改为lisi

​		在事务2修改该行记录数据时，数据库会对该行加排他锁

​		然后把该行数据拷贝到undolog中，作为 旧记录，即在undolog中有当前行的拷贝副本

​		拷贝完毕后，修改该行name为lisi，并且修改隐藏字段的事务id为当前事务2的id，回滚指针指向拷贝到undolog的副本记录中

​		事务提交后，释放锁

![image-20210313220450629](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/2.png)

​		3、假设有第三个事务编号为3对该记录的age做了修改，改为32

​		在事务3修改该行数据的时，数据库会对该行加排他锁

​		然后把该行数据拷贝到undolog中，作为旧纪录，发现该行记录已经有undolog了，那么最新的旧数据作为链表的表头，插在该行记录的undolog最前面

​		修改该行age为32岁，并且修改隐藏字段的事务id为当前事务3的id，回滚指针指向刚刚拷贝的undolog的副本记录

​		事务提交，释放锁

![image-20210313220337624](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/3.png)

​		从上述的一系列图中，大家可以发现，不同事务或者相同事务的对同一记录的修改，会导致该记录的undolog生成一条记录版本线性表，即链表，undolog的链首就是最新的旧记录，链尾就是最早的旧记录。

​		**Read View**

​		上面的流程如果看明白了，那么大家需要再深入理解下read view的概念了。

​		Read View是事务进行快照读操作的时候生产的读视图，在该事务执行快照读的那一刻，会生成一个数据系统当前的快照，记录并维护系统当前活跃事务的id，事务的id值是递增的。

​		其实Read View的最大作用是用来做可见性判断的，也就是说当某个事务在执行快照读的时候，对该记录创建一个Read View的视图，把它当作条件去判断当前事务能够看到哪个版本的数据，有可能读取到的是最新的数据，也有可能读取的是当前行记录的undolog中某个版本的数据

​		Read View遵循的可见性算法主要是将要被修改的数据的最新记录中的DB_TRX_ID（当前事务id）取出来，与系统当前其他活跃事务的id去对比，如果DB_TRX_ID跟Read View的属性做了比较，不符合可见性，那么就通过DB_ROLL_PTR回滚指针去取出undolog中的DB_TRX_ID做比较，即遍历链表中的DB_TRX_ID，直到找到满足条件的DB_TRX_ID,这个DB_TRX_ID所在的旧记录就是当前事务能看到的最新老版本数据。

​		Read View的可见性规则如下所示：

​		首先要知道Read View中的三个全局属性：

​		trx_list:一个数值列表，用来维护Read View生成时刻系统正活跃的事务ID（1,2,3）

​		up_limit_id:记录trx_list列表中事务ID最小的ID（1）

​		low_limit_id:Read View生成时刻系统尚未分配的下一个事务ID，（4）

​		具体的比较规则如下：

​		1、首先比较DB_TRX_ID < up_limit_id,如果小于，则当前事务能看到DB_TRX_ID所在的记录，如果大于等于进入下一个判断

​		2、接下来判断DB_TRX_ID >= low_limit_id,如果大于等于则代表DB_TRX_ID所在的记录在Read View生成后才出现的，那么对于当前事务肯定不可见，如果小于，则进入下一步判断

​		3、判断DB_TRX_ID是否在活跃事务中，如果在，则代表在Read View生成时刻，这个事务还是活跃状态，还没有commit，修改的数据，当前事务也是看不到，如果不在，则说明这个事务在Read View生成之前就已经开始commit，那么修改的结果是能够看见的。

### 7、MVCC的整体处理流程

假设有四个事务同时在执行，如下图所示：

|  事务1   |  事务2   |  事务3   |    事务4     |
| :------: | :------: | :------: | :----------: |
| 事务开始 | 事务开始 | 事务开始 |   事务开始   |
|  ......  |  ......  |  ......  | 修改且已提交 |
|  进行中  |  快照读  |  进行中  |              |
|  ......  |  ......  |  ......  |              |

从上述表格中，我们可以看到，当事务2对某行数据执行了快照读，数据库为该行数据生成一个Read View视图，可以看到事务1和事务3还在活跃状态，事务4在事务2快照读的前一刻提交了更新，所以，在Read View中记录了系统当前活跃事务1，3，维护在一个列表中。同时可以看到up_limit_id的值为1，而low_limit_id为5，如下图所示：

![image-20210227183316573](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/4.png)

在上述的例子中，只有事务4修改过该行记录，并在事务2进行快照读前，就提交了事务，所以该行当前数据的undolog如下所示：

![image-20210227183849998](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/5.png)

​		当事务2在快照读该行记录的是，会拿着该行记录的DB_TRX_ID去跟up_limit_id,lower_limit_id和活跃事务列表进行比较，判读事务2能看到该行记录的版本是哪个。

​		具体流程如下：先拿该行记录的事务ID（4）去跟Read View中的up_limit_id相比较，判断是否小于，通过对比发现不小于，所以不符合条件，继续判断4是否大于等于low_limit_id,通过比较发现也不大于，所以不符合条件，判断事务4是否处理trx_list列表中，发现不再次列表中，那么符合可见性条件，所以事务4修改后提交的最新结果对事务2 的快照是可见的，因此，事务2读取到的最新数据记录是事务4所提交的版本，而事务4提交的版本也是全局角度的最新版本。如下图所示：

![image-20210227185820394](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/6.png)

当上述的内容都看明白了的话，那么大家就应该能够搞清楚这几个核心概念之间的关系了，下面我们讲一个不同的隔离级别下的快照读的不同。

### 8、RC读已提交、RR可重复级别下的InnoDB快照读有什么不同

​		**在RC隔离级别下，是每个快照读都会生成并获取最新的Read View,而在RR隔离级别下，则是同一个事务中的第一个快照读才会创建Read View，之后的快照读获取的都是同一个Read View.**



## querylog日志文件

记录mysql的操作信息

```sql
SHOW VARIABLES like 'general_log_file%' #查看querylog日志文件的存放位置信息
```



![image-20210716161318809](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716161318809.png)

## 什么是主从复制?

主从复制，是用来建立一个和主数据库完全一样的数据库环境，称为从数据库；主数据库一般是准实时的业务数据库



## 主从复制的作用（好处，或者说为什么要做主从）重点!

1、做数据的热备，作为后备数据库，主数据库服务器故障后，可切换到从数据库继续工作，避免数据丢失。
2、架构的扩展。业务量越来越大，I/O访问频率过高，单机无法满足，此时做多库的存储，降低磁盘I/O访问的频率，提高单个机器的I/O性能。
3、读写分离，使数据库能支撑更大的并发。在报表中尤其重要。由于部分报表sql语句非常的慢，导致锁表，影响前台服务。如果前台使用master，报表使用slave，那么报表sql将不会造成前台锁，保证了前台速度。



## 主从复制的原理
1.数据库有个bin-log二进制文件，记录了所有sql语句。
2.我们的目标就是把主数据库的bin-log文件的sql语句复制过来，并让其在从数据的relay-log重做日志文件中再执行一次这些sql语句即可。

下面的主从配置就是围绕这个原理配置，具体需要三个线程来操作：

1. **Binlog-Output-Thread:**每当有从库连接到主库的时候，主库都会创建一个线程然后发送binlog内容到从库。在从库里，当复制开始的时候，从库就会创建两个线程进行处理：
2. **从库IO-Thread:**当START SLAVE语句在从库开始执行之后，从库创建一个I/O线程，该线程连接到主库并请求主库发送binlog里面的更新记录到从库上。从库I/O线程读取主库的binlog输出线程发送的更新并拷贝这些更新到本地文件，其中包括relay log文件。
3. **从库的SQL-Thread:**从库创建一个SQL线程，这个线程读取从库I/O线程写到relay log的更新事件并执行。

**可以知道，对于每一个主从复制的连接，都有三个线程。拥有多个从库的主库为每一个连接到主库的从库创建一个binlog输出线程，每一个从库都有它自己的I/O线程和SQL线程。**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210718143840053.png" alt="image-20210718143840053" style="zoom:50%;" />

## 2、复制的基本原则

1. 每个slave只有一个master
2. 每个slave只能有一个唯一的服务器ID
3. 每个master可以有多个salve

## 3、复制最大问题

因为发生多次 IO， 存在延时问题

## 4、一主一从常见配置

> **前提：mysql 版本一致，主从机在同一网段下**

**ping 测试**

- Linux 中 ping Windows

- Windows 中 ping Linux



> **主机修改 my.ini 配置文件（Windows）**

主从都配置都在 [mysqld] 节点下，都是小写，以下是老师的配置文件

![image-20200812191912521](https://img-blog.csdnimg.cn/img_convert/c09828ad99fb06855b3ed9374a533f7a.png)

------

**以下两条为必须配置**

- 配置主机 id

```ini
server-id=1
```

- 启用二进制日志

```ini
log-bin=C:/Program Files (x86)/MySQL/MySQL Server 5.5/log-bin/mysqlbin
```

------

**以下为非必须配置**

- 启动错误日志

```ini
log-err=C:/Program Files (x86)/MySQL/MySQL Server 5.5/log-bin/mysqlerr
```

- 根目录

```ini
basedir="C:/Program Files (x86)/MySQL/MySQL Server 5.5/"
```

- 临时目录

```ini
tmpdir="C:/Program Files (x86)/MySQL/MySQL Server 5.5/"
```

- 数据目录

```ini
datadir="C:/Program Files (x86)/MySQL/MySQL Server 5.5/Data/"
```

- 主机，读写都可以

```init
read-only=0
```

- 设置不要复制的数据库

```ini
binlog-ignore-db=mysql
```

- 设置需要复制的数据

```ini
binlog-do-db=需要复制的主数据库名字
```

> **从机修改 my.cnf 配置文件（Linux）**

- 【必须】从服务器唯一ID

```ini
server-id=2
```

- 【可选】启用二进制文件

> **修改配置文件后的准备工作**

**因修改过配置文件，主机+从机都重启 mysql 服务**

- Windows

```bash
net stop mysql
net start mysql
```

- Linux

```bash
service mysqld restart
```

------

**主机从机都关闭防火墙**

- Windows 手动关闭防火墙
- 关闭虚拟机 linux 防火墙

```bash
service iptables stop
```

> **在 Windows 主机上简历账户并授权 slave**

- 创建用户， 并授权

```bash
GRANT REPLICATION SLAVE ON *.* TO '备份账号'@'从机器数据库 IP' IDENTIFIED BY '账号密码';

GRANT REPLICATION SLAVE ON *.* TO 'Heygo'@'192.168.152.129' IDENTIFIED BY '123456';
```

- 刷新权限信息

```bash
flush privileges;

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

- 查询 master 的状态，将 File 和 Position 记录下来，在启动 Slave 时需要用到这两个参数

```bash

mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      107 | mysql        |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

> **在 Linux 从机上配置需要复制的主机**

- 从机进行认证

```bash
CHANGE MASTER TO 
MASTER_HOST='主机 IP',
MASTER_USER='创建用户名',
MASTER_PASSWORD='创建的密码',
MASTER_LOG_FILE='File 名字',
MASTER_LOG_POS=Position数字;
123456
CHANGE MASTER TO 
MASTER_HOST='10.206.207.131',
MASTER_USER='Heygo',
MASTER_PASSWORD='123456',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=107;
```

- 启动从服务器复制功能

```bash
start slave;
```

- 查看从机复制功能是否启动成功：Slave_SQL_Running:Yes 和 Slave_IO_Running:Yes 说明从机连接主机成功

```bash
show slave status\G;
```

- 如何停止从服务复制功能

```bash
stop slave;
```

> **备注**

1. 从机连不上啊。。。一直显示 Slave_IO_Running: Connecting
2. 我 Windows 中的 mysql 版本为 5.5，Linux 中的 mysql 版本为 5.6



## 验证主从配置

```sql
SHOW GLOBAL STATUS LIKE	'innodb_rows%'; # 验证数据库中受影响的行数
```

![image-20210716215654251](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210716215654251.png)

通过查看主库和从库数据的变化，来分析主从配置是否生效



# 数据库设计的三大范式

为了建立冗余较小、结构合理的数据库，设计数据库时必须遵循一定的规则。在关系型数据库中这种规则就称为范式。范式是符合某一种设计要求的总结。要想设计一个结构合理的关系型数据库，必须满足一定的范式。

​         

在实际开发中最为常见的设计范式有三个：

**1．第一范式(确保每列保持原子性)**

第一范式是最基本的范式。如果数据库表中的所有字段值都是不可分解的原子值，就说明该数据库表满足了第一范式。

第一范式的合理遵循需要根据系统的实际需求来定。比如某些数据库系统中需要用到“地址”这个属性，本来直接将“地址”属性设计成一个数据库表的字段就行。但是如果系统经常会访问“地址”属性中的“城市”部分，那么就非要将“地址”这个属性重新拆分为省份、城市、详细地址等多个部分进行存储，这样在对地址中某一部分操作的时候将非常方便。这样设计才算满足了数据库的第一范式，如下表所示。

![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/2012040114023352.png)

上表所示的用户信息遵循了第一范式的要求，这样在对用户使用城市进行分类的时候就非常方便，也提高了数据库的性能。

​         

**2．第二范式(确保表中的每列都和主键相关)**

第二范式在第一范式的基础之上更进一层。第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。也就是说在一个数据库表中，一个表中只能保存一种数据，不可以把多种数据保存在同一张数据库表中。

比如要设计一个订单信息表，因为订单中可能会有多种商品，所以要将订单编号和商品编号作为数据库表的联合主键，如下表所示。

 **订单信息表**

![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/2012040114063976.png)

这样就产生一个问题：这个表中是以订单编号和商品编号作为联合主键。这样在该表中商品名称、单位、商品价格等信息不与该表的主键相关，而仅仅是与商品编号相关。所以在这里违反了第二范式的设计原则。

而如果把这个订单信息表进行拆分，把商品信息分离到另一个表中，把订单项目表也分离到另一个表中，就非常完美了。如下所示。

![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/2012040114082156.png)

这样设计，在很大程度上减小了数据库的冗余。如果要获取订单的商品信息，使用商品编号到商品信息表中查询即可。

​         

**3．第三范式(确保每列都和主键列直接相关,而不是间接相关)**

第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。

比如在设计一个订单数据表的时候，可以将客户编号作为一个外键和订单表建立相应的关系。而不可以在订单表中添加关于客户其它信息（比如姓名、所属公司等）的字段。如下面这两个表所示的设计就是一个满足第三范式的数据库表。

![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/2012040114105477.png)

这样在查询订单信息的时候，就可以使用客户编号来引用客户信息表中的记录，也不必在订单信息表中多次输入客户信息的内容，减小了数据冗余。

## 扩展：mysql中的utf8和utf8mb4有什么区别

utf8 是 Mysql 中的一种字符集，只支持最长三个字节的 UTF-8字符，也就是 Unicode 中的基本多文本平面。
Mysql 中的 utf8 为什么只支持持最长三个字节的 UTF-8字符呢？我想了一下，可能是因为 Mysql 刚开始开发那会，Unicode 还没有辅助平面这一说呢。那时候，Unicode 委员会还做着 “65535 个字符足够全世界用了”的美梦。Mysql 中的字符串长度算的是字符数而非字节数，对于 CHAR 数据类型来说，需要为字符串保留足够的长。当使用 utf8 字符集时，需要保留的长度就是 utf8 最长字符长度乘以字符串长度，所以这里理所当然的限制了 utf8 最大长度为 3，比如 CHAR(100) Mysql 会保留 300字节长度。至于后续的版本为什么不对 4 字节长度的 UTF-8 字符提供支持，我想一个是为了向后兼容性的考虑，还有就是基本多文种平面之外的字符确实很少用到。

要在 Mysql 中保存 4 字节长度的 UTF-8 字符，需要使用 utf8mb4 字符集，但只有 5.5.3 版本以后的才支持。我觉得，为了获取更好的兼容性，应该总是使用 utf8mb4 而非 utf8. 对于 CHAR 类型数据，utf8mb4 会多消耗一些空间，根据 Mysql 官方建议，使用 VARCHAR 替代 CHAR。

四、为什么要使用utf8mb4字符集
既然utf8应付日常使用完全没有问题，那为什么还要使用utf8mb4呢? 低版本的MySQL支持的utf8编码，最大字符长度为 3 字节，如果遇到 4 字节的字符就会出现错误了。三个字节的 UTF-8 最大能编码的 Unicode 字符是 0xFFFF，也就是 Unicode 中的基本多文平面（BMP）。也就是说，任何不在基本多文平面的 Unicode字符，都无法使用MySQL原有的 utf8 字符集存储。这些不在BMP中的字符包括哪些呢？最常见的就是Emoji 表情（Emoji 是一种特殊的 Unicode 编码，常见于 ios 和 android 手机上），和一些不常用的汉字，以及任何新增的 Unicode 字符等等。
那么utf8mb4比utf8多了什么的呢?
多了emoji编码支持.
如果实际用途上来看,可以给要用到emoji的库或者说表,设置utf8mb4.
比如评论要支持emoji可以用到。




# mysql分区



## 分区表

对于用户而言，分区表是一个独立的逻辑表，但是底层是由多个物理子表组成。分区表对于用户而言是一个完全封装底层实现的黑盒子，**对用户而言是透明的**，从文件系统中我们可以看到多个使用#分隔命名的表文件，这就是分区表文件。分区表在执行查询的时候，优化器会根据分区定义过滤掉那些我们不需要数据的分区，这样查询就无须扫描所有分区。 分区的主要目的是将数据安好一个较粗的力度分在不同的表中，这样可以将相关的数据存放在一起。



### 分区表的底层原理

​	分区表的操作按照以下的操作逻辑进行：

​		**select查询**

​		当查询一个分区表的时候，分区层先打开并锁住所有的底层表，优化器先判断是否可以过滤部分分区，然后再调用对应的存储引擎接口访问各个分区的数据

​		**insert操作**

​		当写入一条记录的时候，分区层先打开并锁住所有的底层表，然后确定哪个分区接受这条记录，再将记录写入对应底层表

​		**delete操作**

​		当删除一条记录时，分区层先打开并锁住所有的底层表，然后确定数据对应的分区，最后对相应底层表进行删除操作

​		**update操作**

​		当更新一条记录时，分区层先打开并锁住所有的底层表，mysql先确定需要更新的记录再哪个分区，然后取出数据并更新，再判断更新后的数据应该再哪个分区，最后对底层表进行写入操作，并对源数据所在的底层表进行删除操作

​		有些操作时支持过滤的，例如，当删除一条记录时，MySQL需要先找到这条记录，如果where条件恰好和分区表达式匹配，就可以将所有不包含这条记录的分区都过滤掉，这对update同样有效。如果是insert操作，则本身就是只命中一个分区，其他分区都会被过滤掉。mysql先确定这条记录属于哪个分区，再将记录写入对应得曾分区表，无须对任何其他分区进行操作

​		虽然每个操作都会“先打开并锁住所有的底层表”，但这并不是说分区表在处理过程中是锁住全表的，如果存储引擎能够自己实现行级锁，例如innodb，则会在分区层释放对应表锁。





 **分区表的应用场景**

- 表非常大以至于无法全部都放在内存中，或者只有表的最后部分有热点数据，其他均是历史数据（区分热点和非热点数据）
- 批量删除大量数据可以使用清除整个分区的方式
- 可以备份和恢复独立的分区
- 对一个独立分区进行优化、检查、修复等操作
- 分区表的数据可以分布在不同的物理设备上，从而高效地利用多个硬件设备
- 可以使用分区表来避免某些特殊的瓶颈
-  innodb的单个索引的互斥访问
-  ext3文件系统的inode锁竞争

  

 **分区表的限制**

- 一个表最多只能有1024个分区，在5.7版本的时候可以支持8192个分区
- 在早期的mysql中，分区表达式必须是整数或者是返回整数的表达式，在mysql5.5中，某些场景可以直接使用列来进行分区
-  如果分区字段中有主键或者唯一索引的列，那么所有主键列和唯一索引列都必须包含进来
- 分区表无法使用外键约束



**分区表的原理**



**分区表的类型**

1.  范围分区

​    根据列值在给定范围内将行分配给分区，很好理解

2. 列表分区

​    类似于按范围分区，区别在于 ：list分区是**基于列值匹配一个离散值集合中的某些指定值**来进行选择

    ```sql
    CREATE TABLE employees
    (
        id        INT  NOT NULL,
        fname     VARCHAR(30),
        lname     VARCHAR(30),
        hired     DATE NOT NULL DEFAULT '1970-01-01',
        separated DATE NOT NULL DEFAULT '9999-12-31',
        job_code  INT,
        store_id  INT
    ) PARTITION BY LIST(store_id) (
        PARTITION pNorth VALUES IN (3,5,6,9,17),
        PARTITION pEast VALUES IN (1,2,10,11,19,20),
        PARTITION pWest VALUES IN (4,12,13,14,18),
        PARTITION pCentral VALUES IN (7,8,15,16) );
    ```

3. 列分区

​    mysql从5.5开始支持column分区，可以认为是范围分区和列表分区的升级版，在5.5之后，可以使用column分区替代range和list，但是column分区只接受普通列不接受表达式

 ```sql
 CREATE TABLE `list_c`
 (
     `c1` int(11) DEFAULT NULL,
     `c2` int(11) DEFAULT NULL
 ) ENGINE=InnoDB DEFAULT CHARSET=latin1
 /*!50500 PARTITION BY RANGE COLUMNS(c1) (PARTITION p0 VALUES LESS THAN (5) ENGINE = InnoDB,  PARTITION p1 VALUES LESS THAN (10) ENGINE = InnoDB) */
 CREATE TABLE `list_c`
 (
     `c1` int(11) DEFAULT NULL,
     `c2` int(11) DEFAULT NULL,
     `c3` char(20) DEFAULT NULL
 ) ENGINE=InnoDB DEFAULT CHARSET=latin1 /*!50500 PARTITION BY RANGE COLUMNS(c1,c3) (PARTITION p0 VALUES LESS THAN (5,'aaa') ENGINE = InnoDB,  PARTITION p1 VALUES LESS THAN (10,'bbb') ENGINE = InnoDB) */
 CREATE TABLE `list_c`
 (
     `c1` int(11) DEFAULT NULL,
     `c2` int(11) DEFAULT NULL,
     `c3` char(20) DEFAULT NULL
 ) ENGINE=InnoDB DEFAULT CHARSET=latin1 /*!50500 PARTITION BY LIST COLUMNS(c3) (PARTITION p0 VALUES IN ('aaa') ENGINE = InnoDB,  PARTITION p1 VALUES IN ('bbb') ENGINE = InnoDB)
 ```

4.  hash分区

​    基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含myql中有效的、产生非负整数值的任何表达式

  ```sql
  CREATE TABLE employees
  (
      id        INT  NOT NULL,
      fname     VARCHAR(30),
      lname     VARCHAR(30),
      hired     DATE NOT NULL DEFAULT '1970-01-01',
      separated DATE NOT NULL DEFAULT '9999-12-31',
      job_code  INT,
      store_id  INT
  ) PARTITION BY HASH(store_id) PARTITIONS 4;  
  --  通过StoreId进行分区，将StoreId对4取模，这样就能放到模为0，1，2，3 这四个不同的分区中
  
  CREATE TABLE employees
  (
      id        INT  NOT NULL,
      fname     VARCHAR(30),
      lname     VARCHAR(30),
      hired     DATE NOT NULL DEFAULT '1970-01-01',
      separated DATE NOT NULL DEFAULT '9999-12-31',
      job_code  INT,
      store_id  INT
  ) PARTITION BY LINEAR HASH(YEAR(hired)) PARTITIONS 4;
  ```



5. key分区

​    类似于hash分区，区别在于key分区只支持一列或多列，且mysql服务器提供其自身的哈希函数，必须有一列或多列包含整数值

```sql
CREATE TABLE tk (
      col1 INT NOT NULL,
      col2 CHAR(5),
      col3 DATE ) PARTITION BY LINEAR KEY (col1) PARTITIONS 3;
```



6. 子分区

   在基于某个分区的基础之上，再进行分区

```sql
CREATE TABLE `t_partition_by_subpart`
(
    `id`     INT AUTO_INCREMENT,
    `sName`  VARCHAR(10) NOT NULL,
    `sAge`   INT(2) UNSIGNED ZEROFILL NOT NULL,
    `sAddr`  VARCHAR(20) DEFAULT NULL,
    `sGrade` INT(2) NOT NULL,
    `sStuId` INT(8) DEFAULT NULL,
    `sSex`   INT(1) UNSIGNED DEFAULT NULL,
    PRIMARY KEY (`id`, `sGrade`)
) ENGINE = INNODB PARTITION BY RANGE(id) SUBPARTITION BY HASH(sGrade) SUBPARTITIONS 2 ( PARTITION p0 VALUES LESS THAN(5), PARTITION p1 VALUES LESS THAN(10), PARTITION p2 VALUES LESS THAN(15) );
```



**如何使用分区表，应用场景**

 如果需要从非常大的表中查询出某一段时间的记录，而这张表中包含很多年的历史数据，数据是按照时间排序的，此时应该如何查询数据呢？ 因为数据量巨大，肯定不能在每次查询的时候都扫描全表。考虑到索引在空间和维护上的消耗，也不希望使用索引，即使使用索引，会发现会产生大量的碎片，还会产生大量的随机IO，但是当数据量超大的时候，索引也就无法起作用了，此时可以考虑使用分区来进行解决

  全量扫描数据，不要任何索引

  使用简单的分区方式存放表，不要任何索引，根据分区规则大致定位需要的数据为止，通过使用where条件将需要的数据限制在少数分区中，这种策略适用于以正常的方式访问大量数据

  索引数据，并分离热点

  如果数据有明显的热点，而且除了这部分数据，其他数据很少被访问到，那么可以将这部分热点数据单独放在一个分区中，让这个分区的数据能够有机会都缓存在内存中，这样查询就可以只访问一个很小的分区表，能够使用索引，也能够有效的使用缓存



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210905221724584.png" alt="image-20210905221724584" style="zoom:50%;" />

# mysql服务器参数设置

**通用参数**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723143550602.png" alt="image-20210723143550602" style="zoom:50%;" />

**字符参数**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723143613210.png" alt="image-20210723143613210" style="zoom:50%;" />

**连接参数**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723143727223.png" alt="image-20210723143727223" style="zoom:50%;" />

**日志参数**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723143830389.png" alt="image-20210723143830389" style="zoom:50%;" />

**缓存参数**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723143959826.png" alt="image-20210723143959826" style="zoom:40%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210724100850407.png" alt="image-20210724100850407" style="zoom:33%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210724101616557.png" alt="image-20210724101616557" style="zoom:50%;" />

**Innodb参数**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210723144135377.png" alt="image-20210723144135377" style="zoom:50%;" />



## innodb_flush_log_at_trx_commit

- 如果innodb_flush_log_at_trx_commit设置为0，log buffer将每秒一次地写入log file中，并且log file的flush(刷到磁盘)操作同时进行.该模式下，在事务提交的时候，不会主动触发写入磁盘的操作。

- 如果innodb_flush_log_at_trx_commit设置为1，每次事务提交时MySQL都会把log buffer的数据写入log file，并且flush(刷到磁盘)中去.

- 如果innodb_flush_log_at_trx_commit设置为2，每次事务提交时MySQL都会把log buffer的数据写入log file.但是flush(刷到磁盘)操作并不会同时进行。该模式下,MySQL会每秒执行一次 flush(刷到磁盘)操作。

  

注意： 由于进程调度策略问题,这个“每秒执行一次 flush(刷到磁盘)操作”并不是保证100%的“每秒”。

 

**sync_binlog**

sync_binlog 的默认值是0，像操作系统刷其他文件的机制一样，MySQL不会同步到磁盘中去而是依赖操作系统来刷新binary log。

当sync_binlog =N (N>0) ，MySQL 在每写 N次 二进制日志binary log时，会使用fdatasync()函数将它的写二进制日志binary log同步到磁盘中去。

注:

  如果启用了autocommit，那么每一个语句statement就会有一次写操作；否则每个事务对应一个写操作。

  根据上述描述，我做了一张图，可以方便大家查看。
 ![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/bb.jpeg)



**二 性能**

  两个参数在不同值时对db的纯写入的影响表现如下：

  ![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/bb-20210724103121374.jpeg)

 测试场景1 

 innodb_flush_log_at_trx_commit=2 

 sync_binlog=1000

 测试场景2 

 innodb_flush_log_at_trx_commit=1 

 sync_binlog=1000

 测试场景3 

 innodb_flush_log_at_trx_commit=1 

 sync_binlog=1

 测试场景4

 innodb_flush_log_at_trx_commit=1

 sync_binlog=1000

 测试场景5 

 innodb_flush_log_at_trx_commit=2 

 sync_binlog=1000 

 

| 场景  | TPS   |
| ----- | ----- |
| 场景1 | 41000 |
| 场景2 | 33000 |
| 场景3 | 26000 |
| 场景4 | 33000 |

**由此可见，当两个参数设置为双1的时候，写入性能****最差，sync_binlog=N (N>1 )** **innodb_flush_log_at_trx_commit=2 时，(在当前模式下)MySQL的写操作才能达到最高性能。**



**三 安全**

当innodb_flush_log_at_trx_commit和sync_binlog 都为 1 时是最安全的，在mysqld 服务崩溃或者服务器主机crash的情况下，binary log 只有可能丢失最多一个语句或者一个事务。但是鱼与熊掌不可兼得，双11 会导致频繁的io操作，因此该模式也是最慢的一种方式。

当innodb_flush_log_at_trx_commit设置为0，mysqld进程的崩溃会导致上一秒钟所有事务数据的丢失。
当innodb_flush_log_at_trx_commit设置为2，只有在操作系统崩溃或者系统掉电的情况下，上一秒钟所有事务数据才可能丢失。

**双1适合数据安全性要求非常高，而且磁盘IO写能力足够支持业务，比如订单,交易,充值,支付消费系统。双1模式下，当磁盘IO无法满足业务需求时 比如11.11 活动的压力。推荐的做法是 innodb_flush_log_at_trx_commit=2 ，sync_binlog=N (N为500 或1000) 且使用带蓄电池后备电源的缓存cache，防止系统断电异常。**















# 复习大纲

![178999e4dcb8d4f7a29e36d7dfb94909-1144961-20210719222822324](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/178999e4dcb8d4f7a29e36d7dfb94909-1144961-20210719222822324.png)x
