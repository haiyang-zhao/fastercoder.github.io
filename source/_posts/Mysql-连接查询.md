title: Mysql 连接查询
date: 2018-06-19 09:40:50
categories: SQL
tags: Mysql
---
[Join 查询有三种](http://www.runoob.com/mysql/mysql-join.html)

* INNER JOIN （内连接|等值连接） ： 获取连个表中 满足字段匹配关系的记录，相当取交集。
* LEFT JOIN (左连接) ：查询左边所有值，满足所有匹配关系的记录，即使右表中没有对应的的匹配记录。
* RIGHT JOIN （有连接）
：和LEFT JOIN 相反，查询右边的所有值，即使左边中没有对应的匹配记录。

两张表如下
  
    Database changed
    MariaDB [test_db]> show tables;
    +-------------------+
    | Tables_in_test_db |
    +-------------------+
    | runoob_tbl        |
    | tcount_tbl        |
    +-------------------+
    
    
    MariaDB [test_db]> select * from runoob_tbl;
    +-----------+---------------+---------------+-----------------+
    | runoob_id | runoob_title  | runoob_author | submission_date |
    +-----------+---------------+---------------+-----------------+
    |         1 | 学习 PHP      | 菜鸟教程      | 2017-04-12      |
    |         2 | 学习 MySQL    | 菜鸟教程      | 2017-04-12      |
    |         3 | 学习 Java     | RUNOOB.COM    | 2015-05-01      |
    |         4 | 学习 Python   | RUNOOB.COM    | 2016-03-06      |
    |         5 | 学习 C        | FK            | 2017-04-05      |
    +-----------+---------------+---------------+-----------------+
    
    MariaDB [test_db]> select * from tcount_tbl;
    +---------------+--------------+
    | runoob_author | runoob_count |
    +---------------+--------------+
    | 菜鸟教程      |           10 |
    | RUNOOB.COM    |           20 |
    | Google        |           22 |
    +---------------+--------------+

#### INNER JOIN （可以使用INNER,直接使用JOIN）


```
select a.runoob_id,a.runoob_title,a.runoob_author,b.runoob_count from runoob_tbl a inner join tcount_tbl b on a.runoob_author = b.runoob_author
```

    +-----------+---------------+---------------+--------------+
    | runoob_id | runoob_title  | runoob_author | runoob_count |
    +-----------+---------------+---------------+--------------+
    |         1 | 学习 PHP      | 菜鸟教程      |           10 |
    |         2 | 学习 MySQL    | 菜鸟教程      |           10 |
    |         3 | 学习 Java     | RUNOOB.COM    |           20 |
    |         4 | 学习 Python   | RUNOOB.COM    |           20 |
    +-----------+---------------+---------------+--------------+

可以看出查询的记录是满足 on a.runoob_author = b.runoob_author 的交集。

* INNER JOIN 可以用等价的 WHERE子句替换。
等价的SQL：

```
select a.runoob_id,a.runoob_title,a.runoob_author,b.runoob_count from runoob_tbl a , tcount_tbl b where a.runoob_author = b.runoob_author;

```
    
    MariaDB [test_db]> select 		 a.runoob_id,a.runoob_title,a.runoob_author,b.runoob_count from runoob_tbl a , tcount_tbl b where a.runoob_author = b.runoob_author;
    +-----------+---------------+---------------+--------------+
    | runoob_id | runoob_title  | runoob_author | runoob_count |
    +-----------+---------------+---------------+--------------+
    |         1 | 学习 PHP      | 菜鸟教程      |           10 |
    |         2 | 学习 MySQL    | 菜鸟教程      |           10 |
    |         3 | 学习 Java     | RUNOOB.COM    |           20 |
    |         4 | 学习 Python   | RUNOOB.COM    |           20 |
    +-----------+---------------+---------------+--------------+

#### LEFT JOIN

```
MariaDB [test_db]> select a.runoob_id,a.runoob_author,b.runoob_count from runoob_tbl a left join tcount_tbl b on  a.runoob_author = b.runoob_author;
+-----------+---------------+--------------+
| runoob_id | runoob_author | runoob_count |
+-----------+---------------+--------------+
|         1 | 菜鸟教程      |           10 |
|         2 | 菜鸟教程      |           10 |
|         3 | RUNOOB.COM    |           20 |
|         4 | RUNOOB.COM    |           20 |
|         5 | FK            |         NULL |
+-----------+---------------+--------------+
```

####  RIGHT JOIN
    
    MariaDB [test_db]> select a.runoob_id,a.runoob_author,b.runoob_count from runoob_tbl a right join tcount_tbl b on  a.runoob_author = b.runoob_author;
    +-----------+---------------+--------------+
    | runoob_id | runoob_author | runoob_count |
    +-----------+---------------+--------------+
    |         1 | 菜鸟教程      |           10 |
    |         2 | 菜鸟教程      |           10 |
    |         3 | RUNOOB.COM    |           20 |
    |         4 | RUNOOB.COM    |           20 |
    |      NULL | NULL          |           22 |
    +-----------+---------------+--------------+

