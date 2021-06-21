CREATE TABLE Student(
	s_id VARCHAR(10),
	s_name VARCHAR(10),
	s_birth DATE,
	s_sex VARCHAR(10));


insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙⻛' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '⼥');
insert into Student values('06' , '吴兰' , '1992-03-01' , '⼥');
insert into Student values('07' , '郑⽵' , '1989-07-01' , '⼥');
insert into Student values('08' , '王菊' , '1990-01-20' , '⼥');

create table Course(
      C_Id varchar(10),
      C_name varchar(10),
      T_Id varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');

create table Teacher(
      T_Id varchar(10),
      T_name varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');

create table SC(
      S_Id varchar(10),
      C_Id varchar(10),
      score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87);
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
 
#1 查询” 01 “课程比” 02 “课程成绩高的学生的信息及课程分数
select s.*,sc1.c_id,sc1.score,sc2.c_id,sc2.score
from student as s 
join sc as sc1
on s.s_id = sc1.s_id
join sc as sc2
on s.s_id = sc2.s_id
 and sc1.c_id = '01'
 and sc2.c_id = '02'
where sc1.score > sc2.score

#1.1 查询同时存在”01″课程和”02″课程的情况
select s.*,a.c_id,a.score,b.c_id,b.score
from student as s 
join (select * from sc where c_id = '01' ) as a on s.s_id = a.s_id 
join (select * from sc where c_id = '02' ) as b on a.s_id = b.s_id

select * from 
    (select * from sc where sc.c_id = '01') as t1, 
    (select * from sc where sc.c_id = '02') as t2
where t1.S_Id = t2.S_Id

#1.2 查询存在”01″课程但可能不存在”02″课程的情况(不存在时显示为 null )
select a.s_id,a.c_id,b.s_id,b.c_id
from (select * from sc where c_id = '01') as a
left join (select * from sc where c_id = '02') as b
on a.s_id = b.s_id

#1.3 查询不存在“01”课程但存在”02″课程的情况
select a.c_id,b.s_id,b.c_id
from (select * from sc where c_id = '01') as a
right join (select * from sc where c_id = '02') as b
on a.s_id = b.s_id
where a.c_id is null

#2 查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
select s.s_id,s.s_name,avg(score)
from student as s 
join sc
on s.s_id = sc.s_id 
group by s.s_id,s.s_name 
having avg(score) >= 60

#3 查询在 SC 表存在成绩的学生信息
select distinct s.*
from student as s 
join sc 
on s.s_id = sc.s_id 
order by s.s_id 

#4 查询所有同学的学生编号、学生姓名、选课总数、所有课程的成绩总和
select s.s_id,s.s_name,count(sc.c_id),sum(sc.score)
from student as s 
left join sc
on s.s_id = sc.s_id 
group by s.s_id,s.s_name 
order by s.s_id

#4.1 查有成绩的学生信息
select distinct s.*
from student s 
left join sc 
on s.s_id = sc.s_id 
where score is not null 
order by s.s_id 

#5.查询「李」姓老师的数量
select count(*)
from teacher
where t_name like '李%'

#6.查询学过「张三」老师授课的同学的信息
select s.*
from student as s 
join sc on s.s_id = sc.s_id 
join course as c on sc.c_id = c.c_id 
join teacher as t on c.t_id = t.t_id 
where t.t_name = '张三'

#7.查询没有学全所有课程的同学的信息
select s.*
from student as s 
left join sc on s.s_id = sc.s_id 
group by s.s_id,s.s_name,s.s_birth,s.s_sex 
having count(sc.c_id) < (select count(*) from course)
order by s.s_id 

#8.查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
select distinct s.*
from student as s 
left join sc as sc1
on s.s_id = sc1.s_id 
where exists (select * from sc as sc2 where s_id = '01' and sc2.c_id = sc1.c_id)
and s.s_id != '01'

#9.查询和" 01 "号的同学学习的课程完全相同的其他同学的信息
select s_id from 
(select sc.s_id,
	   sc.c_id,
	   s1.c_id as exists_id from sc 
left join 
(select s_id, c_id from sc where s_id = '01') as s1 
on sc.c_id  = s1.c_id) as bigmap 
group by s_id 
having count(exists_id) = (select count(*) from sc where s_id = '01');

#10.查询没学过"张三"老师讲授的任一门课程的学生姓名
select *
from student as s 
where not exists 
(select * 
from sc
join course as c 
on sc.c_id = c.c_id 
join teacher as t 
on c.t_id = t.t_id 
where t.t_name = '张三'
 and sc.s_id = s.s_id)

 #11.查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
 select s.s_id,s.s_name,avg(sc.score)
 from student as s 
 join sc 
 on s.s_id = sc.s_id 
 group by s.s_id,s.s_name 
 having sum(case when sc.score < 60 then 1 else 0 end) >= 2
 
 #12.检索" 01 "课程分数小于 60，按分数降序排列的学生信息
 select s.*,sc.c_id, sc.score 
 from student as s 
 join sc 
 on s.s_id = sc.s_id 
 where c_id = '01' and score < 60
 order by sc.score desc 
             
#13 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
select s_id,
       min(case when c_id = '01'then score end) as chinese,
       min(case when c_id = '02'then score end) as math,
       min(case when c_id = '03'then score end) as english,
       avg(score) as avg     
from sc 
group by s_id 
order by avg desc 

#14 查询各科成绩最高分、最低分和平均分,以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，
优良率，优秀率 
及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
(要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列)
select c.c_id,c.c_name,max(sc.score),min(sc.score),avg(sc.score),
       avg(case when sc.score >= 60 then 1 else 0 end) as 及格率,
       avg(case when sc.score >= 70 and sc.score < 80 then 1 else 0 end) as 中等率,
       avg(case when sc.score >= 80 and sc.score < 90 then 1 else 0 end) as 优良率,
       avg(case when sc.score >= 90 then 1 else 0 end) as 优秀率,
       count(*) as 总人数
from course as c 
join sc 
on c.c_id = sc.c_id 
group by c.c_id,c.c_name 
order by count(1) desc,c.c_id 

#15 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺
select s_id,c_id,score,
       rank ( ) over ( partition by c_id order by score desc) as 排名
from sc 

#15.1 按各科成绩进行行排序，并显示排名， Score 重复时合并名次
select s_id,c_id,score,
       dense_rank ( ) over (partition by c_id order by score desc) as 排名
from sc 

#16.查询学生的总成绩，并进行排名，总分重复时保留名次空缺
#16.1 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺

select q.s_id, q.total, row_number () over () as rank from(
select sc.s_id, sum(sc.score) as total from sc
group by sc.s_id
order by total desc)q;

select sc.s_id, sum(sc.score) as total from sc
group by sc.s_id
order by total desc;
#17.统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比
select c.c_id,c.c_name,
      sum(case when score >= 0 and score <= 60 then 1 else 0 end) as "[60-0]",
      sum(case when score > 60 and score <= 70 then 1 else 0 end) as "[70-60]",
      sum(case when score > 70 and score <= 85 then 1 else 0 end) as "[85-70]",
      sum(case when score > 85 and score <= 100 then 1 else 0 end) as "[85-100]"     
from course as c 
join sc 
on c.c_id = sc.c_id 
group by c.c_id,c.c_name
order by c.c_id 

#18.查询各科成绩前三名的记录
select *
from sc 
where (select count(sc1.score) 
       from sc as sc1 
       where sc.c_id =sc1.c_id and sc.score < sc1.score) < 3
order by c_id,s_id 
             
#19.查询每门课程被选修的学生数
select c.c_id,c.c_name,count(sc.c_id) as 人数
from course as c 
left join sc 
on c.c_id = sc.c_id 
group by c.c_id,c.c_name 

#20.查询出只选修两门课程的学生学号和姓名
select s.s_id,s.s_name
from student as s 
left join sc 
on s.s_id = sc.s_id 
group by s.s_id,s.s_name 
having count(sc.c_id) = 2

#21.查询男生、女生人数
select s_sex,count(s_id) as 人数
from student 
group by s_sex 

#22.查询名字中含有「风」字的学生信息
select *
from student 
where s_name like '%风%';

#23.查询同名同性学生名单，并统计同名人数
select s_id,s_sex,count(s_id) as 人数
from student 
group by s_id,s_sex
having count(s_id) > 1

#24.查询 1990 年出生的学生名单
select *
from student 
where s_birth >= '1990-01-01' 
  and s_birth <= '1990-12-31'
  
#25.查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
select avg(score),c_id
from sc 
group by  c_id 
order by avg(score) desc,c_id

#26.查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
select s.s_id,s.s_name,avg(sc.score)
from student as s 
join sc 
on s.s_id = sc.s_id 
group by s.s_id,s.s_name 
having avg(sc.score) >= 85

#27.查询课程名称为「数学」，且分数低于 60 的学生姓名和分数
select s.s_name,sc.score
from student as s 
join sc 
on s.s_id = sc.s_id 
join course as c 
on sc.c_id = c.c_id 
where c.c_name = '数学'
  and sc.score < 60
  
28.查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
select s.*,c.c_id,sc.score 
from student as s 
left join sc on s.s_id = sc.s_id 
left join course as c on sc.c_id = c.c_id 

#29.查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
select s.s_name,c.c_name,sc.score
from student as s 
join sc on s.s_id = sc.s_id 
join course as c on sc.c_id = c.c_id 
where sc.score > 70

#30.查询不及格的课程
select c.c_id,c.c_name
from course as c 
join sc on c.c_id = sc.c_id 
where sc.score < 60
group by c.c_id,c.c_name 

#31.查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名
select s.s_id,s.s_name
from student as s 
join sc 
on s.s_id = sc.s_id 
where sc.c_id = '01'
  and sc.score > 80
  
#32.求每门课程的学生人数
select sc.c_id,count(*)
from sc 
group by sc.c_id
order by sc.c_id 

#33.成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
select s.*,sc.score 
from student as s 
join sc on s.s_id = sc.s_id 
join course as c on sc.c_id = c.c_id 
join teacher as t on c.t_id = t.t_id 
where t.t_name = '张三'
order by sc.score desc 
limit 1

#34.成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生
select a.*
from (select s.*,sc,score
      from student as s
      join sc on s.s_id = sc.s_id
      join course as c on c.c_id = sc.c_id
      join teacher as t on t.t_id = c.t_id
      where t.t_name = '张三') as a
where a.score = (select max(sc.score)
                 from student as s
                 join sc on s.s_id = sc.s_id
                 join course as c on c.c_id = sc.c_id
                 join teacher as t on t.t_id = c.t_id
                 where t.t_name = '张三')

#35.查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩
select distinct s1.*
from sc as s1
join sc as s2
on s1.s_id = s2.s_id 
where s1.score = s2.score 
  and s1.c_id != s2.c_id
  
#36.查询每门成绩最好的前两名
select s1.s_id,s1.c_id,s1.score
from sc as s1
left join sc as s2
on s1.c_id = s2.c_id and s1.s_id != s2.s_id and s1.score < s2.score 
group by s1.s_id,s1.c_id,s1.score 
having count(s1.score) < 2
order by s1.c_id,s1.s_id 

#37.统计每门课程的学生选修人数（超过 5 人的课程才统计）
select c_id,count(s_id) as 人数
from sc 
group by c_id 
having count(s_id)> 5
order by c_id 

#38.检索至少选修两门课程的学生学号
select s_id,count(c_id)
from sc 
group by s_id
having count(c_id)>= 2
order by s_id 

#39.查询选修了全部课程的学生信息
select s.*
from student as s 
join sc on s.s_id = sc.s_id 
group by s.s_id,s.s_name,s.s_birth,s.s_sex
having count(c_id) = (select count(c_id) from course)
order by s.s_id 

#40.查询各学生的年龄，只按年份来算
select s_name, year(now())-year(s_birth)
from student 

#41.按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一
select student.s_id as 学生编号,student.s_name  as  学生姓名,
TIMESTAMPDIFF(YEAR,student.s_birth,now() ()) as 学生年龄
from student

#42.查询本周过生日的学生
select *
from student 
where week(s_birth) = week(now())

#43.查询下周过生日的学生
select *
from student 
where week(s_birth) = week(now()) +1

#44.查询本月过生日的学生
select *
from student 
where month(s_birth) = month(now())

#45.查询下月过生日的学生
select *
from student 
where month(s_birth) = month(now()) +1


    

















