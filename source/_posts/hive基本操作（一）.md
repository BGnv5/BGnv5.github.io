---
title: hive基本操作(一)
comments: true
date: 2020-04-07 00:02:36
tags: hive
categories: hive
description: hive的基本操作
---

包括Hive的DDL操作、DML操作以及Hive的join操作

<!--more-->

### Hive 基本操作（一）

#### DDL操作

##### 创建表

- 创建内部表
```
hive> create table if not exists student1 (sno int,sname string,age int,sex string) row format delimited fields terminated by ' ';
```
- 创建外部表
```
hive> create external table if not exists student (sno int,sname string,age int,sex string) row format delimited fields terminated by ' ' location '/user/external';
```
- 创建与已知表相同结构的表

```
##copy_student1表的结构和student1相同但是并不包含student1中的数据

hive> create table copy_student1 like student1;
```
- 创建分区表
```
hive> create table book(id int,name string) partitioned by (country string) row format delimited fields terminated by ' ';
```
- 创建分桶表
```
##桶操作是通过clustered by实现的，bucket中的数据可以通过sort by排序。
##分桶表不允许以外部文件方式导入数据，只能从另外一张表导入数据

//1.开启分桶机制
hive> set hive.enforce.bucketing=true;

//2.创建分桶表teacher，以id作为分桶机制，分为2个桶
hive> create table teacher(id int,name string,age int) clustered by(id) into 2 buckets row format delimited fields terminated by ' ';
```
- 创建Struct类型的表
```
##导入数据格式：1,zhou:30

hive> create table student_struct(id int,info struct<name:string,age:int>) row format delimited fields terminated by ',' collection items terminated by ':';
```
- 创建Array类型的表
```
##导入数据的格式：034,1:2:3:4

hive> create table course_array(name string,course_id array<int>) row format delimited fields terminated by ',' collection items terminated by ':';
```
- 创建Map类型的表
```
##导入数据的格式：1       job:80,team:60,person:70 

hive> create table employee(id string,perf map<string,int>) row format delimited fields terminated by '\t' collection items terminated by',' map keys terminated by ':';
```
##### 修改表

- 添加列
```
hive> alter table student1 add columns(address string,grade string);
```
- 修改列
```
##将test表中name字段重命名为sname,类型为string，放到age字段后面

hive> alter table test change name sname string  after age;
```
- 修改表名
```
hive> alter table students rename to student2;
```
- 修改分区名
```
hive> alter table book partition(country='us') rename to partition(country='US');
```
- 增加分区
```
hive> alter table book add partition(country='jp');
```
- 删除分区
```
hive> alter table book drop partition(country='jp');
```
- 修复分区
```
hive> msck repair table book;
```
##### 删除表
```
hive> drop table test1;
```
##### 显示命令
- 查看数据库
```
hive> show databases;
```
- 查看数据表
```
hive> show tables;
```
- 查看表的结构
```
hive> desc student1;
```
- 查看表中都有哪些分区
```
hive> show partitions book;
```
- 查看表的信息
```
hive> describe formatted table_name；
```
#### DML操作

##### Load

- 加载本地数据到普通表中
```
hive> load data local inpath '/home/bgnv5/data/student1.txt' into table student1;
```
- 加载本地数据到分区表中
```
hive> load data local inpath '/home/bgnv5/data/book.txt' into table book partition(country='cn');
```
- 加载HDFS中的文件到表中
```
hive> load data inpath'/user/hive/student1.txt' into table copy_student1;
```

##### insert

- 将表中的数据插入到一张表中
```
hive> insert overwrite table copy_student2 select *  from student1;
```
- 将表中的数据插入到一个分区中
```
hive> insert overwrite table book partition(country='jp') select id,name from book_input where country='jp';
```
- 将表中的数据插入到多张表中
```
hive> from student1 
    > insert overwrite table copy_student3 
    > select * 
    > insert overwrite table copy_student4 
    > select *;
```
- 将表中的数据插入到多个分区中
```
hive> from book_input
    > insert overwrite table book partition(country='英国') 
    > select id,name where country='英国' 
    > insert overwrite table book partition(country='俄罗斯') 
    > select id,name where country='俄罗斯';
```
- 使用动态分区将表中的数据自动插入到分区中
```
//1.开启动态分区，默认是false
hive> set hive.exec.dynamic.partition=true;

//2.开启允许所有分区都是动态的，否则必须要有静态分区才能使用。
hive> set hive.exec.dynamic.partition.mode=nonstrict;

//3.插入数据
hive> insert overwrite table book partition(country) select id,name,country from book_input;
```
- 将表中的数据插入到本地目录下
```
hive> insert overwrite local directory '/home/bgnv5/data_test' row format delimited fields terminated by ' ' select * from book_input;
```
- 将表中的数据插入到hdfs目录下
```
hive> insert overwrite directory '/stu' row format delimited fields terminated by ' ' select * from student1;
```
##### select

- 查询表的所有字段
```
hive> select * from student1;
```
- 查询表的某个字段属性
```
hive> select sname from student1;
```
- 查看某一具体分区
```
hive> select * from book where country='cn';
```
- where条件查询
```
hive> select * from student1 where sno>201501004 and address="北京";
```
- distinct去重查询
```
##查询student1表中age和grade都不相同的学生

hive> select distinct age,grade from student1;
```
- limit限制查询
```
##只显示4条记录

hive> select * from student1 limit 4;
```
- Struct类型查询
```
select id,info.name,info.age from student_struct;
```
- Array类型查询
```
select name,course_id[0],size(course_id) from course_array;
```
- Map类型查询
```
select id,pref['job'],size(pref) from employee;
```
- order by 排序查询
```
##全局排序，默认升序
##只有一个reduce，即使设置多个reduce个数hive在运行的时候也会设置成1个.
##可以通过设置hive.mapred.mode属性优化查询速度，设置此模式必须指定limit否则会报错

//hive> select * from student1 order by age;

hive> set hive.mapred.mode=strict;
hive> select * from student1 order by age limit 5;
```
- group by分组查询
```
##必须配合聚合函数使用,且可以同时做多个聚合操作，但是聚合操作的对象必须是同一列

hive> select count(*) from student1 group by sex;
```
- sort by 
```
##同一个reduce中的数据是有序的，不同的reduce则不能保证有序

//1.设置reduce个数
hive> set mapred.reduce.tasks=2;
//2.使用sort by 进行排序
hive> select * from student1 sort by age;
```
- distribute by分区排序
```
##按照指定的字段把数据划分到不同的reduce中
##与sort by 同时使用时要放到前面

//1.设置reduce个数
hive> set mapred.reduce.tasks=2;
//2.按照性别划分到不同的reduce中进行排序，
hive> select * from student1 distribute by age;
```
- cluster by簇排序
```
##只能升序
##包含了distribute by 和sort by 两个功能
##当 distribute by 和 sorts by 字段相同时，可使用 cluster by 方式替代，如：

hive> select * from student1 cluster by age;
等价于
hive> select * from student1 distribute by age sort by age;
```
- 分桶表的抽样查询
```
##假设桶数为n,则bucket x out of y on XXX表示为：
##从第x个桶开始抽样，抽取n/y个桶的数据
##例如，n=6,x=1,y=3抽取两个桶的数据，因此抽取的样本为1和4桶中的数据
 
hive> select * from teacher tablesample(bucket 1 out of 2 on name);
```

#### Hive的join操作

- 内关联 inner join
```
hive> select * from a;
OK
1	zhangsan
2	wangwu
3	lisi
Time taken: 0.23 seconds, Fetched: 3 row(s)

hive> select * from b;
OK
1	20
2	18
4	21
6	23
Time taken: 0.129 seconds, Fetched: 4 row(s)

hive> select a.id,a.name,b.age from a join b on(a.id=b.id);
OK
1	zhangsan	20
2	wangwu	18
Time taken: 75.971 seconds, Fetched: 2 row(s)
```
- 左外关联
```
##以左表为基础，如果右侧表可以和左侧表关联，则为右侧表值，否则为null

hive> select a.id,a.name,b.age from a left join b on(a.id=b.id);
OK
1	zhangsan	20
2	wangwu	18
3	lisi	NULL
Time taken: 51.883 seconds, Fetched: 3 row(s)
```
- 右外关联
```
##以右表为基础，如果左侧表可以和右侧表关联，则值为左侧表值，否则为NULL

hive> select b.id,a.name,b.age from a right join b on(a.id=b.id);
OK
1	zhangsan	20
2	wangwu	18
4	NULL	21
6	null	23
Time taken: 45.56 seconds, Fetched: 4 row(s)
```
- 全关联
```
##两个表进行关联，如果右表没有值补充NULL、如果左边没有值补充NULL

hive> select stu.sno,stu.sname,msg.age from stu full join msg on (stu.sno=msg.sno);
OK
1	zhangsan	20
2	wangwu	18
3	lisi	NULL
NULL	NULL	21
NULL	NULL	23
Time taken: 152.029 seconds, Fetched: 4 row(s)
```
- left semi join
```
##左边表返回key在右边表中存在的值对应的数据

hive> select * from a left semi join b on(a.id=b.id);
1	zhangsan
2	wangwu
```
- 笛卡尔积关联 cross join
```sql
hive> select a.id,a.name,b.age from a cross join b;
OK
1	zhangsan	20
1	zhangsan	18
1	zhangsan	21
1	zhangsan	23
2	wangwu	20
2	wangwu	18
2	wangwu	21
2	wangwu	23
3	lisi	20
3	lisi	18
3	lisi	21
3	lisi	23
Time taken: 43.367 seconds, Fetched: 12 row(s)
```