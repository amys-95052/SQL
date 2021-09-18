###### 查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数

```sql
select S.* ,s1,s2 
from Student S 
	 RIGHT JOIN
	 (select t1.SId SId,t1.score s1,t2.score s2
        from (select SId,score from SC where CId = '01') AS t1,
             (select SId,score from SC where CId = '02') AS t2
       		 where t1.SId > t2.SId and t1.score > t2.score) G
          ON S.SId = G.SId;
```

###### 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息

1. 查询’01‘同学所学的课程
2. 从SC表中查询所有同学的SId，如果其CId在1结果中
3. 从Student表中查询2得到的SId得学生信息，不包括’01‘同学

```sql
select Student.*
from Student,(select distinct SId
              from SC
              where CId in (select CId from SC where SId = '01')) AS t
where t.SId=Student.SId
```

###### 查询没有学全所有课程的同学的信息

学生和课程表求笛卡尔积，与SC right join,找出SC.SId is null

```sql
select DISTINCT student.*
from (select student.SId,course.CId from student,course ) as t1 
      LEFT JOIN SC as t2 
      on t1.SId=t2.SId and t1.CId=t2.CId,
      Student
where t2.SId is null
and   t1.SId=student.SId
```

###### 查询和" 01 "号的同学学习的课程完全相同的其他同学的信息

所学课程不相同的SId：Student表与'01'学生所学的课程SC.CId自然连接后（获得全集），与SC左连接，SC.SId is null（与全集没有交集的记录）

```sql
select Student.*
from Student
where SId not in (select distinct t1.SId
                  from (select Student.SId,t.CId
                        from (select CId from SC where SId='01') as t,
                        Student
                       ) as t1
                  left join SC
                  on t1.SId = SC.SId and t1.CId = SC.CId
 where SC.SId is null) and SId != '01'
```

###### 查询没学过"张三"老师讲授的任一门课程的学生姓名

EXISTS 相关子查询，查询学过张三老师课的学生

```sql
select Student.Sname
from Student
where SId not in (select SId
                  from SC
                  where EXISTS (select * 
                               from Teacher,Course
                               where Tname='张三' 
                                 and Teacher.TId=Course.TId 												 and SC.CId = Course.CId)
                 )
```

```sql
select * from student
where student.sid not in(
    select sc.sid from sc,course,teacher 
    where
        sc.cid = course.cid
        and course.tid = teacher.tid
        and teacher.tname= "张三"
);
```

###### 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

```sql
select Student.*,score_avg
from Student,
     (select SId,AVG(score) AS score_avg
      from SC
      where SId in (select SId from SC where score<60)
      group by SId
      having count(SId) >= 2
     ) as t2
where Student.SId=t2.SId
```

```sql
/*表连接，从SC表找出存在不及格的记录，然后分组统计，过滤*/
select Student.*,AVG(score)
from Student,SC
where Student.SId=SC.SId 
  and score<60 
group by Student.SId
having count(Student.SId)>=2
```

