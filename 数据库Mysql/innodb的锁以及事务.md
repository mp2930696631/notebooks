# 基本操作

```sql
-- 查看隔离级别
show variables like "%isolation%";

-- 关闭自动提交
set @@autocommit=0;

-- 开启自动提交
set @@autocommit=1;

-- 开启事务
begin或者start transaction;

-- 提交事务
commit;

-- 回滚事务
rollback;

-- 设置系统的隔离级别
-- 1、读未提交
set session transaction isolation level read uncommitted;
-- 2、读已提交
set session transaction isolation level read committed;
-- 3、可重复读
set session transaction isolation level repeatable read;
-- 4、串行化
set session transaction isolation level serializable;

-- 获取myisam的表共享读锁
lock table t_name read;

-- 获取myisam的表独占写锁
lock table t_name write;

-- 释放myisam的锁
unlock tables;

-- 获取innodb的排他锁（针对的是索引记录行）
-- 对于update、insert、delete，innodb会自动为其添加排它锁
select ... for update;

-- 获取innodb的共享锁（针对的是索引记录行）
select .... lock in share mode;
```



# mysql中的锁（innodb）

```
在innodb中，不同的隔离级别使用的锁是不一样的，在这里只讨论read committed以及repeatable read这两种事务隔离级别的锁机制
```

==扩展==

当在RR/RC级别下，在事务中使用一致性读取的话，如果事务未提交，在另一个事务是无法删除表的

## 使用的锁种类

### 行锁

```
分为共享锁（S锁）和排它锁（X锁）

我们可以通过select .... lock in share mode;来获取行的共享锁
我们可以通过select ... for update;来获取行的排它锁

mysql中select for update、update、delete加了行排他锁，

虽然叫行锁，但其实是对索引加的锁，也可以叫做索引记录锁。所以就算是筛选普通列，锁定的是隐藏的row_id主键记录

read committed/repeatable read都会加上这种锁
```

### 间隙锁

```
顾名思义，锁定的是记录之间的间隙
```

### next-key 锁

```
它是行锁+间隙锁

repeatable read使用这种锁来规避幻读问题
```

### 表锁

```
1、表共享读锁（S锁）
lock table <tablename> read;

2、表排他写锁（X锁）
lock table <tablename> write;

这里不讨论表锁，因为mysql中很少使用到表锁，就算是对某个表执行alter table、drop table这些DDL语句的时候，其他事务对这个表执行DML操作会阻塞。反过来也是一样。就算这样，mysql也没有使用表锁，而是使用元数据锁（Metadata locks，简称MDL）来实现的
```

### 意向锁

```
分为意向共享锁（IS锁）和意向排他锁（IX锁），它是一种表级锁，它表示有事务正在打算锁定行记录

当我们获取一个行的共享锁（S锁）之前，首先需要在对应的表上获取IS锁
当我们获取一个行的排他锁（X锁）之前，首先需要在对应的表上获取IX锁

意向锁只会对表锁（例如lock table <tablename> write）起作用，而不会阻止获取行锁
```

## read uncommitted

```
官方说该隔离级别下的工作方式类似于read committed，这里不做讨论了
```

## read committed

```
对于read committed级别，我们需要从以下几个方面来探索
1、where子句中不包含索引项（以下简称no index）
2、where子句中包含索引项（index）
3、使用select for update获取排他锁（以下简称 sfu）
4、update或者delete操作（以下简称ud）

总的原则就是：
情况一、如果where中包含索引列，并且使用到了mysql索引了，那么被扫描到的相关的索引记录都会被上锁
情况二、如果where子句中没有索引列，且执行的是update或者是delete操作，那么只会对符合条件的行加锁（也就是符合where条件）
如果是这种情况，对于update，如果当前被访问的行已经加了锁了，那么，innodb会进行“半一致性读取semi-consistent”，也就是读取最新提交的事物记录，然后评估where条件，如果满足，则，再次读取该行记录，此时再决定是否加锁或者等待
情况三、如果是select for update，那么是要是被扫描到的记录都会被加上锁

（注意：就算是limit也是一样的，例如，当我们使用select ... limit offset, n for update的时候，如果显示的是全表扫描，那么所有记录都会被加上锁。由此可见，limit的筛选也是放到了server）
```

### 情况一

```sql
步骤一、测试数据：
create table testable (
id int(1),
name varchar(10),
age int,
key u_id (id)
);
insert into testable values (1,'123', 1);
insert into testable values (2,'0123',2);
insert into testable values (3,'1234',2);
insert into testable values (5,'124',1);
insert into testable values (6,'01123',3);
insert into testable values (20,'143',2);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read committed
set @@autocommit=0;
set session transaction isolation level read committed;

步骤三、
在A窗口执行 update testable set age=10 where id>3  and name='143';
然后在B窗口执行 update testable set age=10 where id>3  and name='124';
发现B窗口发生阻塞，但是很明显，我更新的记录并不是同一条
而如果在B窗口执行update testable set age=10 where id=1  and name='123';却不会发生阻塞
通过查看执行计划，发现执行上面语句的时候，mysql使用了索引

结论：如果where中包含索引列，并且使用到了mysql索引了，那么被扫描到的相关的索引记录都会被上锁
```

==注意==

我们也可以构造一个where中包括了索引列，但是查看执行计划却没有使用索引的情况，这种情况的结果和==情况二==是一样的

```sql
我们再增加一些记录（可能并不需要增加，具体要怎么做，通过查看执行计划可以得知）
insert into testable values (8,'143',2);
insert into testable values (9,'143',2);
insert into testable values (12,'143',2);
insert into testable values (13,'143',2);
insert into testable values (14,'143',2);

然后在A窗口使用update testable set age=10 where id>0  and name='143';
然后在B窗口中使用update testable set age=10 where id>0  and name='124';
发现B窗口可以执行成功

结论：where中包括了索引列，但是实际在执行的过程中没有使用索引的话，那么这种情况会和情况二一样，也就是先对记录加锁，然后会到server判断是否符合where条件，如果符合的话，就持有或者等待锁，如果不符合条件的话，就释放锁
```



### 情况二

```sql
步骤一、测试数据：
create table testable (
id int(1),
name varchar(10),
age int
);
insert into testable values (1,'123', 1);
insert into testable values (2,'0123',2);
insert into testable values (3,'1234',2);
insert into testable values (5,'124',1);
insert into testable values (6,'01123',3);
insert into testable values (20,'143',2);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read committed
set @@autocommit=0;
set session transaction isolation level read committed;

步骤三、
在A窗口执行 update testable set age=10 where id>3  and name='143';
然后在B窗口执行 update testable set age=10 where id>3  and name='124';
发现B窗口可以执行成功

结论：如果where子句中没有索引列，且执行的是update或者是delete操作，那么只会对符合条件的行加锁（也就是符合where条件）
过程是这样的，在全表扫描的时候，首先对记录加锁，然后返回server做where判断，如果符合条件，就持有锁直到事务结束，反之，释放锁
```

### 情况三

```sql
(我们也可以将情况一、情况二中的语句换成select for update看看是什么结果)

步骤一、测试数据：
create table testable (
id int(1),
name varchar(10),
age int,
key u_id (id)
);
insert into testable values (1,'123', 1);
insert into testable values (2,'0123',2);
insert into testable values (3,'1234',2);
insert into testable values (5,'124',1);
insert into testable values (6,'01123',3);
insert into testable values (20,'143',2);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read committed
set @@autocommit=0;
set session transaction isolation level read committed;

步骤三、
1、
在A窗口执行 select * from testable where id>3  and name='143' for update;
然后在B窗口执行 select * from  testable where id>3  and name='124' for update;
发现B窗口发生阻塞，但是很明显，我更新的记录并不是同一条
而如果在B窗口执行 select * from testable where id=1  and name='123' for update;却不会发生阻塞
通过查看执行计划，发现执行上面语句的时候，mysql使用了索引
（这部分情况跟情况一结果相同，因为使用了索引，mysql只会扫描相关的索引项， 也是只有被扫描的记录才会被加锁（不管符不符合条件））

2、
在A窗口执行 select * from testable where name='143' for update;
然后在B窗口执行 select * from  testable where name='124' for update;
发现B窗口阻塞
因为全表扫描，所有的记录都被扫描到了，所以所有的记录都被加上了锁

结论：如果是select for update，那么是要是被扫描到的记录都会被加上锁
```

## repeatable read

```
对于repeatable read隔离级别的事务，因为它还是用next-key 锁，所以多了一些情况

经过了read committed的三种情况的洗礼，我们应该对“被扫描到的记录”这几个字很敏感了吧，每次，对于repeatable read，总的原则就是只要是被扫描的记录，就会被加锁，而不会向read committed的情况二那样，还会释放锁，主要是为了避免出现幻读

总的原则：
只要是被扫描的记录，就会被加锁。具体有分为下面两种情况
1、如果使用了唯一索引，并且，where条件是=号，那么只会添加行锁，而不会使用间隙锁
2、其他情况则加上next-key 锁，也就是行锁+间隙锁

（注意：就算是limit也是一样的，例如，当我们使用select ... limit offset, n for update的时候，如果显示的是全表扫描，那么所有记录都会被加上锁。由此可见，limit的筛选也是放到了server）
```

==注意==

这里就不演示了，因为后面会演示repeatable read如何避免幻读的情况

## serializable

```
官方说该隔离级别下的工作方式类似于RR，但是，如果关闭自动提交，普通的select被默认转换为了select .. lock in share mode,也就是在该隔离级别下，使用的锁定读取，并不是一致性读取了
```

# mysql中的事务演示

```
mysql中的事务隔离级别有4中，等级从低到高依次是：read uncommitted、read committed、repeatable read、serializable
```

## read uncommitted

```sql
步骤一、测试数据
create table testable (
id int(1),
name varchar(10),
age int
);
insert into testable values (1,'123', 1);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read uncommitted
set @@autocommit=0;
set session transaction isolation level read uncommitted;

步骤三、如下表格
```

| A窗口                                                        | B窗口                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set @@autocommit=0;<br/>set session transaction isolation level read uncommitted; | set @@autocommit=0;<br/>set session transaction isolation level read uncommitted; |
| select * from testable;                                      |                                                              |
|                                                              | insert into testable values (21, '456', 5);                  |
| select * from testable;<br/>可以发现，查询到了在B窗口未提交的那条新插入的记录<br/>这就是——读未提交 |                                                              |

## read committed

```sql
步骤一、测试数据
create table testable (
id int(1),
name varchar(10),
age int
);
insert into testable values (1,'123', 1);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read committed
set @@autocommit=0;
set session transaction isolation level read committed;

步骤三、如下表格
```

| A窗口                                                        | B窗口                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set @@autocommit=0;<br/>set session transaction isolation level read committed; | set @@autocommit=0;<br/>set session transaction isolation level read committed; |
| select * from testable;                                      |                                                              |
|                                                              | insert into testable values (21, '456', 5);                  |
| select * from testable;<br>查询不到B窗口插入的那条记录       |                                                              |
|                                                              | commit;                                                      |
| select * from testable;<br>可以查询出B窗口插入的记录<br>这就是——读已提交，可以读取到其他事务以及提交的数据 |                                                              |

## repeatable read

```sql
步骤一、测试数据
create table testable (
id int(1),
name varchar(10),
age int
);
insert into testable values (1,'123', 1);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是repeatable read
set @@autocommit=0;
set session transaction isolation level repeatable read;

步骤三、如下表格
```

| A窗口                                                        | B窗口                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set @@autocommit=0;<br/>set session transaction isolation level repeatable read; | set @@autocommit=0;<br/>set session transaction isolation level repeatable read; |
| select * from testable;                                      |                                                              |
|                                                              | insert into testable values (21, '456', 5);                  |
| select * from testable;<br>看不到B窗口插入的记录             |                                                              |
|                                                              | commit;                                                      |
| select * from testable;<br>看不到B窗口插入的记录             |                                                              |
| commit;                                                      |                                                              |
| select * from testable;<br>可以查询到B窗口插入的记录         |                                                              |

## serializable

```sql
步骤一、测试数据
create table testable (
id int(1),
name varchar(10),
age int
);
insert into testable values (1,'123', 1);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是serializable
set @@autocommit=0;
set session transaction isolation level serializable;

步骤三、如下表格
```

| A窗口                                                        | B窗口                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set @@autocommit=0;<br/>set session transaction isolation level serializable; | set @@autocommit=0;<br/>set session transaction isolation level serializable; |
| select * from testable;                                      |                                                              |
|                                                              | select * from testable;<br>不阻塞                            |
|                                                              | insert into testable values (21, '456', 5);<br>阻塞，说明，所有事务只能串行执行 |

==注意==

为什么户出现上面的情况，是因为，在serializable模式下，如果关闭自动提交，普通的select被默认转换为了select .. lock in share mode,也就是在该隔离级别下，使用的锁定读取，并不是一致性读取了

# 数据不一致问题

```
mysql中可能出现的数据不一致问题主要有：脏读、不可重读读、幻读，在不同的隔离级别下会出现不同的问题
```

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :------: | :--: | :--------: | :--: |
| 读未提交 |  √   |     √      |  √   |
| 读已提交 |  ×   |     √      |  √   |
| 可重复读 |  ×   |     ×      |  ×   |
|  串行化  |  ×   |     ×      |  ×   |

## 脏读

```
脏读：本事务读取到其他事务未提交的数据，别的事务之后可能进行了回滚，那么这个数据就是脏数据了

见mysql中的事务演示——read uncommitted，这个标题中演示的结果就是一种脏读
```

## 不可重复读

```
不可重复读：也就是在一次事务中，两次读取的结果不一样

见mysql中的事务演示——read committed，这个标题中演示的结果就是一种不可重复读
```

## 幻读

```
幻读：针对的是select for update（或者说读取数据库最新数据（当前读），而不是快照读），当使用当前读的时候，两次读取的结果不一样
```

### read committed中的幻读问题

使用当前读来验证

```sql
步骤一、测试数据
create table testable (
id int(1),
name varchar(10),
age int
);
insert into testable values (1,'123', 1);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read committed
set @@autocommit=0;
set session transaction isolation level read committed;

步骤三、如下表格
```

| A窗口                                                        | B窗口                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set @@autocommit=0;<br/>set session transaction isolation level read committed; | set @@autocommit=0;<br/>set session transaction isolation level read committed; |
| select * from testable for update;                           |                                                              |
|                                                              | insert into testable values (21, '456', 5);                  |
| select * from testable for update;<br>阻塞，因为insert的记录也会被加上排它锁<br>按下ctrl+c终止，然后继续操作 |                                                              |
|                                                              | commit;                                                      |
| select * from testable for update;<br>可以发现，看到了B窗口插入的记录 |                                                              |

==注==

为什么这种情况叫幻读，如果你==不使用当前读==来读取记录的话，你就看不到最新插入的记录，这个时候如果该表有唯一约束，你又在你自己的事务里面插入了一条相同的记录，这个时候虽然会阻塞，但是当其他事务提交后，你的这个事务就会报错，也就是说，这条记录对于你的事务来说就像“幻影”样，使用快照读查不到，想插入又插入不进去——但是mysql官方使用的例子是如上面的形式那样，也就是使用==当前读==。其实使用==快照读==或者==当前读==来验证==幻读==，本质上都是一样的，只不过使用==快照读==来验证好理解一些。请看下面例子

使用快照读来验证

```sql
步骤一、测试数据
create table testable (
id int(1),
name varchar(10),
age int,
unique p_id (id)
);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read committed
set @@autocommit=0;
set session transaction isolation level read committed;

步骤三、如下表格
```

| A窗口                                                        | B窗口                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set @@autocommit=0;<br/>set session transaction isolation level read committed; | set @@autocommit=0;<br/>set session transaction isolation level read committed; |
| select * from testable where id=1;<br>发现没有找到记录       |                                                              |
|                                                              | insert into testable values (1, '456', 1);                   |
| select * from testable where id=1;<br/>发现没有找到记录      |                                                              |
| insert into testable values (1, '456', 1);<br>发现阻塞       |                                                              |
|                                                              | commit;                                                      |
| 报错：Duplicate entry '1' for key 'testable.p_id'            |                                                              |

### repeatable read解决幻读问题

```
repeatable read是使用next-key解决幻读问题的，分为两种情况
情况一、如果使用了唯一索引，并且，where条件是=号，那么只会添加行锁，而不会使用间隙锁
情况二、其他情况则加上next-key 锁，也就是行锁+间隙锁
```

#### 情况一

```sql
步骤一、测试数据
create table testable (
id int(1),
name varchar(10),
age int,
unique p_id (id)
);
insert into testable values (1,'123', 1);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read committed
set @@autocommit=0;
set session transaction isolation level repeatable read;

步骤三、如下表格
```

| A窗口                                                        | B窗口                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set @@autocommit=0;<br/>set session transaction isolation level repeatable read; | set @@autocommit=0;<br/>set session transaction isolation level repeatable read; |
| explain select * from testable where id=1 for update;<br>发现确实使用了id这个唯一索引 |                                                              |
| select * from testable where id=1 for update;                |                                                              |
|                                                              | insert into testable values (2, '123', 1);<br>插入成功       |
| select * from testable where id=1 for update;<br>看不到新插入的记录，即没有出现幻读 |                                                              |

#### 情况二

==注意==

在测试这种情况的时候，数据、步骤和<kbd>幻读——read committed中的幻读问题——使用当前读来验证</kbd>一致，只是隔离级别改了

```sql
步骤一、测试数据
create table testable (
id int(1),
name varchar(10),
age int
);
insert into testable values (1,'123', 1);

步骤二、开两个命令行窗口（A、B），关闭自动提交，并设置隔离级别是read committed
set @@autocommit=0;
set session transaction isolation level read committed;

步骤三、如下表格
```

| A窗口                                                        | B窗口                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| set @@autocommit=0;<br/>set session transaction isolation level read committed; | set @@autocommit=0;<br/>set session transaction isolation level read committed; |
| select * from testable for update;                           |                                                              |
|                                                              | insert into testable values (21, '456', 5);<br>阻塞，而在read committed级别下并没有阻塞，这样也就解决了幻读问题 |

#### 附加情况

```
如果使用了唯一索引，但是表中也没有记录，如果在一个select * from testable where id=1 for update;（也就是查询出的结果为空），那么，其他事务也是插入不来id=1的记录的
```

# 锁、事务、当前读、快照读之间的关系

```
通过上面的知识，涉及到锁、事务、当前读、快照读，在一些案例中使用当前读、一些案例使用锁、一些案例又使用快照读等等，感觉有点晕。这里来说一些他们之间的联系以及共同点，其实他们并不冲突，只是对应于不同的情况而已
```

==注意==

这里只讨论RR和RC级别，因为对于读未提交级别，直接读取最新的记录就可以了，并不需要进行特殊处理

## 前置知识

### 一致性读取（非锁定读取）

```
也就是基于快照的读取，它是通过查询数据库某个时间点的快照，使得可以看到该时间点之前提交的事务做出的修改，而看不到该快照之后未提交的事务（RC）或已提交事务的修改（RR）修改。 如果查询的数据已经被别的事务修改，则通过undolog中的内容来重建内容

RR/RC默认就是使用一致性读取，也就是普普通通的select语句

一致性读取不使用任何锁
```

==（这个也就是我们常说的快照读）==

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210127075138.png)

### 锁定读取

```
是与一致性读取相反的情况，使用它可以看到数据库最新的情况（前提是记录未被锁定，也就是其他事务已经提交了），它会使用锁机制（不同的隔离级别使用的所不同，可以见前面讨论的锁机制）

在mysql中，有两种锁定读取
1、select ... lock in share mode(8.0后可以使用select .... for share)
使用的锁是行共享锁

2、select .... for update
使用的锁是行排他锁
```

==（这个也就是我们常说的当前读）==

### innodb行记录的三个隐藏列

```
innodb的每一个行记录的结构都有三个隐藏列：row_id、transaction_id、roll_pointer(滚动指针)，其中row_id是非必须的，只有当表中无主键是才需要用到它，官方文档https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html

transaction_id记录的是最后修改这条行数据的事务id
roll_pointer记录的是指向undolog中回滚段的日志记录
```

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210127073916.png)

### 版本链

```
这个也就是innodb的mvcc的本质
```

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210127074055.png)

### ReadView

```
它是innodb的mvcc机制使用的快照, 也就是我们常说的快照读使用的快照
```

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210127074325.png)

## 快照读与RR、RC

```
这个也就是RR、RC快照读取的原理,或者说innodb中mvcc的原理
```

![](F:\zehua\personalFiles\newStart\数据库Mysql\pictures\QQ截图20210127080732.png)

## 锁与事务

```
当我们在使用锁定读取（select lock in share mode/select for update）、update、delete时，就会使用到每个隔离级别下相应的锁来保证每种隔离级别下的特性
```

## 总结

```
锁、事务、当前读、快照读它们之间的关系就是：一致性读取是使用mvcc来保证的，而其他情况是使用锁来保证的，他们共同构成了不同隔离级别不同的特性
```

