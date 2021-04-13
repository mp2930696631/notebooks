# 基本操作

```sql
-- 查看当前数据库连接信息
show processlist;

-- 查看上一条查询的代价
show status like 'last_query_cost';
（可以先使用explain，然后再使用这个命令）
```

# 性能监控

## 开启性能监控

```sql
set profiling=1;
```

## 使用

```sql
1、查看命令具体执行时间
show profiles;

2、查看每个步骤具体执行时间
show profile for query n;
其中，n表示show profiles列表中的query_id编号

3、查询所有性能相关信息，例如磁盘，io操作次数等
show profile all for query n
其中，n表示show profiles列表中的query_id编号
```

# 执行计划

```sql
作用：
查看sql语句具体的执行过程，以便知道sql是怎么执行的，进而做出相应的优化

语法：
explain sql语句

示例：
explain select * from emp;
```

## 输出格式

|      列       |                      解释                      |
| :-----------: | :--------------------------------------------: |
|      id       |            The `SELECT` identifier             |
|  select_type  |               The `SELECT` type                |
|     table     |          The table for the output row          |
|   partition   |            The matching partitions             |
|     type      |                 The join type                  |
| possible_keys |         The possible indexes to choose         |
|      key      |           The index actually chosen            |
|    key_len    |          The length of the chosen key          |
|      ref      |       The columns compared to the index        |
|     rows      |        Estimate of rows to be examined         |
|   filtered    | Percentage of rows filtered by table condition |
|     extra     |             Additional information             |

### id

```
代表select查询的序列号，包含一组数字，表示查询中执行select子句或者操作表的顺序,数字越大越先被执行，如果id相同的话，从上往下执行
```

### select_type

```sql
主要用来分辨查询的类型，是普通查询还是联合查询还是子查询

-- SIMPLE: 简单的查询，不包含子查询和union

-- PRIMARY: 查询中若包含任何复杂的子查询，最外层查询则被标记为Primary 

-- UNION: 若第二个select出现在union之后，则被标记为union
explain select * from emp where deptno = 10 union select * from emp where sal >2000;
（所以 select * from emp where sal >2000这个查询被标记为UNION 类型）

-- UNION RESULT: 从union表（临时表，可以在extra中看到using temporary）获取结果的select
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

-- DERIVED: from或者join子句中出现的子查询，也叫做派生类
explain select * from emp join (select empno from emp order by empno limit 11, 3) t using(empno)

-- SUBQUERY: 在select或者where列表中包含子查询
explain select * from emp where sal > (select avg(sal) from emp) ;
```

### table

```sql
对应正在访问哪一个表，表名或者别名，可能是临时表或者union合并结果集

1、如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名

2、表名是derivedN的形式，表示使用了id为N的查询产生的衍生表

3、当有union result的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id
```

1、如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名

explain select * from emp e

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130132754.png)

2、表名是derivedN的形式，表示使用了id为N的查询产生的衍生表

explain select * from emp join (select empno from emp order by empno limit 11, 3) t using(empno)

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130132923.png)

<derived2>中的数字2表示的是查询id为2的这个查询所产生的衍生表

3、当有union result的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id

explain select * from emp where deptno = 10 union select * from emp where sal >2000;

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130133110.png)

<union1,2>中的数字1,2表示将查询id为1,2的结果集合并

### type

```
type显示的是访问类型，访问类型表示我是以何种方式去访问我们的数据

性能排序：system>const>eq_ref>ref>range>index>all 
all表示全表扫描
```

#### all

```sql
含义：
全表扫描，也就是从聚集索引第一个叶子节点开始遍历，性能最低

情况：
无法使用索引时，或者优化器认为全表扫描更快时，会使用全表扫描

示例：
explain select * from emp;
```

#### index

```sql
含义：
全索引扫描，也就是从索引树的第一个叶子节点开始遍历，全表扫描是一种特殊的全索引扫描

情况：
select中没有带where，但是select的字段出现在某个单一的索引中（也就是出现了索引覆盖），如果是这种情况，mysql会选择扫描这颗普通索引树，而不是全表扫描，因为普通索引树存储的数据量相对于聚集索引树来说要小

示例：
explain select empno from emp;
使用的是外键索引，而不是主键索引，因为外键索引覆盖了empno这个主键
```

#### range

```sql
含义：
范围查找

情况：
当在索引列上进行范围查找时，会出现range，此时，explain的ref列为null

示例：
explain select * from emp where empno>7000;
```

#### ref

```sql
含义：
wu

情况：
命中普通索引，并且使用等值判断的时候，（对应可能有多条记录满足条件）

示例：
explain select * from emp where deptno=20;
deptno是外键索引
```

#### eq_ref

```sql
含义：
wu

情况：
当进行关联查询时，并且使用了主键索引或者非空唯一索引，并且是等值判断的情况下，会出现eq_ref，所以，它是性能最高的join查询了

示例：
explain select * from emp e, emp m where e.empno=m.empno;
从表命中主键索引，所以，从表相关查询会出现eq_ref
```

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210128173704.png)

#### const

```
含义：
wu

情况：
等值查询，命中主键索引或者唯一索引，（对应最多只会有一条记录满足条件）

它与eq_ref的区别就是，eq_ref对应的是关联查询

示例：
explain select * from emp where empno=7902;
```

#### system

### possible_keys

```
表示可能会使用到的索引
```

### key

```
表示实际使用到的索引
```

### key_len

```
表示实际使用到的索引的长度（如果是组合索引，则实际用到了几个列，就显示这几个列长度的总和）
```

### ref

```
代表与索引相比较的列（等值查询），如果该列是常数，则显示为const
```

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210128180043.png)

### rows

```
估计本次查询涉及到行数

优化器可以根据这个值，来决定是否使用索引（如果命中了普通索引，但是需要回表，且这个值得占比很大，可能就不会使用这个索引，而会进行全表扫描）
```

### extra

```
附加的信息,主要了解以下几个
using index
using where
using index condition
using join buffer
using filesort
```

#### using index

```sql
作用：
使用覆盖索引

示例：
explain select deptno from emp where deptno=20;
```

#### using where

```sql
作用：
条件过滤，条件过滤是放在服务器端，而不是存储引擎，服务器会根据存储引擎检索的行进行条件过滤，是这样的一个过程：存储引擎检索到一行（不管是全表扫描还是命中索引），返回到server，server进行条件过滤，决定该条记录是去还是留，然后server再要求存储引擎读取下一行

情况：
当有where子句，就可能会出现这个字段（如果使用了ICP，可能就不会出现了）

示例：
explain select * from emp where empno>7000;
```

#### using index condition

```sql
作用：
表示使用了索引条件下推（ICP）

情况：
在innodb中，只适用于二级索引。在这种情况下，如果命中了索引，并且需要回表，就会出现这个字段

示例：
explain select * from emp where deptno>30;
```

==扩展==

官方文档：https://dev.mysql.com/doc/refman/5.7/en/index-condition-pushdown-optimization.html

当不使用索引条件下推时，mysql是如何进行扫描的：

1、获取下一行，首先读取索引元组，然后利用索引元组去查找并读取完整的行记录（回表）

2、返回server进行where条件过滤，决定接受还是拒接该记录

当使用索引条件下推时，mysql是如何进行扫描的：

1、获取下一行，首先读取索引元组，然后存储引擎使用索引列进行条件过滤，如果不满足条件，则获取下一个索引元组

2、如果满足条件，再利用索引元组去查找并读取完整的行记录

3、返回server，完成==余下==的where条件过滤

#### using join buffer

```sql
作用：
使用join buffer进行关联查询

情况：
当从表无法使用索引的时候，主表会使用一个join buffer批量去从表进行比对，加快速度，一个join对于一个join buffer

示例：
explain select * from emp join dept;
```

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210128184124.png)

==扩展==

mysql中有三种join算法，但是总的来说，join其实就是一个嵌套循环的过程

1、Simple Nested-Loop Join

也就是最简单的嵌套循环，不做其他任何约束,一般不会使用这种算法

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210128184543.png)

2、Index Nested-Loop Join

也就是匹配表可以使用索引，拿着驱动表的每条记录，使用索引在匹配表上进行查找

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210128184557.png)

3、Block Nested-Loop Join

当匹配表不能使用索引时，mysql会进行一个优化，使用buffer存放部分驱动表的记录，然后批量到匹配表进行比对，加快速度，而不会使用第一种算法

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210128184612.png)

#### using filesort

```sql
作用：
表示没有进行索引排序

情况：
使用了order by，但是没使用索引

当需要进行排序时（也就是没有使用索引），如果需要排序的数据量大于sort_buffer_size，mysql会使用内存和磁盘交换进行排序，反之，就可以直接在内存中进行排序

示例：
explain select * from emp order by ename;
```

==扩展==

mysql中排序有两种，一种是==使用索引==，本身就是有序集，并不需要排序，一种是进行==文件排序==，也就是没有使用索引（由优化器决定是否使用索引，有时候还不如全表扫描来的效率高）

文件排序又分为两种情况

1、单次传输排序

也就是将涉及到的列全部取出，进行排序，然后返回给客户端

2、两次传输排序

也就是先只将需要排序的列取出，进行排序，然后在拿排好序的列去查找需要的记录

当需要排序的列的总大小超过max_length_for_sort_data定义的字节，mysql会选择双次排序，反之使用单次排序，当然，用户可以设置此参数的值来选择排序的方式

==注意==

上面两种排序方式和mysql使用sort_buffer_size进行排序没有冲突：

单次传输排序/两次传输排序是一种排序算法，而mysql使用sort_buffer_size进行排序是具体实现，当mysql选择了一种排序算法后，就可以使用sort_buffer_size进行排序了

#### using temporary

```sql
作用：
表示使用临时表

示例：
explain select * from emp where deptno = 10 union select * from emp where sal >2000;
```

#### Using index for group-by

```sql
作用：
表示使用了索引进行了分组，而不是临时表（这只是使用索引进行分组的一种情况，其实在进行分组的时候，只要extra中没有出现using temporary，就表示使用了索引）

示例：
explain select empno from emp group by deptno;
```

# 查询优化

```
这里涉及count()、关联查询的优化、limit的优化、order by的优化、group by的优化
```

==注意==

因为where的优先级很高，所以，下面的优化都会受到where子句的影响

## count()

```sql
count()这个聚集函数用来统计行数，有如下几种方式
1、count(*)
2、count(num)
3、count(column)
这三种方式的效率为1=2>=3，为什么是>=3呢，因为在执行count的时候，总是要进行全表扫描的（1、全纪录表扫描，或者2、全索引扫描）
当使用第三种方式进行计算时，column可能并没有命中索引，那么就会进行全记录表扫描，而第1或者第2中方式进行计算时，mysql会优先选择普通索引进行全索引扫描！！

示例：
explain select count(*) from emp;

explain select count(ename) from emp;
```

count(*)/count(1)

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130172653.png)

可以发现，就算没有加where子句，mysql还是会选择普通索引进行扫描，而不是扫描聚集索引

count(column)

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130172829.png)

进行了全记录表扫描，效率相对低一点

## 关联查询的优化

```sql
mysql的关联查询的算法就那三种，可以发现，三种算法中的驱动表都是进行全表扫描的，如果想让驱动表、匹配表都使用索引来缩小查询范围的话，可以加上where子句，但前提是：条件表达式只涉及一张表的索引字段，所以，关联查询的优化涉及下面两个方面：
1、只有当where、group by 、order by只涉及到一个表中的列时，mysql才可以使用索引来优化（这句话中的优化指的是，在那三种算法基础上进行的优化！！）
explain select * from emp e join dept d using(deptno) where d.deptno>=20;

2、否则，就是老老实实使用那三种算法中的一种——也就是驱动表全表扫描，这种情况下，驱动表建立索引只会增加维护成本，还不如不建立
```

==注意==

考虑以下三句sql的不同之处

1、explain select * from emp e join dept d using(deptno);

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130174018.png)

由此可知，==d表是驱动表，e表是匹配表==，在此基础上对比以下两句sql的不同之处

2、explain select * from emp e join dept d using(deptno) where d.deptno>=20;

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130174215.png)

由此可知，驱动表也使用了索引

3、explain select * from emp e join dept d using(deptno) where e.empno>=7500;

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130174352.png)

由此可知，匹配表在第一个sql的基础上还是用了索引下推

总结：只有当where、group by 、order by只涉及到一个表中的列时，mysql才可以使用索引来优化（这句话中的优化指的是，在那三种算法基础上进行的优化！！）

## limit优化

```sql
limit用来限制结果集的输出，所以，能使用limit就使用limit，因为limit是需要进行结果集扫描的，如果没有where条件的话，要么是全记录表扫描，要么是全索引扫描，所以，优化的关键是让其进行全索引扫描，而不是全记录表扫描，有以下两种思路：
```

考虑这个题目：我需要emp表的第12,13,14条记录，必须使用limit

如果不做优化，就是会写成下面的sql语句

explain select * from emp limit 11, 3;

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130175847.png)

可以发现，进行了全记录表扫描

1、使用覆盖索引

explain select * from emp join (select empno from emp ==order by empno== limit 11, 3) t using(empno)

（注意这里高亮的order by empno，因为如果不使用它的话，mysql会进行优化，因为子查询只需要empno的值，而外键索引可以覆盖，索引会选择走外键索引，而不是主键索引，这样会造成这句sql最后的执行结果和explain select * from emp limit 11, 3;这句sql的结果不同）

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\微信截图_20210130180329.png)

注意以上query id为1的查询，它使用的是query id为2的查询的派生表，所以实际上只会有少量的记录（虽然这里的rows显示的为14）

所以，当==普通索引占用空间特别小==的时候，这种方式就可以突显出性能高！

2、去掉limit的offset，也就是使用>=条件运算符（本质上还是利用了覆盖索引）

explain select * from emp where empno>=(select empno from emp ==order by empno== limit 11, 1) limit 3

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210227120618.png)

所以，当==普通索引占用空间特别小==的时候，这种方式就可以突显出性能高！

## order by优化

```
order by的优化就是让mysql使用索引排序而不是文件排序

我们可以通过explain的extra栏来区分，如果有using filesort，表示使用了文件排序，没有的话，就表示使用了索引

order by的优化涉及以下两方面
```

==前提==：where是否使用索引或者使用何种索引，都会影响到order by是否使用索引，原因是因为where的优先级高一些。只有在where和order by使用到的索引列都位于同一个==索引元组==中时，order by才有可能会使用索引

1、当使用组合索引时且没有where时，或者where中是范围查询时，order by中的索引列必须连续

如果where是范围查询，完全可以进行全索引扫描，而不用进行全表扫描，这也是利用索引的一种情况

2、当order by中的索引列不连续，但是where子句中的==索引列为常量==，且与order by的索引列组合起来可以构成组合索引中的连续索引列，这时，order by也可使用索引

==连续指的是：从索引的最左变开始，索引项可以不全，但是中间不能断开==

## group by优化

```
group by优化就是让mysql使用索引，而不是使用临时表来完成group by的操作

group by的原理：是使用一张临时表，在这张临时表中，每个组的行都是连续的（换句话说就是各组的行都聚在一起了），然后再发现组并使用聚集函数等操作

我们可以通过explain的extra栏来判断是否使用了索引，如果extra出现了using temporary，表示没有使用索引，反之，表示使用了索引

group by优化的情况和order by类似
```

==前提==：where是否使用索引或者使用何种索引，都会影响到group by是否使用索引，原因是因为where的优先级高一些。只有在where和group by使用到的索引列都位于同一个==索引元组==中时，group by才有可能会使用索引

1、当使用组合索引时且没有where时，或者where中是范围查询时，group by中的索引列必须连续

2、当group by中的索引列不连续，但是where子句中的==索引列为常量==，且与group by的索引列组合起来可以构成组合索引中的连续索引列，这时，group by也可使用索引

==连续指的是：从索引的最左变开始，索引项可以不全，但是中间不能断开==

## order by和group by

```
关键点是需要将where和order by/group by区分开来
也就是：
where是where，他是先于order by/group by执行的，所以，先要考虑where能用什么索引，再在这个基础上考虑其他的
举个例子：
就拿上面order by（gruop by是一样的理解）的第一点来说，
为什么当使用组合索引时且没有where时，或者where中是范围查询时，order by中的索引列必须连续
分为两点
1、如果索引项不连续，那么不管where怎么样，查出来的数据都是无序的，所以，必须使用文件排序！！
2、如果where是范围查询的话，诀窍是先看where使用了什么索引，这个是遵循组合索引的最左匹配原则的！！！
也就是：如果where条件满足最左匹配，那么执行计划type栏可以显示range及以上级别
如果where条件不满足最左匹配，那么进行全索引表扫描就可以了，而不用进行全表扫描！！也就是type栏显示的是index级别
```





