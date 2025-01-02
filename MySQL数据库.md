﻿﻿﻿# MySQL数据库
## 函数
1. 字符串函数
	- concat(s1, s2, s3)：字符串拼接
	- lower(str)：字符串全部转为小写
	- upper(str)：字符串全部转为大写
	- lpad(str, n, pad)：字符串向左填充直至长度为n
	- rpad(str, n, pad)
	- trim(str)
	- substring(str, start, len)：**索引从1开始**。
2. 数值函数
	- ceil(x)：向上取整
	- floor(x)
	- mod(x, y)
	- rand()：返回0~1之间的随机数
	- round(x, y)：对x四舍五入，保留y位小数。
3. 日期函数
	- curdate()：返回当前日期
	- curtime()：当前时间
	- now()：当前的日期和时间
	- year(date)：返回年
	- month(date)：返回月
	- day(date)：返回日
	- date_add(date, **interval** expr type)：例如`select date_add(now(), interval 70 day);`
	- datediff(date1, date2)：返回日期的间隔天数，date1 - date2
4. 流程控制函数
	- if(value, t, f)：如果value为真，返回t，否则返回f。
	- ifnull(value1, value2)：如果value1不为null返回value1， 否则返回value2
	- case when 变量名 then 值1 else 值2 end：when...then组合可以有多个
##  约束
1. 主键约束 primary key
2. 非空约束 not null
3. 唯一约束 unique
4. 检查约束 check：用户自定义约束。例如，`check(age >= 0 and age <= 120)`
5. 默认约束 default
6. 外键约束 foreign key
	更新、删除行为：
		- cascade 级联：修改父表主键时同时会改变子表的外键
		- set null：删除父表记录时会将子表的外键变为null
##  多表查询
1. 内连接
2. 外连接
3. 自连接
4. 联合查询 union ：将多次查询的结果合并，形成一个新的查询结果。
	- union all：直接合并，不去重
	- union：合并并去重
5. 子查询
	- 行子查询：例如，`(字段1， 字段2） = 子查询结果`
## 事务
### 事务操作
1. 开启事务：start transaction / begin
2. 提交 commit
3. 回滚 rollback
#### 并发事务问题
1. 脏读：读到其他事务还没有提交的数据，该事务进行回滚操作
2. 不可重复读：前后两次读取的数据不一致
3. 幻读：一个事务内执行两次相同的查询时，第二次查询可能会看到在第一次查询后被其他事务插入的新记录。
#### 事务的隔离级别
| 隔离级别      | 脏读 |	不可重复读|	幻读|
| ----------- | ----------- | ---------| ------|
| Read uncommitted | 不能解决      |不能解决| 不能解决|
| Read committed      | 解决      |不能解决| 不能解决|
|  Repeatable Read（默认） | 解决      |解决| 不能解决|
| Serializable | 解决      |解决| 解决|

## SQL 优化

### SQL 执行频率

` SHOW  GLOBAL STATUS LIKE  'Com_______';`命令可以查看增删改查语句执行的频率

### 慢查询日志

1. 查看慢查询日志是否开启：

```
show variables like 'slow_query_log';
```

2. 开启慢查询日志，修改 `/etc/my.cnf `文件

```properties
# 开启MySQL慢日志查询开关 
slow_query_log=1 
# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志 l
ong_query_time=2
```

3. 查看慢查询日志， `my.cnf` 配置文件里面有。

   ` tail -f 目录/mysql-slow.log`

### profile

1. 查看是否支持 profile：`SELECT  @@have_profiling `
2. 开启 profile，`set  profiling = 1`

3. 查看语句耗时情况：

```sql
-- 查看每一条SQL的耗时基本情况
show profiles;
-- 查看指定query_id的SQL语句各个阶段的耗时情况
show profile  for  query query_id;
-- 查看指定query_id的SQL语句CPU的使用情况
show profile  cpu for  query query_id
```

### explain

explain 可以展示查询语句的执行计划

#### 字段含义

| **字段**        | **含义**                                                     |
| --------------- | :----------------------------------------------------------- |
| `id`            | 查询标识符，用于区分查询的不同部分（主查询或子查询等）。     |
| `select_type`   | 查询类型，表示查询的结构。常见类型包括 `SIMPLE`、`PRIMARY`、`UNION`、`SUBQUERY` 等。 |
| `table`         | 显示查询中使用的表的名称。                                   |
| `type`          | 连接类型，表示 MySQL 如何访问表。不同的连接类型会对查询性能产生重大影响。 |
| `possible_keys` | 显示 MySQL 可以使用的索引列表。如果没有合适的索引，会返回 `NULL`。 |
| `key`           | 显示实际选择的索引。                                         |
| `key_len`       | 显示 MySQL 在使用索引时，索引的长度（以字节为单位）。        |
| `ref`           | 显示用于与表中的列进行比较的索引列或常量。                   |
| `rows`          | 显示 MySQL 估算扫描的行数，帮助评估查询的复杂度。            |
| `filtered`      | 显示 MySQL 估算满足查询条件的行百分比。                      |
| `Extra`         | 显示执行的额外信息，常见的值包括 `Using index`、`Using where`、`Using temporary`、`Using filesort`。 |

#### 重点关注的字段
1. **`type`**: （效率由低到高）
- `ALL`：全表扫描，效率最低。
   - index`：全索引扫描。
- range`：基于索引的范围扫描。
   - `ref`：基于索引的查找，效率较高
- `eq_ref`：每个行通过索引唯一查找，通常用于主键或唯一索引
   - `const`：对于常量查询，最多返回一行数据，效率非常高。
- system`：当表只有一行数据时，效率最高。

2. **`key`**: 确保查询使用了合适的索引。

3. **`rows`**: 估算扫描的行数，行数越大，查询越复杂。

4. **`Extra`**: 特别关注 `Using temporary` 和 `Using filesort`，这些可能表示查询存在性能问题。



### 索引失效

#### 常见索引失效 sql 语句

1. 尽量多使用 `>=`而不是`>`
2. 索引不要使用列运算
3. 字符串要加引号
4. 模糊查询如果在头部，索引会失效

#### 最左前缀法则

针对联合索引，查询要从最左边的索引开始，不能跳过中间的索引，否则后面的索引会全部失效。

#### 覆盖索引

查询投影的字段包含在索引的叶节点上，不用回表查询。



### insert 语句

尽量选择顺序、批量插入



### order by 语句

order by 后面的字段如果建立了索引，就不需要额外排序



### limit 语句

`select  *  from  tb_sku  t  ,  (select  id  from  tb_sku  order  by  id  limit  2000000,10)  a  where t.id  =  a.id`



### update 语句

where 后面的条件最好建立了索引，避免升级为表锁

## 锁

### 全局锁

主要在数据库备份的时候使用，期间数据库处于只读状态。

步骤：

1. 加全局锁：`flush tables with read lock ;`

2. 数据备份：`mysqldump  -uroot –p1234  itcast > itcast.sql`

3. 释放锁：`unlock tables ;`

> 如果不想加锁，可以使用 `mysqldump  --single-transaction  -uroot –p123456  itcast > itcast.sql`

### 表级锁

### 行级锁



## MVCC

全称 Multi-Version Concurrency Control，多版本并发控制。指维护一个数据的多个版本， **使得读写操作没有冲突**，快照读为MySQL实现MVCC提供了一个非阻塞读功能。MVCC的具体实现，还需 要依赖于数据库记录中的三个隐式字段、undo log日志、readView。
