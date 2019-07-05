### Mysql安装:

查询命令: rpm -qa|grep -i mysql

**索引定义：** 索引（Index）是帮助Mysql高效获取数据的数据结构。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种形式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引。

**特性：**索引本身也很大，不可能全部存储到内存中，因此索引往往以索引文件的形式存储在磁盘上。

**优势：**类似大学图书馆建立书目索引，提高数据检索的效率，降低数据库的IO成本；通过索引列对数据进行排序，降低了数据排序的成本，降低了CPU的消耗

**劣势：**

1，实际上索引也是一张表，该表保存了主键及索引字段，并指向实体表的记录，所以索引列也是要占用空间的

2，虽然索引大大提高了查询速度，同时却会降低更新表的速度，例如update、insert和delete。因为更新表时，Mysql不仅要保存数据，还要保存一下索引文件每次更新添加了索引类的字段，都会调整因为更新所带来的键值变化的索引信息。

3，索引只是提高效率的一个因素，如果Mysql中有大数据量的表，就需要花时间研究建立最优秀的索引

分类：

1，单值索引：即一个索引只包含单个列，一个表可以有多个单列索引

2，唯一索引：索引列的值必须唯一，但允许有空值

3，复合索引：一个索引包含多个列

基本语法：

1，创建：

create 【UNIQUE】INDEX indexName ON tableName(columnName（length）);

ALTER tableName ADD 【UNIQUE】 INDEX 【indexName】 ON (columnName（length）);

几种方式：

1.添加PRIMARY KEY（主键索引）

mysql>ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` ) 

2.添加UNIQUE(唯一索引) 

mysql>ALTER TABLE `table_name` ADD UNIQUE ( `column` ) 

3.添加INDEX(普通索引) 

mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column` ) 

4.添加FULLTEXT(全文索引) 

mysql>ALTER TABLE `table_name` ADD FULLTEXT ( `column`) 

5.添加多列索引 

mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )

2，删除：

DROP INDEX 【indexName】ON tableName；

3，查看：

SHOW INDEX FROM tableName；



### 哪些情况需要创建索引?

1, 主键自动建立唯一索引

2,频繁作为查询条件的字段应该创建索引

3, 查询中与其他表关联的字段,外键关系建立索引

4, 频繁更新的字段不适合建立索引

5, Where条件里用不到的字段不创建索引

6, 单键/组合索引的选择? 高并发下倾向于创建组合索引,例如京东的商品查找

7, 查询中排序的字段,排序字段通过索引去访问将大大提高排序速度

8, 查询中统计或者分组字段



### 哪些情况不需要建立索引?

1,表记录太少

2,经常增删改查的表

3,数据重复且分布平均的表字段。（假如一个表有一百万条记录，性别这个字段只有男和女两个值，且每个值的分布概率为50%，这种情况下针对该字段建立索引一般不会提高查询效率。索引的选择性是指索引列中不同值的数目与表中记录的比。如果一个表中有2000条记录，表索引列有1980个不同的值，那么这个索引的选择性就是1980/2000=0.99.一个索引的选择性越接近于1，这个索引的效率就越高。）



### Mysql常见瓶颈:

CPU: CPU在饱和的时候一般发生在数据装入内存或从磁盘读取数据的时候

IO: 磁盘I/O瓶颈发生在装入数据远大于内存容量的时候

服务器硬件的性能瓶颈: top, free, iostat和vmstat来查看系统的性能状态



### 性能优化:

Explain关键字: 使用EXPLAIN关键字可以模拟优化器执行SQL|查询语句,从而知道Mysql是如何处理SQL语句的。可以分析出查询语句或是表结构的性能瓶颈。查看SQL的执行计划。

EXPLAIN具体能做什么？

1，表的读取顺序

2， 数据读取操作的操作类型

3， 哪些索引可以使用

4，哪些索引被实际引用

5， 表之间的引用

6，每张表有多少行被优化器查询

如何使用？

EXPLAIN + SQL语句

| id   | select_type | table    | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
| ---- | ----------- | -------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | :------: | ----- |
| 1    | SIMPLE      | sys_dept | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 4    |  100.00  | NULL  |

id: select 查询的序列号, 包含一组数字, 表示查询中执行select子句或操作表的顺序, 包含三种情况,:

```

​   1>  id相同, 执行顺序由上至下

​	2>  id不同, 如果是子查询, id的序号会递增, id值越大优先级越高, 越先被执行

​	3> id相同不同, 同时存在。id如果相同, 可以认为是一组, 从上往下顺序执行; 在所有组中,id值越大,优先级越高, 越先执行。衍生 = Derived
```

select_type: 查询的类型， 主要用于区别普通查询、联合查询、子查询等的复杂查询

```
有哪些？SIMPLE、PRIMARY、SUBQUERY、DERIVED、UNION、UNION RESULT

​	SIMPLE：简单的select查询，不包含子查询或者union

​	PRIMARY：查询中若包含任何复杂的字部份，最外层会被标记为primary

​	SUBQUERY：在select或者where列表中包含了子查询

​	DERIVED：在from列表中包含的子查询被标记为derived（衍生），Mysql会递归执行这些子查询并把结果放到临时表

​	UNION： 若第二个select出现在union之后，则标记为union；若union包含在from子句的子查询中，外层select被标记为：derived

​	UNION RESULT:: 从union表获取结果的select
```

table： 表示数据是来自于哪张表的，若为衍生表（即衍生出来的临时表）则会带其id一起展示

type：表的连接类型 。显示查询使用了何种类型，从最好到最差依次为

```
system --> const --> eq_ref --> ref --> fulltext --> ref_or_null --> index_merge --> unique_subquery --> index_subquery --> range --> index --> All (全部的，不须记）

system --> const --> eq_ref --> ref --> range --> index --> All （常用的，切记）

一般来说，需要保证查询至少达到range级别，最好能到达ref。

​	system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，可以忽略不计

​	const： 表示通过查询一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快，如将主键置于where列表中，MySQL就能将该查询转换为一个常量。

​	eq_ref: 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描

​	ref： 非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以它应该属于查找和扫描的混合体。

​	range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。

​	index： Full index scan，index与All区别为index类型只遍历索引树。这通常比All快，因为索引文件通常比数据文件小。（简单说就是All和Index都是读全表，但Index是从索引中读取的，而All是从硬盘中读取的）

​	all： Full Table Scan，将遍历全表以找到匹配的行
```

 possible_keys： 表示查询中可能使用的索引；查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用。如果备选的数量大于3那说明已经太多了，因为太多会导致选择索引而损耗性能， 所以建表时字段最好精简，同时也要建立联合索引，避免无效的单列索引。

key：表示查询使用到的索引；如果为Null，则没有使用索引或索引失效；查询中若使用了覆盖索引，则该索引仅出现在key列表中。 

key_len：表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确度的情况下，长度越短越好；key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的

ref：表示使用哪个列或常数与索引一起来查询记录。

rows： 表示查询的行数;试图分析所有存在于累计结果集中的行数，虽然只是一个估值，却也足以反映 出SQL执行所需要扫描的行数，因此这个值越小越好；

filtered：指返回结果的行占需要读到的行(rows列的值)的百分比.

Extra：表示查询过程的附件信息。 

```
    Using filesort（急需优化）： 说明Mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。Mysql中无法利用索引完成的排序操作称为“文件排序“。

​	Using temporary（急需优化）： 用了临时表保存中间结果，Mysql在对查询结果排序时使用临时表。常见于排序order by 和分组查询group by。

​	Using index：表示相应的select操作中使用了索引覆盖（Coving index），避免了访问表的数据行，效率不错！如果同时出现using where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。

​ 	Using where： 表明使用了where 过滤

​	using join buffer：使用了连接缓存

​	impossible where： where子句的值总是false，不能用来获取任何数据，例如where 1=2

​	select tables optimized away：在没有Group by子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎优化COUNT（*）操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。

​	distinct： 优化distinct操作，在找到第一匹配的元组后即停止找相同值的动作
```

索引覆盖（Coving index): select的数据列只用从索引中就能够取得，不必读取数据行，Mysql可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，简而言之就是查询列要被所建的索引覆盖。

思考1：数据为何采用逻辑删除而非物理删除？

1，保留原始数据方便做数据分析

2，维护索引

思考2：为何建立了索引的字段不能频繁的增删改？

索引的建立过程中会以某种形式指向数据，并实现相应的查找算法提高查询效率，对数据频繁的的修改和删除最终会导致索引失效。

### order  by排序优化

1. MySQL支持两种方式的排序filesort和index，Using index是指MySQL扫描索引本身完成排序。index效率高，filesort效率低。 

2. order by满足两种情况会使用Using index。

     order by语句使用索引最左前列。

     使用where子句与order by子句条件列组合满足索引最左前列。

3. 尽量在索引列上完成排序，遵循索引建立（索引创建的顺序）时的最佳左前缀法则。

4. 如果order by的条件不在索引列上，就会产生Using filesort。

   filesort有两种排序算法：双路排序和单路排序。

   ```
   双路排序：在MySQL4.1之前使用双路排序，就是两次磁盘扫描，得到最终数据。读取行指针和order by列，对他们进行排序，然后扫描已经排好序的列表，按照列表中的值重新从列表中读取对应的数据输出。即从磁盘读取排序字段，在buffer进行排序，再从磁盘取其他字段。
   
   如果使用双路排序，取一批数据要对磁盘进行两次扫描，众所周知，I/O操作是很耗时的，因此在MySQL4.1以后，出现了改进的算法：单路排序。
   
   单路排序：从磁盘中查询所需的列，按照order by列在buffer中对它们进行排序，然后扫描排序后的列表进行输出。它的效率更高一些，避免了第二次读取数据，并且把随机I/O变成了顺序I/O，但是会使用更多的空间，因为它把每一行都保存在内存中了。
   ```

   单路排序出现的问题：

   当读取数据超过sort_buffer的容量时，就会导致多次读取数据，并创建临时表，最后多路合并，产生多次I/O，反而增加其I/O运算。

   解决方式：

   a.增加sort_buffer_size参数的设置。

   b.增大max_length_for_sort_data参数的设置。

5. 提升order by速度的方式：

   ```
   1.在使用order by时，不要用select *，只查询所需的字段。
   因为当查询字段过多时，会导致sort_buffer不够，从而使用多路排序或进行多次I/O操作。
   2.尝试提高sort_buffer_size。
   3.尝试提高max_length_for_sort_data。
   ```

6. group by与order by很类似，其实质是先排序后分组，遵照索引创建顺序的最佳左前缀法则。当无法使用索引列的时候，也要对sort_buffer_size和max_length_for_sort_data参数进行调整。注意where高于having，能写在where中的限定条件就不要去having限定了。

   ![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/ordeby索引情况.png?dkey=843d3fcd-cbd9-48b1-a1ee-0291503dd3a4>)

###  如何避免索引失效？

1. 遵守最佳左前缀法则。（如果索引了多列，要遵守最佳左前缀法则。指的是查询从索引的最左前列开始并且**不跳过索引中的列**）
2. 不在索引列上做任何操作。（计算、函数、自动或手动的类型转换,会导致索引失效进而转向全表扫描）
3. 尽量避免在存储引擎下使用索引中使用了范围条件的右边的列。（这种情况下在范围条件之后的索引列将失效）
4. Mysql在使用不等于（!=或者<>）的时候无法使用索引会导致全表扫描。
5. 在使用is null或者is not null时同样无法使用索引导致全表扫描。
6. like以通配符开头（'%abc'）Mysql索引失效会变成全表扫描的操作。（如何解决？使用索引覆盖）
7. 字符串不加单引号索引失效。（会使用隐式的自动转换将整型转换为字符型导致索引失效）
8. 尽量避免用or，用它来连接会导致索引失效。
9. 尽量使用索引覆盖，避免select * 情况。

### 慢查询日志

介绍：

```
MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过指定阀值的SQL语句，运行时间超过long_query_time值的SQL，会被记录到慢查询日志中。

默认情况下，MySQL数据库并不启动慢查询日志，需要手动开启。如果不是调优需要的话，一般不建议开启，因为开启慢查询日志会或多或少带来一定的性能影响。

在SQL Server中我们利用SQL Profile来记录SQL执行情况，在Oracle中我们可以使用AWR、ASH报告来分析历史SQL执行情况，类似的调优方式映射到MySQL中，即对应为慢查询日志。
```

开启：

```sql
开启慢查询日志只需要如下修改部分参数即可，这里设置的慢查询阈值为0.1s，项目上可以根据实际情况调整，这边的建议是如果压测可以设置为0.1s，如果是非压测可以设置为2s。由于开启了log_queries_not_using_indexes，所以慢日志不仅会记录超过阈值的SQL，没有走索引的SQL也会记录日志，即使这些SQL并没有超过阈值。通过set global设置参数后，只会在新连接中生效，所以要查看修改后的参数值，需要重新连接。

mysql> set global long_query_time = 0.1;   
Query OK, 0 rows affected (0.00 sec)
 
mysql> set global log_queries_not_using_indexes = 1;    
Query OK, 0 rows affected (0.00 sec)
 
mysql> set global log_throttle_queries_not_using_indexes = 0;
Query OK, 0 rows affected (0.00 sec)
 
mysql> set global min_examined_row_limit = 100;
Query OK, 0 rows affected (0.00 sec)
 
mysql> set global slow_query_log = 1;
Query OK, 0 rows affected (0.00 sec)
```



### show profile优化

show profile 是Mysql提供的可以用来分析当前会话中语句执行的资源消耗情况，可以用于SQL调优的测量。

开启：

```sql
Show profiles是5.0.37之后添加的，要想使用此功能，要确保版本在5.0.37之后。

#查看数据库版本
mysql> select version();

#查看开启状态
mysql> show variables like "profiling ";

#开启profile
mysql> set profiling=1;


```

参数展示：

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/showProfile参数展示.png?dkey=45e6f4cc-ab93-4f44-93cc-91cf7c519bd9>)

以下参数急需优化：

```
covert HEAP to MyISAM：查询结果太大，内存已经不够用开始搬到磁盘了。
Creating tmp table：创建临时表，之后还需要拷贝数据到临时表，用完再删除，增加了磁盘IO。
Copying to tmp table on disk：把内存中的临时表复制到磁盘，极端危险！！！
locked： 锁
```

### SQL优化流程

1. 慢查询的开启并捕获。
2. explain+慢SQL分析。
3. show profile查询SQL在Mysql服务器里面的执行细节和生命周期情况。
4. SQL数据库服务器的参数调优。

### SQL执行顺序

from --> on -->join --> where --> group by --> having --> select distinct --> order by --> limit

### 七种SQL JOIN方案（针对两张及两张以上关联表）

INNER JOIN（内连接）

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/sql_inner join.png?dkey=cd4ec8f5-e899-429b-ac90-931d85100510>)

```sql
SELECT <select_list> 
FROM Table_A A
INNER JOIN Table_B B
ON A.Key = B.Key
```

LEFT JOIN (左连接)

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/sql_left  join.png?dkey=c024ae95-62af-45df-95ff-39fb05d16f2f>)

```sql
SELECT <select_list>
FROM Table_A A
LEFT JOIN Table_B B
ON A.Key = B.Key
```

RIGHT JOIN (右连接)

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/sql_right join.png?dkey=760b5529-89f4-438a-9bea-53ede720ad1f>)

```sql
SELECT <select_list>
FROM Table_A A
RIGHT JOIN Table_B B
ON A.Key = B.Key
```

OUTER JOIN (外连接)

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/sql_outer join.png?dkey=90fb2cca-4410-4e54-94bc-1daa8aa3f004>)

```sql
SELECT <select_list>
FROM Table_A A
FULL OUTER JOIN Table_B B
ON A.Key = B.Key
```

LEFT Excluding JOIN（左连接排除内连接结果）

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/sql_Left Exclude join.png?dkey=b83e522c-f80e-44a1-aacb-85dc4ad92057>)

```sql
SELECT <select_list> 
FROM Table_A A
LEFT JOIN Table_B B
ON A.Key = B.Key
WHERE B.Key IS NULL
```

RIGHT Excluding JOIN（右连接排除内连接结果）

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/sql_Right Exclude join.png?dkey=91bab137-2f62-4f80-bba9-cc0582b973a3>)

```sql
SELECT <select_list>
FROM Table_A A
RIGHT JOIN Table_B B
ON A.Key = B.Key
WHERE A.Key IS NULL
```

OUTER Excluding JOIN（外连接排除内连接结果）

![](<http://39.96.187.148:8080/externalLinksController/downloadFileByKey/sql_Outer Exclude join.png?dkey=28e0690b-0065-48e1-b892-4793913a340f>)

```sql
SELECT <select_list>
FROM Table_A A
FULL OUTER JOIN Table_B B
ON A.Key = B.Key
WHERE A.Key IS NULL OR B.Key IS NULL
```

join语句的优化：

1> 尽可能地减少Join语句中的NestedLoop的循环总次数；“永远用小的结果集驱动大的结果集”。

2> 优先优化NestedLoop的内层循环。

3> 保证Join语句中被驱动表上Join条件字段已经被索引。

4> 当无法保证被驱动表的Join条件字段被索引且内存资源充足的前提下，不要太吝惜JoinBuffer的设置。