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
	a.deptno
from
    (select
        deptno, avg(sal) as avgsal
    from
        emp
    group by
        deptno) as a
where 
	a.avgsal > (select max(a.avgsal) from a);
```

## 6、取得平均薪水最高的部门的部门名称
```sql
select 
	d.dname
from
	dept
join
	
	