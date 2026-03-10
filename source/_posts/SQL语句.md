---
title: SQL语句
date: '2025-11-04 14:06:13'
updated: '2025-11-07 13:33:22'
---
![](/images/9b73fcf80c6e990f8f88fd3ba98c2774.png)

<font style="background-color:rgba(255, 255, 255, 0);">SQL 对大小写不敏感，SELECT 和 select 是一样的</font>

## <font style="background-color:rgba(255, 255, 255, 0);">一些最重要的 SQL 命令</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">SELECT</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 从数据库中提取数据</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">UPDATE</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 更新数据库中的数据</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">DELETE</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 从数据库中删除数据</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">INSERT INTO</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 向数据库中插入新数据</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">CREATE DATABASE</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 创建新数据库</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">ALTER DATABASE</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 修改数据库</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">CREATE TABLE</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 创建新表</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">ALTER TABLE</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 变更（改变）数据库表</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">DROP TABLE</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 删除表</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">CREATE INDEX</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 创建索引（搜索键）</font>
+ **<font style="background-color:rgba(255, 255, 255, 0);">DROP INDEX</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">- 删除索引</font>

<font style="background-color:rgba(255, 255, 255, 0);">以下是一些常用的 SQL 语句和语法：</font>

**<font style="background-color:rgba(255, 255, 255, 0);">SELECT</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于从数据库中查询数据</font>

```sql
SELECT column_name(s)
FROM table_name
WHERE condition
ORDER BY column_name [ASC|DESC]
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">column_name(s)</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要查询的列</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">table_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要查询的表</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">condition</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 查询条件（可选）</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">ORDER BY</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 排序方式，</font>`<font style="background-color:rgba(255, 255, 255, 0);">ASC</font>`<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">表示升序，</font>`<font style="background-color:rgba(255, 255, 255, 0);">DESC</font>`<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">表示降序（可选）</font>

**<font style="background-color:rgba(255, 255, 255, 0);">INSERT INTO</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于向数据库表中插入新数据</font>

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...)
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">table_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要插入数据的表</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">column1, column2, ...</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要插入数据的列</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">value1, value2, ...</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 对应列的值</font>

**<font style="background-color:rgba(255, 255, 255, 0);">UPDATE</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于更新数据库表中的现有数据</font>

```go
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">table_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要更新数据的表</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">column1 = value1, column2 = value2, ...</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要更新的列及其新值</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">condition</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 更新条件</font>

**<font style="background-color:rgba(255, 255, 255, 0);">DELETE</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于从数据库表中删除数据</font>

```plain
DELETE FROM table_name
WHERE condition
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">table_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要删除数据的表</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">condition</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 删除条件</font>

**<font style="background-color:rgba(255, 255, 255, 0);">CREATE TABLE</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于创建新的数据库表</font>

```plain
CREATE TABLE table_name (
    column1 data_type constraint,
    column2 data_type constraint,
    ...
)
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">table_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要创建的表名</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">column1, column2, ...</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 表的列</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">data_type</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 列的数据类型（如</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>`<font style="background-color:rgba(255, 255, 255, 0);">INT</font>`<font style="background-color:rgba(255, 255, 255, 0);">、</font>`<font style="background-color:rgba(255, 255, 255, 0);">VARCHAR</font>`<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">等）</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">constraint</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 列的约束（如</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>`<font style="background-color:rgba(255, 255, 255, 0);">PRIMARY KEY</font>`<font style="background-color:rgba(255, 255, 255, 0);">、</font>`<font style="background-color:rgba(255, 255, 255, 0);">NOT NULL</font>`<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">等）</font>

**<font style="background-color:rgba(255, 255, 255, 0);">ALTER TABLE</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于修改现有数据库表的结构</font>

```plain
ALTER TABLE table_name
ADD column_name data_type
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">table_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要修改的表</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">column_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要添加的列</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">data_type</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 列的数据类型</font>

<font style="background-color:rgba(255, 255, 255, 0);">或：</font>

```plain
ALTER TABLE table_name
DROP COLUMN column_name
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">column_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要删除的列</font>

**<font style="background-color:rgba(255, 255, 255, 0);">DROP TABLE</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于删除数据库表</font>

<font style="background-color:rgba(255, 255, 255, 0);">DROP TABLE table_name</font>

+ `<font style="background-color:rgba(255, 255, 255, 0);">table_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要删除的表</font>

**<font style="background-color:rgba(255, 255, 255, 0);">CREATE INDEX</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于创建索引，以加快查询速度</font>

```plain
CREATE INDEX index_name
ON table_name (column_name)
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">index_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 索引的名称</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">column_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要索引的列</font>

**<font style="background-color:rgba(255, 255, 255, 0);">DROP INDEX</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于删除索引</font>

```plain
DROP INDEX index_name
ON table_name
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">index_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要删除的索引名称</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">table_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 索引所在的表</font>

### <font style="background-color:rgba(255, 255, 255, 0);">WHERE</font>
<font style="background-color:rgba(255, 255, 255, 0);">用于指定筛选条件</font>

```plain
SELECT column_name(s)
FROM table_name
WHERE condition
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">condition</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 筛选条件</font>

<font style="background-color:rgba(255, 255, 255, 0);">下面的运算符可以在 WHERE 子句中使用：</font>

| 运算符 | 描述 |
| --- | --- |
| = | 等于 |
| <> | 不等于**注释：**在 SQL 的一些版本中，该操作符可被写成 != |
| > | 大于 |
| < | 小于 |
| >= | 大于等于 |
| <= | 小于等于 |
| BETWEEN | 在某个范围内 |
| LIKE | 搜索某种模式 |
| IN | 指定针对某个列的多个可能值 |
| OR | 条件或（可以和 AND 一起用，加上（）） |
| AND | 条件和 |
| is null | 空值判断 |


<font style="background-color:rgba(255, 255, 255, 0);">LIKE 模糊查询：</font>

```plain
Select * from emp where ename like 'M%';
```

<font style="background-color:rgba(255, 255, 255, 0);">查询 EMP 表中 Ename 列中有 M 的值，M 为要查询内容中的模糊信息</font>

+ <font style="background-color:rgba(255, 255, 255, 0);"> </font>**<font style="background-color:rgba(255, 255, 255, 0);">%</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">表示多个字值，</font>**<font style="background-color:rgba(255, 255, 255, 0);">_</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">下划线表示一个字符；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);"> </font>**<font style="background-color:rgba(255, 255, 255, 0);">M%</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">: 为能配符，正则表达式，表示的意思为模糊查询信息为 M 开头的</font>
+ <font style="background-color:rgba(255, 255, 255, 0);"> </font>**<font style="background-color:rgba(255, 255, 255, 0);">%M%</font>**<font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">: 表示查询包含M的所有内容</font>
+ <font style="background-color:rgba(255, 255, 255, 0);"> </font>**<font style="background-color:rgba(255, 255, 255, 0);">%M_ </font>**<font style="background-color:rgba(255, 255, 255, 0);">: 表示查询以M在倒数第二位的所有内容</font>

**<font style="background-color:rgba(255, 255, 255, 0);">ORDER BY</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于对结果集进行排序</font>

```plain
SELECT column_name(s)
FROM table_name
ORDER BY column_name [ASC|DESC]
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">column_name</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 用于排序的列</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">ASC</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 升序（默认）</font>
+ `<font style="background-color:rgba(255, 255, 255, 0);">DESC</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 降序</font>

**<font style="background-color:rgba(255, 255, 255, 0);">GROUP BY</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于将结果集按一列或多列进行分组</font>

```plain
SELECT column_name(s), aggregate_function(column_name)
FROM table_name
WHERE condition
GROUP BY column_name(s)
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">aggregate_function</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 聚合函数（如 COUNT、SUM、AVG 等）</font>

**<font style="background-color:rgba(255, 255, 255, 0);">HAVING</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于对分组后的结果集进行筛选</font>

```plain
SELECT column_name(s), aggregate_function(column_name)
FROM table_name
GROUP BY column_name(s)
HAVING condition
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">condition</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 筛选条件</font>

**<font style="background-color:rgba(255, 255, 255, 0);">JOIN</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于将两个或多个表的记录结合起来</font>

```plain
SELECT column_name(s)
FROM table_name1
JOIN table_name2
ON table_name1.column_name = table_name2.column_name
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">JOIN</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 可以是 INNER JOIN、LEFT JOIN、RIGHT JOIN 或 FULL JOIN</font>

**<font style="background-color:rgba(255, 255, 255, 0);">DISTINCT</font>**<font style="background-color:rgba(255, 255, 255, 0);">：用于返回唯一不同的值（除去重复项）</font>

```plain
SELECT DISTINCT columns
FROM table_name
```

+ `<font style="background-color:rgba(255, 255, 255, 0);">columns</font>`<font style="background-color:rgba(255, 255, 255, 0);">: 要查询的列（例如 url,contry,...）</font>

## <font style="background-color:rgba(255, 255, 255, 0);">补充</font>
### UNION
用于合并两个或多个 SELECT 查询结果集

### || 符号
在Oracle / PostgreSQL 中 || 是字符串拼接

在 MySQL 当中 || 是逻辑或操作符

例如：`select 1,2 || flag from ctf`

