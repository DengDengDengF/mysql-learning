[TOC]

### 1.登录

```java
mysql -u root -p

mysql -h localhost -P 3306 -u root -p
```

### 2.引入脚本

```
NAVICAT: 文件--->打开外部文件--->查询
Intellij Idea: File--->open----->sql文件
```

### 3.数据类型

```sql
名称	           类型	    说明
INT	            整型	   4字节整数类型，范围约+/-21亿
BIGINT	       长整型	  8字节整数类型，范围约+/-922亿亿
REAL	       浮点型    4字节浮点数，范围约+/-1038
DOUBLE	       浮点型	  8字节浮点数，范围约+/-10308
DECIMAL(M,N) 高精度小数	 由用户指定精度的小数，例如，DECIMAL(20,10)表示一共20位，其中小数10位，通常用于财务计算
CHAR(N)	     定长字符串	 存储指定长度的字符串，例如，CHAR(100)总是存储100个字符的字符串
VARCHAR(N)	 变长字符串	 存储可变长度的字符串，例如，VARCHAR(100)可以存储0~100个字符的字符串
BOOLEAN	      布尔类型	  存储True或者False
DATE	      日期类型	  存储日期，例如，2018-06-22
TIME	      时间类型	  存储时间，例如，12:20:59
DATETIME	日期和时间类型	存储日期+时间，例如，2018-06-22 12:20:59
```

### 4.关系模型

#### 4.1主键

```sql
主键最好是完全业务无关的字段

-- 查看主键
SHOW KEYS FROM students WHERE Key_name = 'PRIMARY';
```

#### 4.2外键

```sql
-- 定义外键约束
ALTER TABLE students
ADD CONSTRAINT fk_class_id
FOREIGN KEY (class_id)
REFERENCES classes (id);
-- students表中的class_id，必须在classes的id里能找到，否则报错

-- 查看详细约束结构信息
SHOW CREATE TABLE students;

-- 查看索引
SHOW INDEX FROM students;

-- 删除外键约束
ALTER TABLE students DROP FOREIGN KEY fk_class_id;
```

#### 4.3索引

```sql
-- 查看详细约束结构信息
SHOW CREATE TABLE students;

-- 查看索引
SHOW INDEX FROM students;

-- 创建索引，可以多列索引
ALTER TABLE students ADD INDEX idx_name_score (name, score);

-- 列，唯一索引
ALTER TABLE students ADD UNIQUE INDEX uni_name (name);

-- 列，唯一约束，不添加唯一索引
ALTER TABLE students ADD CONSTRAINT uni_name UNIQUE (name);

-- 强制使用索引
SELECT * FROM students FORCE INDEX (idx_class_id) WHERE class_id = 1 ORDER BY id DESC;
```

### 4.显示、创建、删除、使用、退出数据库

```sql
显示
SHOW DATABASES;
创建
CREATE DATABASE test;
使用
USE `test`;
删除
DROP DATABASE test;
退出
exit;
```



### 5显示所有表、显示表结构、查看创建表的sql语句，创建表、删除表、新增列、修改列名、删除列。

```sql
-- 显示所有表
SHOW TABLES;

--显示表结构
DESC students;

-- 查看创建表的sql语句
SHOW CREATE TABLE students;

-- 创建表
USE `testdb`;
CREATE TABLE students(
id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
class_id BIGINT NOT NULL,
name VARCHAR(255) NOT NULL,
gender CHAR(1) NOT NULL,
score TINYINT NOT NULL
);

-- 删除表
DROP TABLE students;

-- 新增列
ALTER TABLE students ADD COLUMN birth VARCHAR(10) NOT NULL;

-- 修改列名
ALTER TABLE students CHANGE COLUMN birth birthday VARCHAR(20) NOT NULL;

-- 删除列
ALTER TABLE students DROP COLUMN birthday;
```

### 6.查询

####       6.1基本查询

```sql
SELECT * FROM <表名>
查询一个表的所有行和列数据，结果是个二维表
```

#### 6.2条件查询

```sql
SELECT * FROM <表名> WHERE <条件表达式>

SELECT * FROM <表名> WHERE <条件表达式> AND  <条件表达式>

SELECT * FROM <表名> WHERE <条件表达式> OR  <条件表达式>

SELECT * FROM <表名> WHERE  NOT <条件表达式>

条件运算符优先级 NOT、AND、OR

tips:
WHERE score BETWEEN 60 AND 90 (左闭右闭)
WHERE gender <> 'M'(不等于)
WHERE name LIKE 'ab%'(%表示任意字符，例如'ab%'将匹配 'ab','abc','abcd')
```

#### 6.3投影查询

```sql
以上的都是全部查询到的都是全部列
如果，需要返回某些列：
SELECT <列1>,<列2>,<列3>  FROM <表名>
SELECT <列1> 别名1,<列2> 别名2,<列3>别名3 FROM  表 WHERE <条件表达式>
```

#### 6.4排序

```sql
升序：
SELECT * FROM  <表名> ORDER BY <列名> 
SELECT * FROM  <表名> WHERE <条件表达式> ORDER BY <列名> ASC
//先按照<my>升序，<my>相同，按照<fucker>升序
SELECT * FROM  <表名> WHERE <条件表达式> ORDER BY <my> ASC,<fucker>


降序:
SELECT * FROM  <表名> ORDER BY <列名>  DESC
SELECT * FROM  <表名> WHERE <条件表达式> ORDER BY <列名> DESC
//先按照<my>降序，<my>相同，按照<fucker>降序
SELECT * FROM  <表名> WHERE <条件表达式> ORDER BY <my> DESC,<fucker>
```

#### 6.5分页查询

```sql
数据库总共 sum 条数据，一页显示  M  条数据
那么，可以分成 pageSum = Math.ceil(sum / M) 页，
let currentPage = 0, N = 0;
while(currentPage < pageSum){
    N = currnetPage * M;
    SELECT * FROM  <表名> ORDER BY <列名> LIMIT M OFFSET N;
    currentPage++;
}

tips:
OFFSET是可选的，如果只写LIMIT 15，那么相当于LIMIT 15 OFFSET 0。
在MySQL中，LIMIT 15 OFFSET 30还可以简写成LIMIT 30, 15。
使用LIMIT <M> OFFSET <N>分页时，随着N越来越大，查询效率也会越来越低。
这个N它不是键，如果初始N很大，就得从b+数叶子节点从0开始数到N， 后续就快了。

```

#### 6.6聚合查询

1.**COUNT**

```sql
查询表的记录数
SELECT COUNT(*) <别名> FROM <表名>;
加入条件后
SELECT COUNT(*) <别名> FROM <表名> WHERE <条件表达式>;
```

2.**AVG**

```sql
SELECT AVG(<列名>) <别名> FROM <表名> WHERE <条件表达式>;
```

3.**MAX**

```sql
SELECT MAX(<列名>) FROM students;
加入条件后同理。
```

4.**MIN**

```sql
SELECT MIN(<列名>) FROM students;
加入条件后同理。
```

5.**分组**

```sql
SELECT COUNT(*) <别名> FROM <表名> GROUP BY <列名1>;
会把 会把<列名1>相同的列先分组，再分别计算。
但是结果不好看出来，因此，把<列名1>列也放入结果集中。


SELECT <列名1>,<列名2>,COUNT(*) <别名> FROM <表名> GROUP BY <列名1>,<列名2>;
其他类比。
```

#### 6.7 多表查询

```sql
多表查询结果是个笛卡尔积，获取M × N 行记录，小心使用！
SELECT * FROM <表1>,<表2>
加入<表1> <表2>都有个列名叫 id, 那么会重名，不方便观察
因此，
SELECT
    <别名1>.<别名1的列> <别名1的别名>,
    .....
    <别名2>.<别名2的列> <别名2的别名>,
  
FROM <表1> <别名1>,<表2> <别名2>

当然，也可以添加  WHERE <条件表达式>
```

#### 6.8 连接查询

1.**INNER JOIN**

![png](https://liaoxuefeng.com/books/sql/query/join/inner-join.jpg)

```sql
内连接得到的是交集。
SELECT
   <别名1>.xxxxx,
   ....
   <别名2>.xxxx,
   ....
FROM
  <表1> <别名1>
  INNER JOIN <表2> <别名2> ON  <别名1>.xx=<别名2>.xx

可选加上 WHERE、ORDER BY 等子句
就是先找出指定数据后再加条件、排序等
```

2.**RIGHT OUTER JOIN**

![png](https://liaoxuefeng.com/books/sql/query/join/right-outer-join.jpg)

```sql
右外连接得到的是，右表都存在的行返回"右表都存在"的行。如果某一行仅在右表存在，那么结果集就会以"NULL"填充剩下的字段。
SELECT
   <别名1>.xxxxx,
   ....
   <别名2>.xxxx,
   ....
FROM
  <表1> <别名1>
 RIGHT OUTER JOIN <表2> <别名2> ON  <别名1>.xx=<别名2>.xx
 
```

3.**LEFT OUTER JOIN**

![PNG](https://liaoxuefeng.com/books/sql/query/join/left-outer-join.jpg)

```sql
左外连接得到的是，右表都存在的行返回"左表都存在"的行。如果某一行仅在左表存在，那么结果集就会以"NULL"填充剩下的字段。
SELECT
   <别名1>.xxxxx,
   ....
   <别名2>.xxxx,
   ....
FROM
  <表1> <别名1>
 RIGHT OUTER JOIN <表2> <别名2> ON  <别名1>.xx=<别名2>.xx
```

4.**LEFT OUTER JOIN  UNION  RIGHT OUTER JOIN**

![PNG](https://liaoxuefeng.com/books/sql/query/join/full-outer-join.jpg)

```sql
并集。
SELECT
    ...
FROM
  <表1> <别名1>
 RIGHT OUTER JOIN <表2> <别名2> ON  <别名1>.xx=<别名2>.xx
 
UNION  

SELECT
    ....
FROM
  <表1> <别名1>
 RIGHT OUTER JOIN <表2> <别名2> ON  <别名1>.xx=<别名2>.xx
```

### 7.修改数据

#### 7.1插入数据

```sql
INSERT INTO <表名> (字段1, 字段2, ...) VALUES (值1, 值2, ...),(值1, 值2, ...),(值1, 值2, ...),...;
一次性添加多条数据。
```

#### 7.2更新数据

```sql
UPDATE <表名> SET 字段1=值1, 字段2=值2, ... WHERE <条件表达式>
如果没有WHERE <条件表达式>，就会'修改整个表'！
```

#### 7.3删除数据

```sql
DELETE FROM <表名> WHERE  <条件表达式>;
如果没有WHERE <条件表达式>，就会'删除整个表'！
```

#### 7.4插入或替换 ，插入或更新 ，插入或忽略

```sql
-- 插入或替换
REPLACE INTO <表> (<列1>,<列2>...) VALUES (<value1>,<value2>...);
若<列1>=<value1>的记录不存在，REPLACE语句将插入新记录，否则，当前<列1>=<value1>的记录将被删除，然后再插入新记录。

-- 插入或更新
INSERT INTO <表> (<列1>,<列2>...) VALUES (<value1>,<value2>...) ON DUPLICATE KEY UPDATE <列1>=<value1>...;
若<列1>=<value1>的记录不存在，INSERT语句将插入新记录，否则，当前<列1>=<value1>的记录将被更新，更新的字段由UPDATE指定。


-- 插入或忽略
INSERT IGNORE INTO  <表> (<列1>,<列2>...) VALUES (<value1>,<value2>...);
若<列1>=<value1>的记录不存在，INSERT语句将插入新记录，否则，不执行任何操作。
```

#### 7.5 快照

```sql
-- 对class_id=1的记录进行快照，并存储为新表students_of_class1:
CREATE TABLE students_of_class1 SELECT * FROM students WHERE class_id=1;
```

#### 7.6写入查询结果集

```sql
-- 创建一个新表
CREATE TABLE statistics (
    id BIGINT NOT NULL AUTO_INCREMENT,
    class_id BIGINT NOT NULL,
    average DOUBLE NOT NULL,
    PRIMARY KEY (id)
);

-- 从指定表中写入到 新表
INSERT INTO statistics (class_id, average) SELECT class_id, AVG(score) FROM students GROUP BY class_id;
```

### 8.事务

```sql
把多条语句作为一个整体进行操作的功能，被称为`数据库事务`。
原子、一致、隔离、持久。
```

![image.png](https://s2.loli.net/2024/08/01/O7cPsLIABTnGMlu.png)

#### 8.1Read Uncommitted

```sql
一个事务会"读到"另一个事务"更新后但未提交"的数据，如果另一个事务回滚，那么当前事务读到的数据就是脏数据，这就是脏读（Dirty Read）。
```

#### 8.2Read Committed

```sql
一个事务"不会读到"另一个事务还"没有提交"的数据。一个事务内，多次读同一数据，在这个事务还没有结束时，如果另"一个事务恰好修改并提交"了这个数据，那么，在第一个事务中，两次读取的数据就可能不一致,这就是"不可重复读"。
```

#### 8.3Repeatable Read

```sql
"幻读"是指，在一个事务中，第一次查询某条记录，发现没有，但是，当试图更新这条不存在的记录时，竟然能成功，并且，再次读取同一条记录，它就神奇地出现了。"可以重复读".
```

#### 8.4Serializablel

```sql
Serializable是最严格的隔离级别。在Serializable隔离级别下，所有事务按照次序依次执行，因此，脏读、不可重复读、幻读都不会出现。
```



### 
