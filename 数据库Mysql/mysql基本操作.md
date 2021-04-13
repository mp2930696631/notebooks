# 基本操作

```
1、mysql8.0 my.ini配置文件默认的路径
C:\ProgramData\MySQL\MySQL Server 8.0


2、对于关联查询，在92语法中，是采用如下形式
select column1, ... 
from table1, table2, .....

3、笛卡尔积
当进行表关联的时候，不指定关联条件，产生的结果就是笛卡尔积
```

```sql
-- 登录mysql
mysql -u<用户名> -p<密码>

-- 连接远程数据库
mysql -h <ip> -P <port> -u <username> -p <password>

-- 切换数据库
use database;

-- 查看当前数据库下面的所有表格
show tables;

-- 查看变量
show variables like '%xxx%';(模糊匹配)

-- 创建用户变量
set @variable_name;

-- 查看系统变量
select @@xxx;

-- 查看用户变量
select @xxx;

-- 查看用户权限
show grants for username;

-- 查看mysql中的连接
show processlist;

-- 查看表结构
desc t_name;

-- 查看表索引
show index from t_name;

-- 查看mysql基本配置信息
status;

-- 获取表的创建语句
show create table t_name;

-- 在使用前缀索引之前，可以先进行索引区分度计算，从而得出前缀使用多少个字符合适
-- 索引区分度计算（索引的选择性）
select concat(count(*)/tt.total*100, '%') 区分度 from 
(select left(<ename>, 1) prefix_name from <emp> group by prefix_name) t
join 
(select count(*) total from <emp>) tt
只需要更换其中的ename、emp就可以了
-- 可以换成一个更简单的方法
select concat(count(distinct t.prefix_name)/count(*)*100, '%') 区分度 from (select left(ename, 1) prefix_name from emp) t;
```



# DML语言

## 种类——传说中的CRUD

```
insert、select、update、delete——传说中的CRUD
```

## 关键字

```
select、from、where、join、order by、group by、having、union等

可以使用别名的关键字：order by、group by、having
不可以使用别名的关键字：where
```

### 关键字的执行顺序

<kbd>from</kbd>><kbd>join</kbd>><kbd>where</kbd>><kbd>group by</kbd>><kbd>having</kbd>><kbd>select</kbd>><kbd>order by</kbd>(select和order by的顺序不是很确定,还有union，它的优先级也很低很低)

## select

### select 

```sql
作用：
查询

语法：
select column_name, .... from t_name where condition

示例：
select * from emp;
select empno from emp;
select * from emp where empno=7782
```



### select distinct

```sql
作用：
去除重复行

语法：
select distinct column_name from t_name;

示例：
select distinct deptno from emp;
```



### where

```sql
作用：
过滤（作用在整个记录上）

语法：
where condition

示例：
select * from emp where empno=7782;
```

```
where可以使用的条件运算符
=
<>不等于
>
<
>=
<=
between...and 在某个范围内
or
like 模糊查询
not like
in 指定针对某个列的多个可能的取值，in不仅使用于单列，还是使用于多列，例如(col1, col2,...) in ....
not in
is null
is not null
```

==注意==

where 条件中无法使用列别名，也无法使用聚集函数，所以出现了having

### and & or

```sql
作用：
基于一个以上的条件进行过滤

语法：
where condition01 and condition02....
where condition01 or condition02....

示例：
select * from emp where deptno=10 and sal<2000
```



## insert 

```sql
作用：
向表中新增记录

语法：
insert into t_name (column_name01, ....) values (v1, .....);(在指定列插入数据， 自增键自动加入)
insert into t_name values (v1, .....);(所有列都插入数据，必须包括自增键)
insert into t_name (column_name01, ....) values (v1, .....), (v1, .....), ......;(批量插入数据)

示例：
insert into testcons (dname, age , address,gender) values ("z", 1, "cc", "ddd");
```



## update

```sql
作用：
更新记录某些字段

语法：
update t_name set column01=value, ...... where condition;

示例：
update testcons set col01_vchar='fdsf' where id=2;
```

==注意==

如果未加where将更新所有记录



## delete

```sql
作用：
用于删除记录

语法：
delete from t_name where condition;
delete from t_name(删除该表的所有记录，只是删除记录，表结构啥的不变)

示例：
delete from testcons where id=11;
delete from testcons;(删除该表的所有记录)
```

### 与truncate的区别

```
1、对于innodb，如果使用delete from t_name删除所有记录，它不会立即释放磁盘空间，而myisam会。
2、如果使用truncate table t_name删除所有记录，不管是innodb还是myisam都会立即释放磁盘空间
3、所以，如果你想删除部分记录，使用delete，向删除全部数据，使用truncate
```

==注意==

如果未加where将删除所有记录

## truncate

```sql
作用：
用于删除表的所有记录，表结构不变

语法：
truncate table t_name;

示例：
......
```

# DDL语言

## 种类

```
create、alter、drop
```

## create

```
用于创建表、视图、数据库、索引等
```

### create table

```sql
作用：
用于创建表

语法：
create table t_name (
column01 data_type(size),
......
);

示例：
create table testable (
id int,
name varchar(10)
);
```

### create database

```sql
作用：
创建数据库

语法：
create database d_name;

示例：
create database mydatabase；
```

### create index

```sql
创建索引
```

#### create index

```sql
作用：
创建普通索引，运行索引列出现重复值

语法：
1、在建表时创建普通索引(匿名)【关键字key可以使用index关键字替换】
create table t_name(
column01 data_type(size),
.....
key (column_name, ...)
)

2、在建表时创建普通索引(命名)【关键字key可以使用index关键字替换】
create table t_name(
column01 data_type(size),
.....
key index_name (column_name, ...)
)

3、建表后创建普通索引
create index index_name on t_name (column_name, ....);

示例：
1、在建表时创建普通索引(匿名)
create table testable (
id int(1),
name varchar(10),
key (id)
);

2、在建表时创建普通索引(命名)
create table testable (
id int(1),
name varchar(10),
key p_id (id)
);

3、建表后创建普通索引
create table testable (
id int,
name varchar(10)
);

create index name_index on testable (name);
```

==注意==

当在建表时创建普通索引时，key关键字可以用index关键字替换

#### create unique index

```sql
作用：
创建唯一索引，不允许索引列出现重复值

语法：
create unique index index_name on t_name (column_name, .....);

示例：
create table testable (
id int,
name varchar(10)
);

create unique index name_index on testable (id);
```

### create view

```sql
作用：
创建视图

语法：
create view view_name as 
select column, .... from t_name where condition;

示例：
create view v1 as 
select name from testable;
```

==扩展==

```
mysql中的视图不是物化视图，只是相当于一张虚拟表，本身并不存储数据，当使用sql操作视图时，其数据还是从相关的表中查询出来的

实现视图的方式分为两种
一、合并算法：比如create view v1 as select * from user where sex=m;当我们要查询视图时，mysql会将select id,name from v1;合并成select id,name from user where sex=m
二、临时表：将视图的数据保存到一张临时表中
```

## alter

```
修改表结构：包括添加、删除或者修改列，或者删除索引
```

### 添加列

```sql
作用：
向表中添加列

语法：
alter table t_name
add
(
column01 datatype(size),
......
);

示例：
alter table testable 
add 
(
address varchar(20),
col1 varchar(30)
);
```

### 删除列

```sql
作用：
删除表中的列（好像只能一列一列的删除）

语法：
alter table t_name
drop column column_name;

示例：
alter table testable drop column address;
```

### 修改列的数据结构

```sql
作用：
修改列的数据结构(好像只能一列一列的修改)

语法：
alter table t_name
modify column 
column_name datatype(size) <约束>;

示例：
alter table testable 
modify column 
col1 varchar(25);
```

==注意==

这种语法会连约束信息一起修改，例如非空约束，如果修改语句后面没有加not null，那么修改后的列将不再具有非空约束

### 删除索引

```sql
作用：
删除索引（也可以使用drop index index_name on t_name）

语法：
alter table t_name 
drop index index_name

示例：
alter table testable drop index name_index;
```

### 表重命名

```sql
作用：
給数据库表重命名

语法：
alter table old_name rename to new_name
或者
rename table old_name to new_table

示例：
alter table new_emp rename to o_new_emp

rename table o_new_emp to a_new_emp
```

### 字段重命名

```sql
作用：
给表的字段重命名

语法：
alter table <表名> change <oldcolumn> <newcolumn> <数据类型> <约束>

示例：
alter table a_new_emp change ename new_ename varchar(25)
```

==注意==

这种语法会连约束信息一起修改，例如非空约束，如果修改语句后面没有加not null，那么修改后的列将不再具有非空约束

## drop

```
用于删除表、视图、索引、数据库等
```

### drop index

```sql
作用：
删除索引

语法：
drop index index_name on t_name

示例：
drop index id_index on testable;
```

### drop  table

```sql
作用：
删除表（表结构和数据都删除）

语法：
drop table table_name

示例：
.......
```

### drop database

```sql
作用：
删除数据库

语法：
drop database d_name

示例：
.....
```

### drop view

```sql
作用：
删除视图

语法：
drop view view_name;

示例：
drop view v1;
```

# DCL语言

```
管理用户、分配权限，关键字：create、grant、revoke
```

## 创建用户

```sql
作用：
创建用户

语法：
create user 'username'@'ip'
identified by 'password'
其中
username: 表示用户名
ip：表示允许用户登录的ip，%代表所有，如果写上localhost，那么该用户就无法远程登录了
password：表示密码

示例：
步骤一、使用root登录mysql
步骤二、创建用户，create user 'testUser'@'%' identified by '123456';
```

## 删除用户

```sql
作用：
删除用户

语法：
drop user 'username'@'ip'

示例：
drop user 'newUser'@'localhost';
```

## 用户账号修改

```sql
作用：
修改用户账号

语法：
(因为用户信息都是记录在mysql这个database中的user表中的，所以，我们可以通过updaya语句直接修改用户名)
update user set user='newUsername' where user='oldUsername'

示例：
步骤一、使用root登录mysql
步骤二、update user set user='newUser' where user='testUser';
步骤三、flush privileges;这样才会生效
```

## 用户密码修改——mysql8.0

```sql
作用：
mysql8.0修改用户密码

语法：
alter user 'username'@'ip' identified with mysql_native_password by 'newPassword';

示例：
步骤一、使用root登录mysql
步骤二、alter user 'newUser'@'localhost' identified with mysql_native_password by '456789';
步骤三、flush privileges;(可以不执行)
```

## 用户密码修改——mysql5.7

```sql
作用：
mysql5.7修改用户密码

语法：
1、grant priv_type on database.table to 'username'@'ip' identified by 'newPassword'  
其中
priv_type：表示赋予用户的权限（如select、update等）
database.table：表示用户可以在哪些表上使用这些权限

2、set password for 'username'@'ip' = password('newPassword');
其中
password()函数用来对密码加密（mysql8.0没有该函数了）

3、update user set password = password("newPassword") where user = "username"; 

4、alter user 'username'@'ip' identified by 'newPassword'

示例：
先使用root登录mysql，执行上面三条语句中的一条即可
......
alter user 'root'@'localhost' identified by 'root';
```

## 赋予权限

```sql
作用：
赋予用户权限

语法：
grant priv_type
on priv_level
to user01, ....
其中
priv_type：表示赋予用户的权限
用户操作的权限——create user
DDL权限——select/update.....
创建索引权限——index
所有——all

priv_level: 权限级别
*——当前数据库的所有表
*.*——表示所有数据库的所有表
dname.tname——表示某数据库下的某张表

user01：格式为'username'@'ip'

示例：
步骤一、grant create user on *.* to 'testUser'@'%';
步骤二、flush privileges;
```

## 撤销权限

```sql
作用：
赋予用户权限

语法：
revoke priv_type
on priv_level
from user01, ...
(也就是差不多和grant是相反的操作)

示例：
步骤一、revoke create user on *.* from testUser;
步骤二、flush privileges;
```

# 关联查询

```
1、内连接
两张表没有主副之分，只要是符合条件的语句都将查询出来

2、外连接（mysql不支持92语法的外连接）
两张表分为一主表、一副表，主表的信息将全部显示，而副表中如果没有相应的信息将显示为null

3、笛卡尔积
进行表关联时不加关联条件，显示出的结果集就是笛卡尔积

4、等值连接
顾名思义，连接条件为等量关系

5、非等值连接
顾名思义，连接条件为非等量关系

6、自连接
也就是一张表当成多张表来看,例如92语法中的
select * 
from emp e, emp m
```

## 92语法的关联查询

```sql
语法规则：
select column1, ... 
from table1, table2, .....
where join_condition <filter_condition>

92语法的关联条件和筛选条件都写在where子句中
```

## 99语法的关联查询

```sql
语法规则：
select column1, .....
from table01 
(inner)join table02 
on join_condition 
where filter_condition

（99语法还可以使用using关键字来进行等值连接，使用using需要两张表中有一模一样的关联列，using(deptno)=where emp.deptno=dept.deptno）

相比于92语法，99语法将关联条件写在on子句中，而筛选条件写在where子句中，层次清晰
```

==本文档中的关联查询只包括99语法==

# mysql高级用法

## limit

```sql
作用：
用于限制数据的输出，常用于分页

语法：
limit [offset, ] count_num;(不包括offset这条记录)

示例：
从第5条记录开始，输出3条记录
select * from emp limit 4, 3;
```

## like

```sql
作用：
用于模糊查询,常与通配符一起使用

语法：
where column like pattern;

示例：
select * from emp where ename like '%mi%';
```

## 通配符（支持正则表达式）

```sql
作用：
匹配某种模式

语法：
where column like pattern;
其中
pattern：
%表示0或者多个字符，
_(下划线)匹配一个字符，

还支持正则表达式，如果pattern是一个正则表达式，需要使用regexp/rlike（not regexp/not rlike）关键字

如果需要转义，使用\（也可以使用escape(转义字符)自定义转义字符，默认为\）

示例：
查询名称中包含a字符的
select * from emp where ename regexp '[a]';

查询名称中不包含a字符的
select * from emp where ename not rlike '[a]';
```

## in

```sql
作用：
允许你在where字句中规定多个值

语法：
where column in (v1, v2)

示例：
select * from emp where ename in ('smith', 'ward');
```

## between

```sql
作用：
范围查找

语法：
where column between v1 and v2;(v1~v2)

示例：
select * from emp where sal between 1500 and 4000;
```

## order by

```sql
作用：
针对某列或者多列对查询出的结果进行排序

语法：
select * from t_name order by column_name desc|asc, column_name desc|asc.....;

示例：
select * from emp order by sal;（默认升序 asc）
select * from emp order by sal desc;(降序)
select * from emp order by dept, sal;(先按dept排序，当dept相同时再按sal排序)
```

## group by

```sql
作用：
结果集分组

语法：
group by column
（select子句中不能出现分组字段以外的其他字段）

示例：
select deptno, count(*) as count_num from emp group by deptno
```

## having

```sql
作用：
分组条件过滤，可以使用聚集函数，也可以使用列别名

语法：
group by column having condition

示例：
select deptno, count(*) as count_num from emp group by deptno having count_num>4
```

## exists

```sql
作用：
只能是在子查询身上，如果子查询有记录存在则返回true，否则返回false

语法：
where exists subquery

示例：
select * from emp where exists (select * from dept where deptno=10);
将返回emp表中的全部数据，因为这个查询中的子查询一直有记录
```

## using

```sql
作用：
99语法中，可以使用using关键字来替代on关键字，前提是等值连接，且关联的列在两张表中都存在
如果使用using，那么该关联列将不属于任何一张表，我们在select子句中，将不再需要使用table.列名的格式了，直接写该列名即可

语法：
select column01, ...
from table01
join table02
using(common_column)

示例：
select * from emp 
join dept
using(deptno)
使用这种形式的话，查询结果集中将只有一列deptno，而如果使用on的话，结果集中将有两列deptno
```

## enum

```sql
作用：
枚举，底层使用数字来进行存储，而排序操作是按照定义enum使，列表中所给出的顺序

语法：
create table t_name (
column01 enum(str1, str2, ...),
......
);

示例：
create table testable (
id int(1),
gender enum('男','女')
);
insert into testable values (1,'男');
```



## 子查询

```sql
作用：
sql允许多层嵌套，子查询就是嵌套在其他查询中的查询。

也就是一句完整的sql中嵌套另一个完整的sql语句

理解子查询的关键就是将子查询的结果当做一张表来看待，子查询需要使用小括号括起来

```

==注意==

一条完整的sql语句可以包括的关键字有：select、from、where、join、order by、group by、having、union等

因为join、union本身就是与两张表发生联系，所以，他们两边的select并不是子查询

### 单行子查询

```sql
作用：
子查询的结果只有一条记录
可以使用单行比较运算符
=
>
>=
<=
<>

示例：
select * from emp 
where deptno=(select deptno from dept where deptno=10)

查询薪水在平均薪水之上的员工信息
select * from emp e
where e.sal > (select avg(sal) from emp)
```

### 多行子查询

```sql
作用：
子查询的结果有多条记录
可以使用集合比较运算符，例如
in

示例：
select * from emp 
where deptno in (select deptno from dept where deptno=10 or deptno=20)
```

## 别名

```sql
作用：
为表或者列起别名

语法：
表的别名：在表名称后面使用空格+别名或者使用as关键字
select * from t_name t_alias;
列的别名：在列名称后面使用空格+别名或者使用as关键字
select c_name c_alias from t_name;

示例：
表别名：
select * from emp e;
列别名：
select empno eno from emp;(输出的列个集中，列名不在是empno，而是eno)
```

## 关联查询

```
分为内连接和外连接
内连接：inner join（inner可以省略）
外连接：分为左外连接和右外连接
```

### join/ inner join

```sql
作用：
返回表中匹配的行

语法：
select * from table01 (inner)join table02 on join_condition where filter_condition

示例：
select * from emp e join dept d on e.deptno=d.deptno where e.sal > 2000;
```

### left join/ left outer join

```sql
作用：
左(外)连接，即返回左表全部行，右表若没有匹配的行返回null

语法：
select * from table01 left (outer) join table02 on join_condition where filter_condition

示例：
select * from emp e left join dept d on e.deptno=d.deptno where e.sal > 2000;
```

### right join/right outer join

```sql
作用：
右(外)连接，即返回右表全部行，左表若没有匹配的行返回null

语法：
select * from table01 right (outer) join table02 on join_condition where filter_condition

示例：
select * from emp e right join dept d on e.deptno=d.deptno;
```

==注意==

mysql中没有full (outer) join ,但是可以使用left join union right join 操作来达到相同的目的，请看mysq中的集合操作

## 集合操作

### union

```sql
作用：
合并结果集

语法：
union(包括去重操作)

示例：
select * from emp where sal>3000 union select * from emp where sal<1000 
```

### union all

```sql
作用：
合并结果集

语法：
union all(保留重复项)

示例：
select * from emp where sal>3000 union all select * from emp where sal<1000 
```

==注意==

mysql并不支持intersect（交集）操作或者差集操作

## insert into....select

```sql
作用：
从一张表中复制数据

语法：
insert into table01 (column01, ....)
select column01, ... from table02

示例：
create table tt (
id int,
name varchar(9)
);

insert into tt (id, name) 
select empno, ename from emp;
```

==注意==

mysql并不支持select into，但是可以使用insert into....select语法来复制表的数据

## 约束（constraints）

```sql
作用：
1、指定表的数据规则，约束表的行为，如果违反了约束，相应的行为就会被终止

2、约束可以在建表的时候规定，也可以在创建表之后使用alter规定

3、如果我们想添加唯一索引，其实他的语法就是添加一个唯一约束， 如果我们想添加一个普通索引，语法就用前面讲过的create index语法

语法：
建表时指定约束
create table t_name(
column01 data_type(size) constraint_name,
.....
)
其中
constraint_name可以由如下选择
not null
unique
primary key
foreign key：参照完整性，参照动作包括[cascade（级联操作），restrict（拒绝操作），set null（设为空），no action，set default]，mysql中可以为主键或者唯一约束添加外键约束
check
default

示例：
create table tt (
id int not null,
name varchar(9)
);
```

### not null

```sql
作用：
not null约束

语法：
建表时添加not null约束
create table t_name(
column01 data_type(size) not null,
.....
)

建表后添加约束
alter table t_name 
modify column_name data_type(size) not null

删除not null约束
alter table t_name 
modify column_name data_type(size) null

示例：
alter table tt 
modify name varchar(9) not null;
```

### unique

```sql
作用：
1、添加唯一约束

语法：
1、建表时添加唯一约束(匿名)
create table t_name(
column01 data_type(size),
.....
unique (column_name, ...)
)

2、建表时添加唯一约束(命名)
create table t_name(
column01 data_type(size),
.....
constraint unique_name unique (column_name, ...)
)

3、建表后添加约束(匿名)
alter table t_name 
add unique (column_name, ...);

4、建表后添加约束(命名)
alter table t_name 
add constraint unique_name unique (column_name, ...);

5、删除unique约束
alter table t_name 
drop index unique_name;

示例：
1、
create table tt (
id int,
name varchar(9),
unique (id, name)
);

2、
create table tt (
id int,
name varchar(9),
constraint id_name_u unique (id, name)
);

3、
alter table tt 
add unique (id,name);

4、
alter table tt 
add constraint id_name_u unique (id,name);

5、
alter table tt 
drop index id_name_u;
```

### primary key

==注意==

前提是必须指定not null约束(虽然在mysql中不指定也没有关系，但是同一指定吧)

```sql
作用：
主键约束

语法：
同unique，只需要将unique关键字改成primary key关键字即可，但是也略有不同
前提必须指定主键列not null约束
1、删除主键约束
alter table t_name
drop primary key

示例：
1、建表时创建主键（匿名）
create table tt (
id int not null,
name varchar(9),
primary key (id, name)
);

2、建表时创建主键（命名）
create table tt (
id int not null,
name varchar(9) not null,
constraint pk_id_name primary key (id, name)
);

3、建表后添加主键（匿名）
create table tt (
id int not null,
name varchar(9) not null 
);

alter table tt 
add primary key (id,name);

4、建表后添加主键（命名）
alter table tt 
add constraint pk_id_name primary key (id,name);

4、删除
alter table tt 
drop primary key
```

### foreign key

==注意==

mysql中可以为主键约束或者==唯一约束==添加外键约束

```sql
作用：
添加外键约束，也就是参照完整性约束

语法：
1、建表时（匿名）
create table t_name(
column01 data_type(size),
.....
foreign key column_name references other_table(column)
)

2、建表时（命名）
create table t_name(
column01 data_type(size),
.....
constraint fk_name foreign key column_name references other_table(column)
)

3、建表后（匿名）
alter table t_name
add foreign key (column) references t_name(column);

4、建表后（命名）
alter table t_name
add constraint fk_name foreign key (column) references t_name(column);

5、删除
alter table t_name
drop foregin key fk_name

示例：
1、
create table tt03 (
id int,
name varchar(9),
foreign key (id) references tt01(id)
);

2、
create table tt02 (
id int,
name varchar(9),
constraint fk_name foreign key (name) references tt01(name)
);

3、
alter table tt02
add foreign key (id) references tt01(id);

4、
alter table tt02
add constraint fk_id_ foreign key (id) references tt01(id);
（不知道为什么外键的命名不可以为fk_id）

5、
alter table tt02
drop foreign key fk_id_
```

### foreign key的参照动作

```sql
分为[cascade（级联操作），restrict（拒绝操作），set null（设为空），no action，set default]
在建表后也可添加参照动作，一个意思

1、cascade
create table tt01 (
id int,
name varchar(9),
unique (id),
unique (name)
);

create table tt02 (
id int,
name varchar(9),
constraint fk_i foreign key (id) references tt01(id) on delete cascade on update cascade 
);
当tt01表发生删除或者更新时，tt02表会发生相应的删除或者更新

2、restrict
create table tt01 (
id int,
name varchar(9),
unique (id),
unique (name)
);

create table tt02 (
id int,
name varchar(9)
);
alter table tt02
add constraint fk_i foreign key (id) references tt01(id) on delete restrict on update restrict 
当tt01表发生删除或者更新时，如果tt02表有引用，则tt01的操作会失败

3、set null
create table tt02 (
id int,
name varchar(9)
);
alter table tt02
add constraint fk_i foreign key (id) references tt01(id) on delete set null on update set null 

4、no action（同restrict）
create table tt02 (
id int,
name varchar(9)
);
alter table tt02
add constraint fk_i foreign key (id) references tt01(id) on delete no action on update no action

5、set default（这个好像也是删除/更新不了，不晓得为啥子）
create table tt02 (
id int default 0,
name varchar(9)
);
alter table tt02
add constraint fk_i foreign key (id) references tt01(id) on delete set default on update set default;
```



### check

```sql
作用：
用于限制列中的取值范围

语法：
匿名/命名规则同unique，只需要将unique关键字改成check关键字即可，规则写在小括号里

示例：
1、
create table tt (
id int,
name varchar(9),
check (id>2)
);

2、
create table tt (
id int,
name varchar(9),
constraint ck_id check (id>2 and name in ('nan', 'nv'))
);

3、
alter table tt 
add check (id>0);

4、
alter table tt 
add constraint ch_id check (id>0);

5、
alter table tt 
drop check ch_id;
```

### default

```sql
作用：
添加默认值

语法：
1、建表时创建
create table t_name (
column01 data_type(size) default default_value,
......
);

2、建表后创建
alter table t_name
alter column set default default_value;

3、删除
alter table t_name
alter column drop default;

示例：
1、create table tt (
id int,
name varchar(9) default 'zehua'
);

2、
alter table tt
alter name set default 'zehua';

3、
alter table tt
alter name drop default;
```

## 复制表

### 仅复制表结构+数据

```sql
作用：
复制表结构和数据，仅仅包括数据，但是不包括索引，约束等其他信息

语法：
create table new_table as select * from other_table

示例：
create table new_emp 
as
select * from emp
```

### 仅复制表结构（不包括数据）

```sql
作用：
复制表结构，仅仅包括表结构，但是不包括索引，约束、数据等其他信息

语法：
create table new_table as select * from other_table where 1=2

示例：
create table new_emp 
as
select * from emp where 1=2
```

### 复制所有

```sql
作用：
复制表的全部信息，包括索引，约束、数据等其他信息

语法：
其实就是分阶段进行
1、创建一张与另一张表一模一样的表（使用show create table t_name 获取需要复制的表的创建表的语句）
2、再向表中插入另一张表的全部数据


示例：
1、在命令行中使用show create table emp\G
2、获取上面创建表的语句后，修改表名，再执行创建表的语句
3、使用insert into.....select语句进行表数据复制即可
```

# mysql的数据类型

## 数值类型

|      类型      |                     大小                      |         范围（有符号）          |   范围（无符号）   |        用途         |
| :------------: | :-------------------------------------------: | :-----------------------------: | :----------------: | :-----------------: |
|   1、tinyint   |                    1 byte                     |           (-128，127)           |      (0，255)      |                     |
|  2、smallint   |                    2 bytes                    |        (-32 768，32 767)        |    (0，65 535)     |                     |
|  3、mediumint  |                    3 bytes                    |     (-8 388 608，8 388 607)     |  (0，16 777 215)   |                     |
| 4、int/integer |                    4 bytes                    | (-2 147 483 648，2 147 483 647) | (0，4 294 967 295) |                     |
|   6、bigint    |                    8 bytes                    |                                 |                    |                     |
|    6、float    |                    4 bytes                    |                                 |                    | 单精度<br/>浮点数值 |
|   7、double    |                    8 bytes                    |                                 |                    | 双精度<br/>浮点数值 |
|   8、decimal   | 对decimal(M,D)<br/> ，如果M>D，为M+2否则为D+2 |         依赖于M和D的值          |   依赖于M和D的值   |       小数值        |

```sql
1、有符号
create table t02
(
col01 tinyint,
col02 float(9, 2)
)

2、无符号
create table t01
(
col01 tinyint unsigned,
col02 float(9, 2) unsigned
)
```

==注意==

对于decimal类型，在mysql中是采用字符串进行存储的。所以它的不会丢失精度

## 日期和时间类型

|   类型    | 大小（bytes） |                  范围                   |        格式         | 用途 |
| :-------: | :-----------: | :-------------------------------------: | :-----------------: | :--: |
|   date    |       3       |          1000-01-01/9999-12-31          |     yyyy-MM-dd      |      |
|   time    |       3       |                                         |      HH:mm:ss       |      |
|   year    |       1       |                1901/2155                |        yyyy         |      |
| datetime  |       8       | 1000-01-01 00:00:00/9999-12-31 23:59:59 | yyyy-MM-dd HH:mm:ss |      |
| timestamp |       4       |        1970-01-01 00:00:00/2038         | yyyy-MM-dd HH:mm:ss |      |

```sql
create table t03
(
col01 date ,
col02 time,
col03 year,
col04 datetime,
col05 timestamp
)
insert into t03 values 
(
curdate(), 
date_format('2020/11/10 23:45:36', '%H:%i:%s'), 
year(now()),
now(),
'2020/11/10 23:45:36'
)
注：timestamp类型存储格式为yyyy-MM-dd HH:mm:ss
```

## 字符串类型

```sql
常用的就两个：char(n)和varchar(n)括号中的n表示字符而不是字节
```

==注意==

char(n)和varchar(n)括号中的n表示字符而不是字节

# mysql常用函数

## 聚集函数

```sql
（又叫组函数，也就是输入多个值，输出一个值）
count(*)、count(数字)、count(列名)都是可以的，count(列名)不包括null项
count(distinct 列名)
avg()
sum()
min()
max()
```

==注意==

在使用聚集函数的时候，select中不能出现除分组字段意外的其他字段（虽然这在mysql中不报错，但是这个额外的字段没有==任何意义==，并不能拿来使用）

## 数学函数

```sql
abs(x)
返回绝对值
select abs(-10)

pi()
圆周率，精确到小数点后6位

sqrt(x)
x的平方根

mod(x,y)
x%y

ceil(x)=ceiling(x)
不小于新的最小整数

floor(x)
不大于x的最大整数

round(x)
四舍五入，精确到整数

round(x,y)
四舍五入，精确小数点后y位

sign(x)
返回x的符号，-1表示负数， 0表示0， 1表示正数

pow(x,y)=power(x,y)
返回x^y

exp(x)
返回e^x

log(x)
返回以e为底的对数值(也就是自然对数)

log10(x)
返回以10位底的对数值

radians(x)
输入角度，返回弧度

degrees(x)
输入弧度，返回角度

sin(x)
输入弧度，输出正弦值
asin(x)
输入正弦值，输出弧度
cos(x)
acos(x)
tan(x)
atan(x)

cot(x)
输入弧度，输出余切
```

## 字符串函数

```sql
char_length(str)
返回str长度

concat(str1, str2, ....)
返回参数拼接后的字符串

concat_ws(x, str1, str2, ...)
返回参数拼接后的字符串，每个字符串之间都有一个x
select concat_ws(',', "a", 'b', 'c')=select concat('a', ',', 'b', ',', 'c')

insert(str1, x, len, str2)
字符串的第[x, x+len-1]的字符串，被str2取代
select insert("hello ward", 2, 3, 'ze')输出hzeo ward
也就是ell 被ze取代了

lower(str)/lcase(str)
upper(str)/ucase(str)

left(str, n)
str左边的n个字符

right(str, n)
str右边的n个字符

lpad(str1, len, str2)
输出长度为len的字符串，只有当str1长度大于len，才从str1的开头开始pad str2
str1长度小于len：
select lpad("hello", 30, "=o")输出==o==o==o==o==o==o==o==o=hello
str1长度大于len
select lpad("hello", 4, "==o")=left(str1, 4)输出hell

rpad(str1, len, str2)
输出长度为len的字符串，只有当str1长度大于len，才从str1的末尾开始pad str2
select rpad("hello", 30, "--o")输出hello--o--o--o--o--o--o--o--o-
str1长度大于len
select rpad("hello", 4, "==o")=left(str1, 4)输出hell

ltrim(str)
删除str左侧的字符串
rtrim(str)
删除str右侧的字符串

trim(str)
删除str两侧的字符串

trim(s1 from str)
删除str两侧的s1子字符串
select trim('s' from 'strfs')输出trf
select trim(' ' from '  str  ')=select trim('  str  ')

repeat(str, n)
str重复n次

space(n)
返回n个空格组成的字符串

replace(str, substr1, substr2)
str中所有substr1使用substr2代替

strcmp(str1, str2)
字符串比较，如果str1=str2，返回0， str1<str2,返回-1， str1>str2,返回1

substring(str, n, len)=mid(str, n, len)
截串，返回第[n, n+len-1]字符串
select substring("hello", 1, 2)输出he

locate(str1, str)=position(str1, str)=instr(str1, str)
返回子串str1在str中开始的位置

reverse(str)
字符串反转

elt(n, str1, str2, ...)
返回第n个字符串
```

## 日期时间函数

```sql
curdate()=current_date()
返回yyyy-MM-dd的当前时间

curtime()=current_time()
返回HH:mm:ss的当前时间

current_timestamp()=localtime()=now()=sysdate()
返回yyyy-MM-dd HH:mm:ss的时间格式

unix_timestamp()
以秒为单位的时间戳（1970-01-01 00:00:00到现在）

unix_timestamp(date)
以秒为单位的时间戳（1970-01-01 00:00:00到date）
select unix_timestamp('2021-1-21 12:56:56');

from_unixtime(unixTimestamp)
输入时间戳（秒），输出日期
select from_unixtime(1611205016)

utc_date()
返回utc（世界标准时间），格式为yyyy-MM-dd

utc_time()
返回utc时间，格式为HH-mm-ss

month(date)
返回日期中的月份
select month('2021-2-21 12:56:56')输出2

monthname(date)
返回月名称
select monthname('2021-2-21 12:56:56')输出February

dayname(date)
返回星期名称
select dayname('2021-1-21 12:56:56')输出Thursday

dayofweek(date)
返回当天在一周中的索引下标，1表示星期天，7表示星期六

weekday(date)
返回当天在一周中的索引下标,0表示周一，6表示周天

week(date)
返回当天位于一年中的第几周

year(date)
返回年份
select year('2021-1-21 12:56:56')

quarter(date)
返回季度
select quarter('2021-6-21 12:56:56')

minute(date)
返回时间对应的分钟数
select minute('2021-6-21 12:57:56')返回57

second(date)
返回时间对应的秒数

extract(type from date)
从日期中提取信息
type可以为
second
minute
hour
day
week
month
quarter
year
year_month
day_hour
day_minute
day_second
.....
(其中year_month等这种形式表示输出year+month)
select extract(year from '2021-6-21 12:56:56') 输出2021

date_add(date, interval expr type)
日期加法
type可以是以下类型
second
minute
hour
day
week
month
quarter
year
year_month
day_hour
day_minute
day_second
.....
select date_add('2021-6-21 12:56:56', interval 2 day)加两天

date_sub(date, interval 2 day)
日期减法
select date_sub('2021-6-21 12:56:56', interval 2 day)减两天

last_day(date)
返回当月最后一天的日期，也就是当月倒数第一天
select date_sub(last_day(now()), interval 1 day)也就是当月倒数第二天的日期

datediff(date01, date02)
返回两个日期之差，精确到天数date01-date02
select datediff(now(), '1998-01-29')
```

### date_format函数

```sql
作用：
以不同的格式显示日期/时间数据

语法：
date_format(data, format)
其中
data：合法日期
format：输出格式，如下表格

示例：
select date_format(now(), '%Y-%m-%d')输出2021-01-23
select date_format('2020/11/10 23:45:36', '%Y-%m-%d')输出2020-11-10
select date_format('2020/11/10 23:45:36', '%H:%i:%s')输出23:45:36
```

==format类型==

| 格式 |                      描述                      |
| :--: | :--------------------------------------------: |
|  %a  |                   缩写星期名                   |
|  %b  |                    缩写月名                    |
|  %c  |                    月，数值                    |
|  %D  |             带有英文前缀的月中的天             |
|  %d  |              月的天，数值(00-31)               |
|  %e  |               月的天，数值(0-31)               |
|  %f  |                      微秒                      |
|  %H  |                  小时 (00-23)                  |
|  %h  |                  小时 (01-12)                  |
|  %I  |                  小时 (01-12)                  |
|  %i  |               分钟，数值(00-59)                |
|  %j  |                年的天 (001-366)                |
|  %k  |                  小时 (0-23)                   |
|  %l  |                  小时 (1-12)                   |
|  %M  |                      月名                      |
|  %m  |                月，数值(00-12)                 |
|  %p  |                    AM 或 PM                    |
|  %r  |       时间，12-小时（hh:mm:ss AM 或 PM）       |
|  %S  |                   秒(00-59)                    |
|  %s  |                   秒(00-59)                    |
|  %T  |            时间, 24-小时 (hh:mm:ss)            |
|  %U  |        周 (00-53) 星期日是一周的第一天         |
|  %u  |        周 (00-53) 星期一是一周的第一天         |
|  %V  |  周 (01-53) 星期日是一周的第一天，与 %X 使用   |
|  %v  |  周 (01-53) 星期一是一周的第一天，与 %x 使用   |
|  %W  |                     星期名                     |
|  %w  |         周的天 （0=星期日, 6=星期六）          |
|  %X  | 年，其中的星期日是周的第一天，4 位，与 %V 使用 |
|  %x  | 年，其中的星期一是周的第一天，4 位，与 %v 使用 |
|  %Y  |                    年，4 位                    |
|  %y  |                    年，2 位                    |

## 条件判断函数

```sql
if(expr, v1, v2)
true输出v1, false输出v2
select if(1>5, 'a', 'b')输出b

isnull(v)
判断是否为null，为null输出1，否则输出0
select isnull(null)输出1

ifnull(v1, v2)
如果v1为空，输出v2，否则输出v1
select ifnull(null, 'b')输出b
select ifnull('a', 'b')输出a

case when then语句，当满足一个when子句后就停止了，不会在继续往后判断了
1、case expr 
	when v1 then result1
	when v2 then result2
	....
else result
end

select 
case 5-1
	when 1 then 'a'
	when 4 then 'b'
else 'c'
end
输出b
------------------------------
2、case  
	when condition then result1
	when condition then result2
	....
else result
end

select 
case 
	when 4>1 then 'a'
	when 4>2 then 'b'
else 'c'
end
输出a


```

## 系统信息函数

```sql
version()
返回mysql版本

connection_id()
返回当前用户的连接id（线程id）

user()
返回用户信息
select user()输出这样的格式root@localhost，也就是'username'@'ip'

charset(str)
返回str所用的字符集
select charset('a')输出这样的格式utf8mb4
```

## 加密函数

```sql
一、不可逆加密
md5(str)
str的md5加密
select md5('abd')

sha(str)
str的sha加密
select sha('str')

sha2(str，length)
str的sha2加密
length可以为224， 256， 384， 512， 0（写0表示256）
select sha2('str', 0)=select sha2('str', 256)
.......

二、加解密
to_base64(str)
使用base64加密

from_base64(crypt_str)
使用base64解密
.......
```

## 其他函数

```sql
format(x, n)
对数字进行格式化，四舍五入精确到小数点后n位
select format(5.621, 2)输出5.62

conv(x, from_, to_)
对数字x进行进制转换，从from_转换为to_
select conv(5, 10, 2)输出101

inet_aton(ip_str)
点分字节的ip格式转化为数字
select inet_aton('192.168.100.4')输出3232261124

inet_ntoa(ip_num)
数字转化为点分字节格式的ip
select inet_ntoa(3232261124)输出192.168.100.4

benchmark(count, expr)
计算重复执行expr count次消耗的时间，返回0表示执行很快，并不是表示不需要时间
select benchmark(5, 1+1)

convert(str using charset)
使用charset表示str
select convert('str' using 'utf8')
select convert('中华' using 'utf8mb4')
```

