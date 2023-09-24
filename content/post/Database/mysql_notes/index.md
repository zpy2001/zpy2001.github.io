---
title: Mysql数据库 - 笔记
description: Notes about Mysql
slug: database-mysql
date: 2022-07-05 00:00:00+0000
categories:
    - Database
tags:
    - Database
    - Mysql
    - CS
weight: 1
---

# sql基础

## 数据类型

![sql](https://www.sqltutorial.org/sql-data-types.aspx)

- tinyint: 8
- smallint: 16
- integer: 32
- bigint: 64

- real: 32
- double/float: 64

精确小数
- decimal(size, d)/numeric(size, d): size位10进制数，小数点右侧有d位小数

字符串使用单引号

null参与的算术运算结果为null，比较操作结果为unknown

### 转换

```sql
CAST(expression AS datatype(length))
CONVERT(data_type(length), expression[, style])
```

### 字符串拼接

+
concat


## 运算符

运算符|作用
|:-:|:-:|
`=`|等于
`<=>`|安全的等于
`<>`或者`!=`|不等于
`<=`|小于等于
`>=`|大于等于
`>`|大于
IS NULL 或者 ISNULL	判断一个值是否为空
IS NOT NULL	判断一个值是否不为空
BETWEEN AND	判断一个值是否落在两个值之间

## 建

```sql
CREATE TABLE [IF NOT EXISTS] table_name(
   column_1_definition,
   column_2_definition,
   ...,
   table_constraints
) ENGINE=storage_engine;
```

column_defination:

```sql
column_name data_type(length) [NOT NULL] [DEFAULT value] [AUTO_INCREMENT] column_constraint;
```

table_constraints:

候选键UNIQUE, CHECK, FOREIGN KEY

```sql
PRIMARY KEY (col1,col2,...)
FOREIGN KEY (col1,col2,...)
    REFERENCES table (col1,col2,...)
    ON UPDATE RESTRICT
    ON DELETE CASCADE
    SET NULL
```

弱实体依赖主实体而存在，主实体被删则弱实体也删

建立/删除索引：

```sql
create index index_name on table_name(column_list);
-- 不允许在表列中插入任何重复值
create unique index index_name on table_name(column_list);


drop index index_name on table_name;
-- mysql --
alter table table_name
drop index index_name;
```



## 增

```sql
INSERT INTO table(c1,c2,...)
VALUES 
   (v11,v12,...),
   (v21,v22,...),
    ...
   (vnn,vn2,...);

INSERT INTO table_name(column_list)
SELECT 
   select_list 
FROM 
   another_table
WHERE
   condition;

insert into table_name(column_list)
values(
    (select xx from xxx),
    (select xx from xxx)
);

INSERT INTO table (column_list)
VALUES (value_list)
ON DUPLICATE KEY UPDATE
   c1 = v1, 
   c2 = v2,
   ...;
# sql将判断
# 若待插入的行已经存在，则更新之，若更新的值不同于原先返回2 row(s) affected
#                           否则返回0 row(s) affected
# 否则插入之，返回1 row(s) affected
```

## 删

```sql
DELETE FROM table_name
WHERE condition;

DELETE T1, T2
FROM T1
INNER JOIN T2 ON T1.key = T2.key
WHERE condition;

DELETE T1 
FROM T1
        LEFT JOIN
    T2 ON T1.key = T2.key 
WHERE
    T2.key IS NULL; # ? condition or example;

DROP TABLE table_name;
```

## 改

```sql
UPDATE [LOW_PRIORITY] [IGNORE] table_name 
SET 
    column_name1 = expr1,
    column_name2 = expr2,
    ...
[WHERE
    condition];

UPDATE T1, T2,
[INNER JOIN | LEFT JOIN] T1 ON T1.C1 = T2. C1
SET T1.C2 = T2.C2, 
    T2.C3 = expr
WHERE condition

# 下面与inner join相同
UPDATE T1, T2
SET T1.c2 = T2.c2,
      T2.c3 = expr
WHERE T1.c1 = T2.c1 AND condition
```

## 查

### order by

```sql
SELECT 
   select_list
FROM 
   table_name
ORDER BY 
   column1 [ASC|DESC], 
   column2 [ASC|DESC],
   ...;
```

### group by

对结果按照(结果的)列分组，若相关列相同，则只返回一条结果，但若在前面count(*)仍然返回所有满足调节行数

#### having

```sql
SELECT 
    select_list
FROM 
    table_name
WHERE 
    search_condition
GROUP BY 
    group_by_expression
HAVING 
    group_condition;
```

```sql
SELECT 
    c1, c2,..., cn, aggregate_function(ci)
FROM
    table
WHERE
    where_conditions
GROUP BY c1 , c2,...,cn;
```

### union

将多个查询结果集合在同一列，其中：

- select的列数量和名称以及数据类型必须一致
- 默认和`DISTINCT`选项删除重复数值的行，`ALL`选项保留所有行

```sql
SELECT column_list
UNION [DISTINCT | ALL]
SELECT column_list
UNION [DISTINCT | ALL]
SELECT column_list
```

### intersect

```sql
(SELECT column_list 
FROM table_1)
INTERSECT
(SELECT column_list
FROM table_2);
```

### except

没有直接的except

需要用not in替代

### limit

```sql
select * from xx limit 30 offset 15;
select * from xx limit 30, 15;
```

显示前n条记录

#### offset

从第几条结果开始

### any/all

条件满足存在/任意

## 其他

### count

`count()`计算查询结果行数

```sql
declare variable int default 0;
select count(*) into variable from tableName where condition
```

### sum

```sql
SELECT SUM(column_name) FROM table_name WHERE condition
```

### join

join: 两个表中有公共列，使用join将两个表中查询得到的结果组合

#### insert join

or join

```sql
SELECT column_list
FROM table_1
INNER JOIN table_2 ON join_condition;
```

- 基于连接谓词的条件链接两个表
- 将第一个表中每一行与第二个表中每一行进行比较
- 两行的值都满足条件，则创建一个新行，列包含两个表中两行的所有列

例：

```sql
SELECT 
    m.member_id, 
    m.name AS member, 
    c.committee_id, 
    c.name AS committee
FROM
    members m
INNER JOIN committees c ON c.name = m.name;
```

- 如果连接条件使用相等运算符，并且用于匹配的两个表中的列名相同，则可以改用该子句：`USING()`:

```sql
SELECT column_list
FROM table_1
INNER JOIN table_2 USING (column_name);
```

#### left join

or left outer join

左连接将左表的每一行与右表每一行比较

- 若两行值不匹配，创建新行，包含两个表该行所有列
- 若两行中的值不匹配，则创建新行，包含左表和右表的列但右表对应行值为NULL

结果会包含左表中的所有行，和右表中满足条件的行，其余为NULL

若要查找右表中没有对应行的行，还可以使用带有运算符的子句：`WHERE IS NULL`，结果只显示右表值为`NULL`的行

```sql
SELECT column_list 
FROM table_1 
LEFT JOIN table_2 ON join_condition;

SELECT column_list 
FROM table_1 
LEFT JOIN table_2 USING (column_name);

SELECT column_list 
FROM table_1 
LEFT JOIN table_2 USING (column_name);
WHERE column_table_2 IS NULL
```

#### right join

or right outer join

#### cross join

无连接条件，对连接表的行笛卡尔积

```sql
SELECT select_list
FROM table_1
CROSS JOIN table_2;
```

#### outer join

or full outer join
or full join

行数等于两张表行数和

### 锁

#### 锁粒度

行锁，页锁，表锁

#### 数据库管理

- 共享锁/读锁：`select * from tableName where … lock in share mode`
    - 可以并发读但不可写
- 排他锁/写锁：`select * from tableName where … for update`
- 意向锁：表级锁，InnoDB自动

#### 程序员

##### 乐观锁

乐观锁（Optimistic Locking）认为对同一数据的并发操作不会总发生，属于小概率事件，不用每次都对数据上锁，也就是不采用数据库自身的锁机制，而是通过程序来实现。

- 乐观锁的版本号机制

在表中设计一个版本字段 version，第一次读的时候，会获取 version 字段的取值。然后对数据进行更新或删除操作时，会执行

```sql
UPDATE ... SET version=version+1 WHERE version=version
```

此时如果已经有事务对这条数据进行了更改，修改就不会成功。

- 乐观锁的时间戳机制

时间戳和版本号机制一样，也是在更新提交的时候，将当前数据的时间戳和更新之前取得的时间戳进行比较，如果两者一致则更新成功，否则就是版本冲突。

##### 悲观锁
悲观锁（Pessimistic Locking）也是一种思想，对数据被其他事务的修改持保守态度，会通过数据库自身的锁机制来实现，从而保证数据操作的排它性。

适用场景

- 乐观锁适合读操作多的场景，相对来说写的操作比较少。它的优点在于程序实现，不存在死锁问题，不过适用场景也会相对乐观，因为它阻止不了除了程序以外的数据库操作。
- 悲观锁适合写操作多的场景，因为写的操作具有排它性。采用悲观锁的方式，可以在数据库层面阻止其他事务对该数据的操作权限，防止读 - 写和写 - 写的冲突

避免死锁的发生：

- 如果事务涉及多个表，操作比较复杂，那么可以尽量一次锁定所有的资源，而不是逐步来获取，这样可以减少死锁发生的概率；
- 如果事务需要更新数据表中的大部分数据，数据表又比较大，这时可以采用锁升级的方式，比如将行级锁升级为表级锁，从而减少死锁产生的概率；
- 不同事务并发读写多张数据表，可以约定访问表的顺序，采用相同的顺序降低死锁发生的概率

### Like

```sql
expression LIKE pattern ESCAPE escape_character
```

%匹配0个或多个字符
_匹配单个字符

### in

```sql
[not ]in (value1, value2);
```

### ifnull

```sql
IFNULL(expression, alt_value)
```

### as

可以重命名表和列

## notice

avg, max这类函数不能在where语句直接使用

要用select子句替代

# sql存储过程

## 基本语法

```sql
delimiter $$

create procedure xx(
    in name type(size),
    inout name type(size),
    out name type(size),
    out othername type(size) # 可返回多个值
)
begin
    # blabla
    -- blabla
    /*
     * blabla
     */
end$$
delimiter ;
```

变量通过`out`返回，`select`直接输出结果

调用存储过程：

```sql
set @inout_param = xxx;
call xx(param_name, @inout_param, @return_name, @other_return_name);
```

### 变量

声明变量：

```sql
declare variable_name_declare datatype(size) [default default_value];   # 局部变量，只在作用域内生效
set variable_name_declare = value;  # 更新declare声明的变量
set @variable_name_set = value; # 类似全局变量，可在作用域外赋值，赋值语句语法相同
# value为表达式
```

### 简单语句

- if:

```sql
if condition then
    statements;
elseif condition then
    statements;
else
    statements;
end if;
```

- case:

```sql
case case_val:
    when val1 then statements
    when val2 then statements
    [else statements]
end case;
```

- loop:

```sql
[begin_label:] loop
    statements;
end loop [end_label]
```

- while:

```sql
[begin_label:] while condition do
    statements;
end while [end_label]
```

- repeat:

```sql
[begin_label:] repeat
    statements;
until condition
end repeat [end_label]
```

- leave:

跳出label存储过程

```sql
leave label;
```

### 错误处理

```sql
declare action handler for condition_val statement;
```

条件匹配将继续或退出代码块

`action`有：
- `continue`
- `exit`

## 删除存储过程

```sql
DROP PROCEDURE [IF EXISTS] stored_procedure_name;
```

# Triggers

为响应关联表中发生的插入、更新或删除等事件自动调用的存储程序

两种类型触发器：行级触发器和语句级触发器

## 创建

```sql
CREATE TRIGGER trigger_name
{BEFORE | AFTER} {INSERT | UPDATE| DELETE }
ON table_name FOR EACH ROW
trigger_body;
```

若要表示原先数据则用`old`和`new`指代触法行的数据

## 显示/删除

```sql
show triggers
DROP TRIGGER [IF EXISTS] [schema_name.]trigger_name;

SHOW TRIGGERS
[{FROM | IN} database_name]
[LIKE 'pattern' | WHERE search_condition];

```

## 多触发器

该触发器将在现有触发器之前或之后激活，以响应相同的事件和操作时间:

```sql
DELIMITER $$

CREATE TRIGGER trigger_name
{BEFORE|AFTER}{INSERT|UPDATE|DELETE} 
ON table_name FOR EACH ROW 
{FOLLOWS|PRECEDES} existing_trigger_name
BEGIN
    -- statements
END$$

DELIMITER ;
```

# 视图

将查询保存在数据库服务器并指定名称为视图

使用时可看作表，执行时进行调用

```sql
CREATE [OR REPLACE] VIEW [db_name.]view_name [(column_list)]
AS
  select-statement;

# 视图处理算法
CREATE [OR REPLACE][ALGORITHM = {MERGE | TEMPTABLE | UNDEFINED}] VIEW 
   view_name[(column_list)]
AS 
   select-statement;
```

不可更新视图,有五种:
(1)select目标列包含聚集函数.
(2)select字句使用了unique或distinct
(3)视图中包括group by字句
(4)视图中包括经算法表达式计算出来的列
(5)由单个表的列构成,但并不包括主键的视图

## 视图处理算法

### Merge

类似inline，将调用sql语句替换成视图语句

### Temptable

将创建一个临时表存储视图语句结果

### Undefined

倾向于Merge

## 更改现有视图

```sql
ALTER
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    VIEW view_name [(column_list)]
    AS select_statement;
```

## 更新视图

```sql
update view set xxx = xx where condition;
```

## 显示视图

```sql
SHOW FULL TABLES 
[{FROM | IN } database_name/sys]
WHERE table_type = 'VIEW';
```

结果：显示视图名称|表类型

显示视图内容：

```sql
show create view view_name;
```

## 重命名视图

```sql
rename table xx to xxx;
```

## 删除视图

```sql
drop view [if exist] view_name_1, view_name_2;
```

## 检查视图

### 一致性

视图的一致性：用户户只能显示或更新通过视图可见的数据.

```sql
CREATE [OR REPLACE VIEW] view_name 
AS
  select_statement
  WITH CHECK OPTION;
```

### 本地检查

```sql
CREATE/ALTER VIEW v2 AS
    SELECT 
        c
    FROM
        v1 
WITH LOCAL CHECK OPTION;
```

### 级联检查

```sql
CREATE OR REPLACE VIEW v2 
AS
    SELECT c FROM v1 
WITH CASCADED CHECK OPTION;
```

# sql事件

sql使用称为事件调度程序线程的特殊线程来执行所有调度事件

启用事件调度程序

```sql
set global event_scheduler = on;
show processlist;
```

## 创建事件

```sql
CREATE EVENT [IF NOT EXIST] event_name
ON SCHEDULE schedule
DO
event_body
```

schedule有以下：

- 如果事件是一次性事件，请使用以下语法：

```sql
AT timestamp [+ INTERVAL]
```

- 如果事件是定期事件，请使用子句：

```sql
EVERY interval STARTS timestamp [+INTERVAL] ENDS timestamp [+INTERVAL]
```

## 删除事件

```sql
DROP EVENT [IF EXIST] event_name;
```

## 修改事件

```sql
ALTER EVENT event_name
ON SCHEDULE schedule
ON COMPLETION [NOT] PRESERVE
RENAME TO new_event_name
ENABLE | DISABLE
DO
  event_body
```

启用/禁用事件：

```sql
ALTER EVENT event_name
ENABLE | DISABLE;
```

重命名事件：

```sql
ALTER EVENT event_name
RENAME TO new_event_name;
```

## 访问权限

```sql
grant priviledges on object to user [WITH GRANT OPTION];
revoke [grant option for] priviledges on object from user [cascade|restrict];
```

priviledges:

- select
- insert
- delete
- index
- create
- alter
- drop
- all
- update
- grant

## check

```sql
check(condition)
[CONSTRAINT [constraint_name]] CHECK (expression) [[NOT] ENFORCED]
```

单个属性（列）之后，或独立声明

```sql
create table xx(
    col_name type check(condition)
    col_name type not null check(condition)
    check(condition [by sql])
);
```

delete不会检查，若为平均分之类的条件则发生错误

### defer

```sql
set constraint defered;
```

## Assertion(mysql无)

```sql
create assertion assertname check(condition);
```

代价更大
