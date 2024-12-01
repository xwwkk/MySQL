# Answer

## 1、取得每个部门最高薪水的人员名称
```sql
select 
	e.ename, e.sal, e.deptno
from
	emp e
join
	(select max(sal) as maxsal, deptno from emp group by deptno) m
on
	e.deptno = m.deptno and e.sal = m.maxsal;
```

## 2、哪些人的薪水在部门的平均薪水之上
```sql
select
	e.ename, e.sal 
from
	emp e
join
	(select avg(sal) as avgsal, deptno from emp group by deptno) a
on
	e.deptno = a.deptno and e.sal > a.avgsal
order by 
	sal;
```

## 3、取得部门中（所有人的）平均的薪水等级
```sql
select
	a.deptno, avg(a.grade)
from
    (select
        e.deptno ,s.grade
    from
        emp e
    join 
        salgrade s
    on
        e.sal between s.losal and s.hisal) a
group by
	a.deptno
order by
	a.deptno;
```

## 4、不准用组函数（Max），取得最高薪水（给出两种解决方案）
```sql
select
	sal
from
	emp
order by
	sal desc
limit 1;
```
```sql
select
	sal
from
	emp e1
where
	not exists(
		select 1
		from emp e2
		where e2.sal > e1.sal
		);
```

## 5、取得平均薪水最高的部门的部门编号（至少给出两种解决方案）
```sql
select
	a.deptno
from
    (select
        deptno, avg(sal) as avgsal
    from
        emp
    group by
        deptno) a
order by
	a.avgsal desc
limit 1;
```
```sql
select
	a1.deptno
from
	(select
		deptno, avg(sal) as avgsal
	from
		emp
	group by
		deptno) a1
where
	not exists(
		select 1
		from (select
				deptno, avg(sal) as avgsal
			from
				emp
			group by
				deptno) a2
		where a2.avgsal > a1.avgsal
	);
```

## 6、取得平均薪水最高的部门的部门名称
```sql
select
	d.dname
from
	dept d
join
	(select
	a.deptno
from
    (select
        deptno, avg(sal) as avgsal
    from
        emp
    group by
        deptno) a
order by
	a.avgsal desc
limit 1) n
on
	n.deptno = d.deptno;
```

## 7、求平均薪水的等级最低的部门的部门名称
```sql
select
	d.dname
from
	dept d
join
	(select
	a.deptno, avg(a.grade) avggrade
from
    (select
        e.deptno ,s.grade
    from
        emp e
    join 
        salgrade s
    on
        e.sal between s.losal and s.hisal) a
	group by
		a.deptno
	order by
		avggrade
	limit 1) n
on
	n.deptno = d.deptno;
```

## 8、取得比普通员工(员工代码没有在 mgr 字段上出现的)的最高薪水还要高的领导人姓名

```sql
select 
	a.ename, a.sal
from
	emp a
join
	(select
		a.sal
	from
		emp a
	where
        a.empno not in(
            select b.mgr
            from emp b
            where b.mgr is not null
            )
    order by
        a.sal desc
    limit 1) n
on
	a.sal > n.sal
where
	a.empno in(
		select
			b.mgr
		from 
			emp b
		where 
			b.mgr is not null);
```

## 9、取得薪水最高的前五名员工
```sql
select
	ename, sal
from 
	emp
order by
	sal desc
limit 5;
```

## 10、取得薪水最高的第六到第十名员工
```sql
select
	ename, sal
from 
	emp
order by
	sal desc
limit 5,5;
```

## 11、取得最后入职的 5 名员工
```sql
select
	ename, hiredate
from
	emp
order by
	hiredate desc
limit 5;
```

## 12、取得每个薪水等级有多少员工
```sql
select
	n.grade, count(n.sal) as countsal
from
	(select
        e.sal, s.grade
    from
        emp e
    join
        salgrade s
    on
        e.sal between s.losal and s.hisal) n 
group by
	n.grade
order by
	n.grade;
```

##  13、面试题  
```sql
select 
	n.sname
from(select
		a.sname, c.cteacher
	from
		(select
        	s.sname, sc.cno
        from
            s
        join
            sc
        on
            s.sno = sc.sno) a
        join
            c
        on
            a.cno = c.cno) n
where 
	n.cteacher = '黎明';
```
```sql
select
	xs.sname, cj.avggrade
from
	(select
		a.sname
    from 
        (select
        s.sname, sc.scgrade
        from
            s
        join
            sc
        on
            s.sno = sc.sno
        where
            sc.scgrade < 60) a
    group by
        a.sname
    having
        count(a.sname) >= 2) xs
join
	(select
        s.sname, n.avggrade
    from
        s
    join
        (select sc.sno, avg(sc.scgrade) avggrade from sc group by sc.sno) n
    on
        n.sno = s.sno) cj
on
	xs.sname = cj.sname;
```
```sql
select
	a.sname
from
	(select
        s.sname
    from
        s
    join
        sc
    on
        s.sno = sc.sno and sc.cno = '1') a
join
	(select
        s.sname
    from
        s
    join
        sc
    on
        s.sno = sc.sno and sc.cno = '2') b
on
	a.sname = b.sname;
```

## 14、列出所有员工及领导的姓名
```sql
select
	a.ename, ifnull(b.ename, '没有上级')
from
	emp a
left join
	emp b
on
	a.mgr = b.empno;
```

## 15、列出受雇日期早于其直接上级的所有员工的编号,姓名,部门名称
```sql
select
	yg.empno, yg.ename as ename_e, dept.dname
from
	(select
        a.empno, a.ename, a.deptno
    from
        emp a
    join
        emp b
    on
        a.mgr = b.empno
    where
        a.hiredate < b.hiredate) yg
join
	dept
on
	dept.deptno = yg.deptno;
```

## 16、列出部门名称和这些部门的员工信息,同时列出那些没有员工的部门.
```sql
select
	d.dname, e.* 
from
	dept d
left join
	emp e
on
	d.deptno = e.deptno;
```

## 17、列出至少有 5 个员工的所有部门
```sql
select
	d.dname, c.countd 
from
	dept d 
join
	(select
        deptno, count(deptno) countd
    from
        emp
    group by
        deptno) c
on
	d.deptno = c.deptno
where
	c.countd >= 5;
```

## 18、列出薪金比"SMITH"多的所有员工信息.
```sql
select
	e.*
from
	emp e
join
	(select
        sal
    from 
        emp 
    where
        ename = 'smith') s
on
	e.sal > s.sal;
```

## 19、列出所有"CLERK"(办事员)的姓名及其部门名称,部门的人数.
```sql
select
	t1.ename, t2.dname, t3.cc
from
	(select
        ename, deptno
    from
        emp 
    where
        job = 'clerk') t1
join
	dept t2
on
	t1.deptno = t2.deptno
join
	(select
        deptno,count(deptno) as cc
    from
        emp
    group by
        deptno) t3
on
	t1.deptno = t3.deptno;
```

## 20、列出最低薪金大于 1500 的各种工作及从事此工作的全部雇员人数.
```sql
select
	jb.job, jb.cj
from
	(select
        job, count(job) cj, min(sal) ms
    from
        emp
    group by
        job) jb
where
	jb.ms > 1500;
```

## 21、列出在部门"SALES"<销售部>工作的员工的姓名,假定不知道销售部的部门编号.
```sql	
select
	e.ename
from
	emp e
join
	(select 
        deptno
    from
        dept
    where
        dname = 'sales') s
on
	s.deptno = e.deptno;
```

## 22、列出薪金高于公司平均薪金的所有员工,所在部门,上级领导,雇员的工资等级.
```sql
select
	t1.ename '员工', t2.dname , t3.ename '领导', t4.grade
from
	emp t1
join
	dept t2
on
	t2.deptno = t1.deptno
left join 
	emp t3
on
	t3.empno = t1.mgr
join
	salgrade t4
on
	t1.sal between t4.losal and t4.hisal
where
	t1.sal > (select avg(sal) from emp);
```

## 23、列出与"SCOTT"从事相同工作的所有员工及部门名称.
```sql
select
	e.ename, d.dname
from
	emp e
join
	dept d
on
	e.deptno = d.deptno
where
	e.job = (select job from emp where ename = 'scott')
and
	e.ename <> 'scott';
```

## 24、列出薪金等于部门 30 中员工的薪金的其他员工的姓名和薪金.
```sql
select
	e.ename, e.sal, d.dname
from
	emp e
join
	dept d
on
	e.deptno = d.deptno
where
	e.sal in (select sal from emp where deptno =30) 
and
	e.deptno <> 30;
```

##  25、列出薪金高于在部门 30 工作的所有员工的薪金的员工姓名和薪金.部门名称.
```sql
select
	e.ename, e.sal, d.dname
from
	emp e
join
	dept d
on
	d.deptno = e.deptno
where
	e.sal > (select max(sal) from emp where deptno = 30);
```

## 26、列出在每个部门工作的员工数量,平均工资和平均服务期限.
```sql
SELECT
    d.dname,
    n.ce,
    n.asal,
    TIMESTAMPDIFF(
        YEAR,
        FROM_UNIXTIME(n.ah),
        NOW()
    ) AS year_difference
FROM
    dept d
JOIN
    (SELECT
         COUNT(empno) AS ce,
         AVG(sal) AS asal,
         AVG(UNIX_TIMESTAMP(hiredate)) AS ah,
         deptno
     FROM
         emp
     GROUP BY
         deptno) n
ON
    d.deptno = n.deptno;
```

## 27、列出所有员工的姓名、部门名称和工资。
```sql
select
	e.ename, d.dname, e.sal
from
	emp e
join
	dept d
on
	d.deptno = e.deptno;
```

## 28、列出所有部门的详细信息和人数

```sql
select
	d.*, ifnull(n.cc, 0) as '人数'
from
	dept d
left join
	(select deptno, count(empno) cc from emp group by deptno) n
on
	n.deptno = d.deptno;
```

## 29、列出各种工作的最低工资及从事此工作的雇员姓名
```sql
select
	*
from
	emp
where
	sal in (select min(sal) from emp group by job);
```

## 30、列出各个部门的 MANAGER(领导)的最低薪金
```sql
select
	n.deptno, min(n.sal)
from
	(select
        deptno, sal
    from
        emp
    where 
        job = 'manager') n
group by
	deptno
order by
	deptno;
```	

## 31、列出所有员工的年工资,按年薪从低到高排序
```sql
select
	ename, (sal*12 + ifnull(comm, 0)*12) as income
from
	emp
order by
	income;
```

## 32、求出员工领导的薪水超过 3000 的员工名称与领导名称
```sql
select
	a.ename, b.ename
from
	emp a
join
	emp b
on
	a.mgr = b.empno
where
	b.sal > 3000;
```

## 33、求出部门名称中,带'S'字符的部门员工的工资合计、部门人数
```sql
select
	d.dname, ifnull(n.ss,' '), ifnull(n.ce, 0)
from
	dept d
left join
	(select
        sum(sal) ss, count(empno) ce, deptno
    from
        emp
    where
		ename like '%s%'
    group by
        deptno) n
on
	d.deptno = n.deptno;
```

## 34、给任职日期超过 30 年的员工加薪 10%.
```sql
update emp set sal = sal * 1.1 where timestampdiff(year, hiredate, now()) > 40;
```
	
