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


