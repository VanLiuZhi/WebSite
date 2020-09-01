---
weight: 1
title: "MySQL Sql sentence"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "MySQL Sql sentence"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [DataBase, Note]
categories: [DataBase]

lightgallery: true

toc:
  auto: false
---

原生语句的使用与学习

<!-- more -->

## 常见join

```
select * from table_a a left join table_b b on a.key = b.key

select * from table_a a inner join table_b b on a.key = b.key

select * from table_a a right join table_b b on a.key = b.key

select * from table_a a left join table_b b on a.key = b.key where b.key is NULL

select * from table_a a right join table_b b on a.key = b.key where a.key is NULL

select * from table_a a full outer join table_b b on a.key = b.key

select * from table_a a full outer join table_b b on a.key = b.key where a.key is NULL or b.key is NULL
```
查询的效果就是取两张表的join结果，有不同的join方法，字段会被组合在一行中，也就是笛卡尔积。

在某些数据库中，FULL JOIN 称为 FULL OUTER JOIN。

左连接以左边表为准，由于结果是两张表的字段，没匹配上的为null。 右连接类推，FULL JOIN 是 左右连接合并

inner join 取笛卡尔积

mysql: left join, left out join是一个东西，加上 inner join，只有3种，full不支持
用union 可以连接两张表，就是把B表拼接到A表后面，但是A,B必须字段数量相同
select table_a.id, table_a.table_b_id from table_a union select table_b.id, table_b.b_name from table_b;
这个例子用，虽然a表用的是table_b_id，b表是b_name，但是都是2个字段，这样就能拼接，但是b_name的数据被算到table_b_id这一列下

### 一些例子

select a.id from app_info a right join agents_profit_setting b on a.id = b.id;
注意使用了别名a为表app_info取别名，所以在其它需要用到表app_info都使用别名访问

select a.id, b.id as b_id from app_info a right join agents_profit_setting b on a.id = b.id;
返回a，b的id，并对b的id取别名b_id

## 函数

系统提供了一些通用函数，可以在语句中使用

### 时间函数

```
mysql> select now(), curdate(), sysdate(), curtime();
+---------------------+------------+---------------------+-----------+
| now()               | curdate()  | sysdate()           | curtime() |
+---------------------+------------+---------------------+-----------+
| 2019-03-07 17:17:44 | 2019-03-07 | 2019-03-07 17:17:44 | 17:17:44  |
+---------------------+------------+---------------------+-----------+
```

### concat 和 contcat_ws

concat用来连接字符串，contcat_ws用来以分隔符参数来连接字符串
`contcat_ws(',', 'a', 'b')  输出为：a,b`

### case 排序把特定字段放在结果前面

```sql
select * from `equipment` order by case when (id='ZJXCA007804' or id='ZJXCA000695') then 0 else 1 end ,ismonitor desc
```

```py
equipments.order_by(Case(When(id__in=[item.get('eqid', 0) for item in equipment_ids], then=0), default=1), '-ismonitor', 'id')
```

## 分页

limit y 分句表示: 读取 y 条数据
limit x, y 分句表示: 跳过 x 条数据，读取 y 条数据
limit y offset x 分句表示: 跳过 x 条数据，读取 y 条数据

第1页： 从第0个开始，获取20条数据
```sql
selete * from testtable limit 0, 20; 
selete * from testtable limit 20 offset 0;  
```

第2页： 从第20个开始，获取30条数据
```sql
selete * from testtable limit 20, 30; 
selete * from testtable limit 30 offset 20;  
```

## datetime 时间范围内查询

datetime可以直接传递字符串，它本质就是存储字符串

```sql
select COUNT(a.ID) from mondata.processlist as a where COLLECT_TIME BETWEEN 
STR_TO_DATE(
        '2020-04-08 00:00:00',
        '%Y-%m-%d %H'
    )
AND STR_TO_DATE(
    '2020-04-09 00:00:00',
    '%Y-%m-%d %H'
);
```

## 分组取最大值

1. 推荐标准用法

SELECT * from learn A inner join 
( SELECT course_id as course_id_child，MAX(learn_time) as learn_time_child FROM learn where user_id = '14201109' GROUP BY course_id) B 
ON A.course_id=B.course_id_child and A.learn_time=B.learn_time_child

learn表按course_id分组，取learn_time最大的，可以用上面的sql来实现，如果learn_time最大的有多个一样的，这些值都会留下来

实际示例:

"SELECT itemid, value, getclock FROM {table} a join ( SELECT itemid as itemid_child, MAX(getclock) as clock_child FROM {table} " \
"WHERE( {items_where} ) AND value !='0' AND getclock > {time_stamp} GROUP BY itemid) b " \
"ON a.itemid = b.itemid_child and a.getclock = b.clock_child ".format(table=table_name, items_where=items_where, time_stamp=time_stamp)

2. 不赞成的用法

这个用法会只取到最大的，如果不关心更多的字段，可以这么用，但是其实是有脏数据的(但是我只关心我想要的数据)

SELECT *
from learn
where learn.learn_time in (
			SELECT MAX(l.learn_time)
			FROM learn l
			where l.user_id = '14201109' 
			GROUP BY l.course_id)

3. 其它用法，大同小异

select * from test as a
where typeindex = (select max(b.typeindex)
from test as b
where a.type = b.type )

select
a.* from test a,
(select type,max(typeindex) typeindex from test group by type) b
where a.type = b.type and a.typeindex = b.typeindex order by a.type

select * from
(
select *,ROW_NUMBER() OVER(PARTITION BY type ORDER BY typeindex DESC) as num
from test
) t
where t.num = 1

SELECT * FROM Test t1 WHERE not EXISTS(select 1 from Test t2 WHERE t2.value>t1.value and t1.name = t2.name);
