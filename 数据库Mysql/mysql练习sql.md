## 基本操作(50题)

```sql
1、查询20号部门的所有员工信息
select * from emp where deptno=20

2、查询所有工作岗位为CLERK的员工的工号、员工名和部门名
select e.empno, e.ename, d.deptno, e.job from emp e
join dept d 
using(deptno)
where e.job='clerk'

3、查询奖金（COMM）高于工资（SAL）的员工信息
select * from emp e where e.comm>e.sal

4、查询工资高于奖金的20%的员工信息
select * from emp where emp.sal>emp.comm*1.2

5、查询10号部门中工种为MANAGER和20号部门中工种为CLERK的员工的信息
select * from emp 
where (emp.deptno=10 and emp.job='manager') or (emp.deptno=20 and emp.job='clerk')

6、查询所有工种不是MANAGER和CLERK，且工资大于或等于2000的员工的详细信息。
select * from emp where sal>=2000 and job not in ('manager', 'clerk')

7、查询有奖金的员工的不同工种
select job from emp where comm is not null group by job

8、查询所有员工工资和奖金的和
select (sal+ifnull(comm, 0)) sal_comm from emp

9、查询没有奖金或奖金低于100的员工信息
select * from emp where comm is null or comm < 100

10、查询各月倒数第3天入职的员工信息
select * from emp where hiredate=date_sub(last_day(hiredate), interval 2 day)

11、查询员工工龄大于或等于39年的员工信息
select * from emp where hiredate<=date_sub(now(), interval 39 year)

12、查询员工信息，要求以首字母大写的方式显示所有员工的姓名
select empno, insert(lower(emp.ename), 1, 1, upper(left(emp.ename, 1))) ename from emp

13、查询员工名正好为6个字符的员工的信息
select * from emp where char_length(emp.ename)=6

14、查询员工名字中不包含字母“S”员工
select ename, empno from emp where ename not like '%s%'（使用not like）

select empno, ename from emp where empno not in 
(select empno from emp where ename like '%s%')（使用not in）

15、查询员工姓名的第2个字母为“M”的员工信息
select * from emp where ename like "_m%"

16、查询所有员工姓名的前3个字符
select left(ename, 3) from emp

17、查询所有员工的姓名，如果包含字母“s”，则用“S”替换
select replace(lower(ename), 's', 'S') from emp

18、查询员工的姓名和入职日期，并按入职日期从先到后进行排列
select ename, hiredate from emp order by hiredate

19、显示所有的姓名、工种、工资和奖金，按工种降序排列，若工种相同则按工资升序排列
select ename, job, sal from emp order by job desc, sal asc

20、显示所有员工的姓名、入职的年份和月份，若入职日期所在的月份排序，若月份相同则按入职的年份排序
select ename, year(hiredate) year_, month(hiredate) month_ from emp order by year_, month_

21、查询在2月份入职的所有员工信息
select * from emp where month(hiredate)=2

22、查询所有员工入职以来的工作期限，用“**年**月**日”的形式表示
select 
empno, 
ename,  
floor(datediff(now(), hiredate)/365) 年,  
floor(mod(datediff(now(), hiredate), 365)/30) 月, 
ceil(mod(mod(datediff(now(), hiredate), 365), 30)) 天
from emp

23、查询至少有一个员工的部门信息
select * from dept where deptno in (select deptno from emp where empno is not null group by deptno)

24、查询工资比SMITH员工工资高的所有员工信息
select * from emp where sal > (select sal from emp where ename='smith')

25、查询所有员工的姓名及其直接上级的姓名
select e.ename, m.ename from emp e
join emp m
on e.mgr=m.empno

26、查询入职日期早于其直接上级领导的所有员工信息
select e.* from emp e
join emp m
on e.mgr=m.empno
where e.hiredate<m.hiredate

27、查询所有部门及其员工信息，包括那些没有员工的部门
select * from 
emp e right join dept d
using(deptno)

28、查询所有员工及其部门信息，包括那些还不属于任何部门的员工
select * from 
emp e left join dept d
using(deptno)

29、查询最低工资大于2500的各种工作
select
	job,
	min(sal) min_sal
from
	emp
group by
	job
having
	min_sal>2500

30、查询最低工资低于2000的部门及其员工信息
select
	*
from
	emp e
join dept d
		using(deptno)
where
	(e.deptno,
	e.sal) in (
	select
		deptno, min(sal) min_sal
	from
		emp
	group by
		deptno
	having
		min_sal<2000)
		
31、查询在SALES部门工作的员工的姓名信息
select * from emp where deptno = (select deptno from dept where dname='sales')

32、查询工资高于公司平均工资的所有员工信息
select * from emp where sal>(select avg(sal) from emp)

33、查询工资高于30号部门中工作的所有员工的工资的员工姓名和工资
select * from emp where emp.sal>(select max(sal) from emp where deptno=30)

34、查询每个部门中的员工数量、平均工资和平均工作年限
select count(*), avg(sal), avg(datediff(now(), hiredate))/365 from emp group by deptno

35、查询从事同一种工作但不属于同一部门的员工信息
select e.ename, e.deptno, job, m.ename, m.deptno from emp e 
join emp m
using(job)
where e.deptno<>m.deptno

36、查询各个部门的详细信息以及部门人数、部门平均工资
select * from dept d
join
(select deptno, count(*), avg(sal) from emp group by deptno) t
using(deptno)

37、查询10号部门员工以及领导的信息
select e.empno, e.ename, t.empno, t.ename from emp e
join
(select * from emp where deptno=10) t
on e.mgr=t.empno
where e.deptno=10

38、列出工资等于30号部门中某个员工工资的所有员工的姓名和工资
select * from emp where sal in (select sal from emp where deptno=30)

39、查询工资为某个部门平均工资的员工信息
select * from emp where sal in(select avg(sal) from emp group by deptno)

40、查询工资高于本部门平均工资的员工的信息
select * from emp e
join 
(select deptno ,avg(sal) avg_sal from emp group by deptno) t
using(deptno)
where e.sal>t.avg_sal

41、统计每个部门中各个工种的人数与平均工资
select deptno, job, count(*), avg(sal) from emp group by deptno, job

42、查询工资、奖金与10 号部门某个员工工资、奖金都相同的员工的信息
select * from 
(select empno, ename, sal, ifnull(comm, 0) comm from emp) tt
where (tt.sal, tt.comm)
in (select sal, ifnull(comm, 0) from emp where deptno=10)

43、查询部门人数大于5的部门的员工的信息
select * from emp where deptno in
(select deptno from emp group by deptno having count(*)>5)

44、查询所有员工工资都大于1000的部门的信息
select * from dept where deptno in
(select deptno from emp group by deptno having min(sal)>1000)

45、查询所有员工工资都大于1000的部门的信息及其员工信息
select
	*
from
	emp e
join (
	select
		*
	from
		dept
	where
		deptno in (
		select
			deptno
		from
			emp
		group by
			deptno
		having
			min(sal)>1000)) t
		using(deptno)

46、查询所有员工工资都在900~3000之间的部门的信息
select * from dept where deptno 
in
(select deptno from (select deptno, min(sal) min_sal, max(sal) max_sal from emp group by deptno) t
where t.min_sal>=900 and max_sal<=3000)

47、查询每个员工的领导所在部门的信息
select * from dept d
join
(select m.empno, m.deptno from emp e
join emp m
on e.mgr=m.empno) t
using(deptno)

48、查询30号部门中工资排序前3名的员工信息
select * from emp where deptno=30 order by sal desc limit 3

49、向emp表中插入一条记录，员工号为1357，员工名字为oracle，工资为2050元，部门号为20，入职日期为2002年5月10日
insert into emp (empno, ename, sal, deptno, hiredate) values (1357, 'oracle', 2050, 20, '2002-5-10')

50、向emp表中插入一条记录，员工名字为FAN，员工号为8000，其他信息与SMITH员工的信息相同。
insert into emp 
select 8000 empno, 'FAN' ename, t.* from 
(select job, mgr, hiredate, sal, comm, deptno from emp where ename='smith') t
```

### update中使用子查询

```sql
将各部门员工的工资修改为该员工所在部门平均工资加1000
update emp e set sal=
(select t.new_sal from (select deptno, avg(sal)+1000 new_sal from emp group by deptno) t where e.deptno = t.deptno)
```

## 进阶操作(20题)

```sql
1、查询部门编号为10和20的员工，要求使用exists
select * from emp e where exists 
(select * from dept d where d.deptno=e.deptno and (d.deptno=10 or d.deptno=20))

2、将每个员工名字前面加上'my name is '字符串
select concat('my name is ', ename) from emp;

3、查询雇佣期满六个月后是星期五的员工信息
select * from emp e 
join 
(select empno, dayname(date_add(hiredate, interval 6 month)) week from emp) t1 
on e.empno=t1.empno 
where t1.week='friday'

4、给不同的部门的员工涨薪，10部门涨薪10%，20部门涨薪20%，30部门涨薪30%
select empno, deptno, sal oldSal,
case deptno 
	when 10 then sal*(1+0.1)
	when 20 then sal*(1+0.2)
	when 30 then sal*(1+0.3)
	else sal
end newSal
from emp

5、查询82年的员工
select * from emp e
join
(select empno, right(year(hiredate), 2) year_str from emp) t1
on e.empno=t1.empno
where t1.year_str='82'

6、查询各部门工龄最大和最小的员工信息
select empno, ename, e.deptno, hiredate from emp e
join 
(select deptno, min(hiredate) min_hi, max(hiredate) max_hi from emp group by deptno) t1
on (e.hiredate=t1.min_hi or e.hiredate=t1.max_hi) and e.deptno=t1.deptno

7、查询雇员名称以及薪水等级
select e.ename, e.sal, sg.grade  from emp e
join
salgrade sg
on e.sal between sg.losal and sg.hisal

8、查询雇员中哪些人是经理人（使用子查询和in关键字）
select * from emp e
where e.empno in (select distinct mgr from emp where mgr is not null)

9、查询雇员中哪些人是经理人（使用关联查询）
select distinct e.* from emp e
join emp m
on e.empno=m.mgr 

10、每个部门平均薪水的等级
select t1.deptno, sg.grade from salgrade sg
join
(select deptno, avg(sal) avg_sal from emp group by deptno) t1
on t1.avg_sal between sg.losal and sg.hisal

11、求各部门平均的薪水等级（注意与22第的区别，这个是求薪水等级的平均值）
select t.deptno, avg(t.grade) from 
(select e.deptno deptno, sg.grade grade from emp e 
join salgrade sg
on e.sal between sg.losal and sg.hisal) t group by t.deptno 

12、求薪水最高的第6-10名员工
select * from emp order by sal desc limit 5, 5

13、求平均薪水最高的部门的部门编号
select * from (select deptno, avg(sal) avg_sal from emp group by deptno) t
where t.avg_sal = 
(select max(avg_sal) from (select deptno, avg(sal) avg_sal from emp group by deptno) tt ) 

使用视图
create view my_view 
as
select deptno, avg(sal) avg_sal from emp group by deptno

select * from my_view
where avg_sal = 
(select max(avg_sal) from my_view) 

drop view my_view

14、求平均薪水最高的部门的部门名称
create view my_view 
as
select deptno, avg(sal) avg_sal from emp group by deptno

select d.dname, deptno from dept d
join
(select * from my_view
where avg_sal = 
(select max(avg_sal) from my_view) ) t
using(deptno)

drop view my_view

15、求平均薪水的等级最低的部门的部门名称
create view my_view
as
select deptno, sg.grade  from (select deptno, avg(sal) avg_sal from emp group by deptno) t
join salgrade sg
on t.avg_sal between sg.losal and sg.hisal

select * from dept where deptno in 
(select deptno from my_view where grade=(select min(grade) from my_view))

drop view my_view

16、求部门经理人中平均薪水最低的部门名称
解析：先求出经理人，在按部门分组
create view my_view
as
select deptno, avg(sal) avg_sal from 
(select * from emp e where e.empno in (select distinct mgr from emp where mgr is not null)) t
group by deptno

select * from dept where deptno in 
(select deptno from my_view where avg_sal=(select min(avg_sal) from my_view))

drop view my_view

17、求比普通员工的最高薪水还要高的经理人名称
解析：分别求出经理人、普通员工
select * from (select * from emp e where e.empno in (select distinct mgr from emp where mgr is not null)) t
where t.sal>
(select max(sal) from emp e where e.empno not in (select distinct mgr from emp where mgr is not null))

18、求薪水最高的前5名雇员
select * from emp order by sal desc limit 5

19、求薪水最高的第6到第10名雇员
select * from emp order by sal desc limit 5, 5

20、求最后入职的5名员工
select * from emp order by hiredate desc limit 5
```

## 高阶操作(50题)

### 1、表结构

```
–1.学生表 
Student(s_id,s_name,s_birth,s_sex) –学生编号,学生姓名, 出生年月,学生性别 
–2.课程表 
Course(c_id,c_name,t_id) – –课程编号, 课程名称, 教师编号 
–3.教师表 
Teacher(t_id,t_name) –教师编号,教师姓名 
–4.成绩表 
Score(s_id,c_id,s_score) –学生编号,课程编号,分数
```

### 2、测试数据

```sql
-- 建表
-- 学生表
CREATE TABLE `Student`(
    `s_id` VARCHAR(20),
    `s_name` VARCHAR(20) NOT NULL DEFAULT '',
    `s_birth` VARCHAR(20) NOT NULL DEFAULT '',
    `s_sex` VARCHAR(10) NOT NULL DEFAULT '',
    PRIMARY KEY(`s_id`)
);
-- 课程表
CREATE TABLE `Course`(
    `c_id`  VARCHAR(20),
    `c_name` VARCHAR(20) NOT NULL DEFAULT '',
    `t_id` VARCHAR(20) NOT NULL,
    PRIMARY KEY(`c_id`)
);
-- 教师表
CREATE TABLE `Teacher`(
    `t_id` VARCHAR(20),
    `t_name` VARCHAR(20) NOT NULL DEFAULT '',
    PRIMARY KEY(`t_id`)
);
-- 成绩表
CREATE TABLE `Score`(
    `s_id` VARCHAR(20),
    `c_id`  VARCHAR(20),
    `s_score` INT(3),
    PRIMARY KEY(`s_id`,`c_id`)
);
-- 插入学生表测试数据
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('08' , '王菊' , '1990-01-20' , '女');
-- 课程表测试数据
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
-- 教师表测试数据
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
-- 成绩表测试数据
insert into Score values('01' , '01' , 80);
insert into Score values('01' , '02' , 90);
insert into Score values('01' , '03' , 99);
insert into Score values('02' , '01' , 70);
insert into Score values('02' , '02' , 60);
insert into Score values('02' , '03' , 80);
insert into Score values('03' , '01' , 80);
insert into Score values('03' , '02' , 80);
insert into Score values('03' , '03' , 80);
insert into Score values('04' , '01' , 50);
insert into Score values('04' , '02' , 30);
insert into Score values('04' , '03' , 20);
insert into Score values('05' , '01' , 76);
insert into Score values('05' , '02' , 87);
insert into Score values('06' , '01' , 31);
insert into Score values('06' , '03' , 34);
insert into Score values('07' , '02' , 89);
insert into Score values('07' , '03' , 98);
```

### 3、测试题

```sql
1、查询"01"课程比"02"课程成绩高的学生的信息及课程分数
select
	*
from
	student
join (
	select
		*
	from
		(
		select
			s_id, t1.s_score t1score, t2.s_score t2score
		from
			(
			select
				*
			from
				score
			where
				c_id = '01') t1
		join (
			select
				*
			from
				score
			where
				c_id = '02') t2
				using(s_id)) tt
	where
		tt.t1score>tt.t2score) ttt
		using(s_id)
		
2、查询"01"课程比"02"课程成绩低的学生的信息及课程分数
select
	*
from
	student
join (
	select
		*
	from
		(
		select
			s_id, t1.s_score t1score, t2.s_score t2score
		from
			(
			select
				*
			from
				score
			where
				c_id = '01') t1
		join (
			select
				*
			from
				score
			where
				c_id = '02') t2
				using(s_id)) tt
	where
		tt.t1score<tt.t2score) ttt
		using(s_id)
		
3、查询平均成绩大于等于60分的同学的学生编号和学生姓名和平均成绩
select * from student stu
join
(select s_id, avg(s_score) avg_sco from score group by s_id having avg_sco>=60) t
using(s_id)

4、查询平均成绩小于60分的同学的学生编号和学生姓名和平均成绩 (包括有成绩的和无成绩的) 
解析：分别查出有成绩的和无成绩的，（无成绩的平均值用0代替
select * from student stu
join
(select s_id, avg(s_score) avg_sco from score group by s_id having avg_sco<60) t
using(s_id)
union 
select *, 0 avg_sco from student where s_id not in (select distinct s_id from score)

5、查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩
解析：注意使用left join，因为有以为同学没有选课
select s_id, t1.s_name, t2.count_, t2.sum_sco from (select s_id, s_name from student) t1
left join
(select s_id, count(*) count_, sum(s_score) sum_sco from score group by s_id) t2
using(s_id)

6、查询"李"姓老师的数量
select count(*) from teacher where t_name like '李%'

7、查询学过"张三"老师授课的同学的信息 
select t.* from 
(select * from student stu
join score sco
using(s_id)) t
where t.c_id 
in
(select cou.c_id from course cou
join 
(select t_id from teacher where t_name="张三") tt
using(t_id))

8、查询没学过"张三"老师授课的同学的信息
explain
select
	*
from
	student
where
	s_id not in (
	select
		stu.s_id
	from
		student stu
	join score sco
			using(s_id)
	where
		sco.c_id in (
		select
			cou.c_id
		from
			course cou
		join (
			select
				t_id
			from
				teacher
			where
				t_name = "张三") tt
				using(t_id)))

9、查询学过编号为"01"并且也学过编号为"02"的课程的同学的信息
select
	stu.*
from
	student stu
join (
	select
		s_id, sco1.c_id sco1_cid, sco2.c_id sco2_cid
	from
		score sco1
	join score sco2
			using(s_id)) t
		using(s_id)
where
	sco1_cid = '01'
	and sco2_cid = '02'

10、查询学过编号为"01"但是没有学过编号为"02"的课程的同学的信息
select * from student where s_id 
in 
(select s_id from score 
where c_id='01' and s_id not in
(select s_id from score where c_id='02'))
```



## 行转列(4题)

==关键==

关键在于：选择一个列，然后分情况将这个列进行拆分，每一个情况对应一个case when，最后再使用==聚集函数进行合并==

```sql
一、
create table test(
   id int(10),
   type int(10) ,
   t_id int(10),
   value varchar(5),
   primary key (id)
);
insert into test values(100,1,1,'张三');
insert into test values(200,2,1,'男');
insert into test values(300,3,1,'50');

insert into test values(101,1,2,'刘二');
insert into test values(201,2,2,'男');
insert into test values(301,3,2,'30');

insert into test values(102,1,3,'刘三');
insert into test values(202,2,3,'女');
insert into test values(302,3,3,'10');
/*
请写出一条查询语句结果如下：

姓名      性别     年龄
--------- -------- ----
张三       男        50
*/
id     type t_id value
100    1    1    张三
解析：
1、我们可以选取type这一列
2、由表数据可知，type一共分为三种情况，也就是对于三个case when
3、使用max对结果进行合并
select 
max(case type
	when 1 then value 
end) 姓名,
max(case type
	when 2 then value
end) 性别,
max(case type
	when 3 then value
end) 年龄
from test 
group by t_id

二、
create table tmp
(
rq varchar(10),
shengfu varchar(5)
);

insert into tmp values('2005-05-09','胜');
insert into tmp values('2005-05-09','胜');
insert into tmp values('2005-05-09','负');
insert into tmp values('2005-05-09','负');
insert into tmp values('2005-05-10','胜');
insert into tmp values('2005-05-10','负');
insert into tmp values('2005-05-10','负');
/*如果要生成下列结果, 该如何写sql语句?

     rq     胜 负
2005-05-09 2 2
2005-05-10 1 2
*/
解析：
1、有表的数据可以发现，原数据表并没有出现数字。所以，我们先使用分组count
2、第一步获取的表形式就像第一题那样了，可以按照第一题的思路继续往下做
select rq, 
max(case shengfu 
	when '胜' then count_
end) 胜,
max(case shengfu 
	when '负' then count_
end) 负
from
(select rq, shengfu, count(*) count_ from tmp group by rq, shengfu) t group by rq

三、
create table student_score
(
  name    VARCHAR(20),
  subject VARCHAR(20),
  score   float(4,1)
);
insert into student_score (NAME, SUBJECT, SCORE) values ('张三', '语文', 78.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('张三', '数学', 88.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('张三', '英语', 98.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('李四', '语文', 89.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('李四', '数学', 76.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('李四', '英语', 90.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('王五', '语文', 99.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('王五', '数学', 66.0);
insert into student_score (NAME, SUBJECT, SCORE) values ('王五', '英语', 91.0);

1、得到类似下面的结果
姓名   语文  数学  英语

王五    89    56    89

select name, 
max(case subject
	when '语文' then score
end) 语文,
max(case subject
	when '数学' then score
end) 数学,
max(case subject
	when '英语' then score
end) 英语
from student_score group by name

2、有一张表，里面有3个字段：语文，数学，英语。其中有3条记录分别表示语文70分，数学80分，英语58分，请用一条sql语句查询出这三条记录并按以下条件显示出来（并写出您的思路）：  
   大于或等于80表示优秀，大于或等于60表示及格，小于60分表示不及格。  
       显示格式：  
       语文              数学                英语  
       及格              优秀                不及格    
------------------------------------------
此题需要在第一问的基础上做
create view my_view
as
(select 
max(case subject
	when '语文' then score
end) 语文,
max(case subject
	when '数学' then score
end) 数学,
max(case subject
	when '英语' then score
end) 英语
from student_score group by name)

select 
case 
	when 语文>=80 then "优秀"
	when 语文>=60 then "及格"
	when 语文<60 then "不及格"
end 语文,
case 
	when 数学>=80 then "优秀"
	when 数学>=60 then "及格"
	when 数学<60 then "不及格"
end 数学,
case 
	when 英语>=80 then "优秀"
	when 英语>=60 then "及格"
	when 英语<60 then "不及格"
end 英语
from my_view

drop view my_view

四、
create table yj01(
       month_ varchar(10),
       deptno int(10),
       yj int(10)
)

insert into yj01(month_,deptno,yj) values('一月份',01,10);
insert into yj01(month_,deptno,yj) values('二月份',02,10);
insert into yj01(month_,deptno,yj) values('二月份',03,5);
insert into yj01(month_,deptno,yj) values('三月份',02,8);
insert into yj01(month_,deptno,yj) values('三月份',04,9);
insert into yj01(month_,deptno,yj) values('三月份',03,8);

create table yjdept(
       deptno int(10),
       dname varchar(20)
)

insert into yjdept(deptno,dname) values(01,'国内业务一部');
insert into yjdept(deptno,dname) values(02,'国内业务二部');
insert into yjdept(deptno,dname) values(03,'国内业务三部');
insert into yjdept(deptno,dname) values(04,'国际业务部');

请用一个sql语句得出结果
从table1,table2中取出如table3所列格式数据，注意提供的数据及结果不准确，
只是作为一个格式向大家请教。
 
table1

月份mon 部门dep 业绩yj
-------------------------------
一月份      01      10
一月份      02      10
一月份      03      5
二月份      02      8
二月份      04      9
三月份      03      8

table2

部门dep      部门名称dname
--------------------------------
      01      国内业务一部
      02      国内业务二部
      03      国内业务三部
      04      国际业务部

1、table3 （result）

部门dep 一月份      二月份      三月份
--------------------------------------
      01      10                  
      02                  10      8
      03                 5        8
      04                          9

------------------------------------------
select deptno, 
max(case month_
	when '一月份' then yj
end) 一月份,
max(case month_
	when '二月份' then yj
end) 二月份,
max(case month_
	when '三月份' then yj
end) 三月份
from yj01 group by deptno

2、table3 （result）

部门名称 		一月份      二月份      三月份
--------------------------------------
国内业务一部      10                  
国内业务二部                 10      		8
国内业务三部                 5        	8
国际业务部                          		 9

------------------------------------------
解析：看到这个表的结果，可能首先想到的是先进行table1和table2的关联，在进行行转列，但是我们也可以先进行行转列，再进行关联
select dname,t.一月份, t.二月份, t.三月份  from yjdept
join
(select deptno, 
max(case month_
	when '一月份' then yj
end) 一月份,
max(case month_
	when '二月份' then yj
end) 二月份,
max(case month_
	when '三月份' then yj
end) 三月份
from yj01 group by deptno) t
using(deptno)
```

