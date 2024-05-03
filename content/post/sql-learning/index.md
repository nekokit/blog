---
title: SQL 基础
slug: sql-learning
description: 记录 sql 学习之路。
image: cover.webp
date: 2022-10-26T10:09:01+08:00
categories: [Technology]
tags: [sql, 数据库]
---

## 简介

本文主要记录学习 SQL 的过程以及基础 SQL 语法。

## DDL - Date Definition Language

数据库定义语言用于数据库与表的创建，用于组织数据结构与逻辑。

### 数据库操作

#### 创建数据库

```sql
CREATE DATABASE IF NOT EXISTS `sql_study`
    DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;

-- 语句结构
CREATE DATABASE [IF NOT EXISTS] `${数据库名}`
    DEFAULT CHARACTER SET ${字符集} COLLATE ${排序};
```

{{< notice tip >}}

在使用工具创建 SQL 时可能会出现如 `/*!40100 */` 的特殊注释，这里使用 DataGrip 连接 MySQL 创建，代表服务器 MySQL 版本大于 4.0.1 时执行。

{{< /notice >}}

{{< notice info >}}

创建数据库时最好加上 `IF NOT EXISTS` 条件，避免数据库已存在导致报错

{{< /notice >}}

#### 删除数据库

```sql
DROP DATABASE IF EXISTS `sql_study`;

-- 语句结构
DROP DATABASE [IF EXISTS] `${数据库名}`;
```

{{< notice info >}}

删除数据库时最好加上 `IF EXISTS` 条件，避免数据库不存在导致报错

{{< /notice >}}

#### 查看所有数据库

```sql
SHOW DATABAS;
```

#### 使用数据库

```sql
USE `sql_study`;

-- 语句结构
USE `${数据库名}`;
```

#### 查看数据库的定义语句

```sql
SHOW CREATE DATABASE `sql_study`;

-- 语句结构
SHOW CREATE DATABASE `${数据库名}`;
```

### 表操作

#### 创建表

```sql
CREATE TABLE IF NOT EXISTS `user_info` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '序列号',
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '姓名',
  `age` int DEFAULT '0' COMMENT '年龄',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT '用户表';

-- 语句结构
CREATE TABLE [IF NOT EXISTS] `${表名}` (
  `${字段名}` ${字段类型} [${字段属性}] COMMENT '${字段备注}',
  ...
  `${字段名}` ${字段类型} [${字段属性}] COMMENT '${字段备注}',
  PRIMARY KEY (`${字段名}`)
) ${引擎} [${表属性}] DEFAULT CHARSET=${字符集} COLLATE=${排序规则} COMMENT '${表备注}';
```

{{< notice info >}}

创建表时最好加上 `IF NOT EXISTS` 条件，避免表已存在导致报错

{{< /notice >}}

#### 删除表

```sql
DROP TABLE IF EXISTS `user`;

-- 语句结构
DROP TABLE [IF EXISTS] `${表名}`;
```

{{< notice info >}}

删除表时最好加上 `IF EXISTS` 条件，避免表不存在导致报错

{{< /notice >}}

#### 查看所有表

```sql
SHOW TABLES;
```

#### 查看表结构

```sql
DESC `user`;

-- 语句结构
DESC `${表名}`;
```

#### 查看表定义语句

```sql
SHOW CREATE TABLE `user`;

-- 语句结构
SHOW CREATE TABLE `${表名}`;
```

#### 修改表结构

##### 修改表名

```sql
ALTER TABLE `user` RENAME AS `user1`;

-- 语句结构
ALTER TABLE `${旧表名}` RENAME AS `${新表名}`;
```

##### 编辑字段

```sql
-- 添加字段
ALTER TABLE `user` ADD `score` INT DEFAULT '0' COMMENT '成绩';
-- 修改字段类型
ALTER TABLE `user` MODIFY `score` FLOAT DEFAULT '0' COMMENT '成绩';
-- 重命名字段
ALTER TABLE `user` CHANGE `score` `math_score` FLOAT DEFAULT '0' COMMENT '成绩';

-- 语句结构
-- 添加字段
ALTER TABLE `${表名}` ADD `${字段名}` ${字段类型} [${字段属性}] COMMENT '${字段备注}';
-- 修改字段类型
ALTER TABLE `${表名}` MODIFY `${字段名}` ${新字段类型} [${新字段属性}] COMMENT '${新字段备注}';
-- 重命名字段
ALTER TABLE `${表名}` CHANGE `${旧字段名}` `${新字段名}` ${新字段类型} [${新字段属性}] COMMENT '${新字段备注}';
```

{{< notice warning >}}

不要使用 `CHANGE` 字段进行字段重命名以外的操作

使用 `MODIFY` 来修改字段类型和属性

{{< /notice >}}

##### 删除字段

```sql
ALTER TABLE `user` DROP `score`;

-- 语句结构
ALTER TABLE `${表名}` DROP `${字段名}`;
```

#### 设置外键

{{< notice info >}}

里演示的时数据库级别的外键，有引用关系时主表无法删除，故在实际生产中不建议使用

生产上数据库只是单纯的数据，只有字段和记录

生产上若想使用多张表数据，可使用应用程序来实现

{{< /notice >}}

方式一：
创建表的时候增加约束

```sql
-- 创建年级表
CREATE TABLE IF NOT EXISTS `grade` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '序列号',
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '年级名',
  `comment` varchar(40) DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT '年级表表';

-- 创建学生表
CREATE TABLE IF NOT EXISTS `student` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '序列号',
  `age` int DEFAULT '0' COMMENT '年龄',
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '姓名',
  `grade_id` int DEFAULT NULL COMMENT '年级id',
  PRIMARY KEY (`id`),
  -- 定义外键 Key 并将添加引用字段
  KEY `FK_grade_id` (`grade_id`),
  -- 创建外键约束，指定被引用表和字段名
  CONSTRAINT `FK_grade_id` FOREIGN KEY (`grade_id`) REFERENCES `grade`(`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT '年级表表';

-- 语句结构
  -- 定义外键 Key 并将添加引用字段
  KEY `${FK_外键名}` (`${从字段名}`),
  -- 创建外键约束，指定被引用表和字段名
  CONSTRAINT `${FK_外键名}` FOREIGN KEY (`${从字段名}`) REFERENCES `${主表名}`(`${主字段名}`)
```

{{< notice tip >}}

约定成俗的，将外键定义成 `FK_` 开头

被引用数据的表称为 `主表`

引用数据的表称为 `从表`

删除直接删除主表会报错，必须先解除引用关系

{{< /notice >}}

方式一：
先创建正常表，再通过命令创建外键引用关系

```sql
ALTER TABLE `student` ADD CONSTRAINT `FK_grade_id` FOREIGN KEY (`grade_id`) REFERENCES `grade` (`id`);

-- 语句结构
ALTER TABLE `${从表名}` ADD CONSTRAINT `${FK_外键名}` FOREIGN KEY (`${从字段名}`) REFERENCES `${主表名}` (`${主字段名}`);
```

## DML - Data Manipulation Language

### INSERT

```sql
-- 插入单条数据
INSERT INTO `grade` (`name`,`comment`) VALUE ('大一','第一年');

-- 插入多条数据
INSERT INTO `grade` (`name`,`comment`) VALUE ('大二','第二年'),('大三','第三年');

-- 语句结构
INSERT INTO `${表名}`
  (`${字段}`,`${字段}`)
  VALUE
  ('${值}','${值}'),
  ('${值}','${值}');
```

{{< notice info >}}

数据会与字段一一匹配

当没有指定字段名时，将会按照所有字段顺序匹配

{{< /notice >}}

### UPDATE

```sql
UPDATE `student` SET `name`='李四',`age`='22' WHERE `name`='李4';

-- 语句结构
UPDATE `${表名}`
  SET
  `${字段}`='${值}',
  ...
  `${字段}`='${值}'
  [WHERE ${条件}];
```

{{< notice warning >}}

修改数据时**一定要带条件**

不带条件**默认修改所有记录**

每一行条件结果为一个布尔值，结果为 `True` 的行才会修改

{{< /notice >}}

### DELETE

```sql
DELETE FROM `student` WHERE `id`='5';

-- 语句结构
DELETE FROM `${表名}` [WHERE ${条件}];
```

{{< notice warning >}}

删除表中的记录一般不使用 `DELETE` 语句

生产环境中删除数据一般使用 `is_delete` 字段作为标识符进行伪删除

{{< /notice >}}

### TRUNCATE

```sql
-- 清空数据表，结构与外键约束不变
TRUNCATE `test`;

-- 语句结构
TRUNCATE `${表名}`;
```

{{< notice tip >}}

TRUNCATE 与 DELETE 的区别

- 都能删除数据
- 都会保留表结构
- TRUNCATE 会重新设置自增列
- DELETE 不会重新设置自增列
- TRUNCATE 不会影响事务
- DELETE 会影响事务

{{< /notice >}}

## DQL - Data Query Language

数据库中最核心的语言，语句： `SELECT` 。

### 基本查询

查询数据库：

```sql
-- 查询所有字段和记录
SELECT * FROM `student`;
-- 查询指定字段和记录
SELECT `name`,`grade_id` FROM `student` WHERE `id`='2';
-- 使用字段别名
SELECT `name` AS '学生姓名',`grade_id` AS '年级id' FROM `student`;

-- 语句结构
SELECT
  `${字段名}` [AS '${字段别名}'],
  ...
  `${字段名}` [AS '${字段别名}']
  FROM `${表名}` [AS '${表别名}']
  [WHERE ${条件}];
```

除了查询数据库中的数据外， SELECT 还能查询一些其他项目：

```sql
-- 去重结果
SELECT DISTINCT `name` FROM `student`;
-- 查询函数结果
SELECT VERSION();
-- 查询计算结果
SELECT 10*2+5 AS '计算结果';
-- 查询系统变量
SELECT @@auto_increment_increment;
```

### 使用函数

#### 常用函数

```sql
SELECT ABS(-1);        -- 绝对值
SELECT CEILING(9.3);   -- 向上取整
SELECT FLOOR(9.3);     -- 向下取整
SELECT RAND();         -- 生成随机数
SELECT SIGN(1);        -- 判断符号

SELECT LENGTH('hello,word!');                 -- 计算字符串长度
SELECT INSERT('hello,word!',7,4,'neko');      -- 删除插入字符串
SELECT REPLACE('hello,word!','word','neko');  -- 替换字符串

SELECT UPPER('hello,word!');  -- 转为大写
SELECT LOWER('INSERT');       -- 转为小写

SELECT CURDATE();   -- 当前日期
SELECT NOW();       -- 当前时间
SELECT LOCALTIME(); -- 本地时间
SELECT SYSDATE();   -- 系统日期

SELECT USER();          -- 当前用户
SELECT VERSION();       -- 当前版本

-- 拼接字段
SELECT CONCAT('姓名：',`name`,'；年级id：',`grade_id`) AS 学生信息 FROM `student`;
-- 数据加密
INSERT INTO `testmd5`(`id`,`user`,`pwd`) VALUES(1,`neko`,MD5('123456'))

-- 函数语法
${函数名}(${参数})
```

#### 聚合函数

```sql
SELECT COUNT(*) FROM `student`;       -- 统计所有的字段，不会忽略 BULL 值，慢
SELECT COUNT(1) FROM `student`;       -- 忽略所有的字段，用 1 代替，不会忽略 NULL 值，最快
SELECT COUNT(`name`) FROM `student`;  -- 值统计指定的字段，会忽略 NULL 值，快

SELECT SUM(age) AS '年龄总和' FROM `student`;  -- 求和
SELECT AVG(age) AS '年龄总和' FROM `student`;  -- 平均
SELECT MAX(age) AS '年龄总和' FROM `student`;  -- 最高
SELECT MIN(age) AS '年龄总和' FROM `student`;  -- 最低

-- 函数语法
${函数名}(${参数})
```

### 条件子句

#### 模糊查询

```sql
-- % 表示任意个字符
-- _ 表示一个字符
SELECT `name` FROM `student` WHERE `name` LIKE '王%';

-- 语句结构
WHERE `${字段名}` LIKE ${匹配字符}
```

#### 在集合中

```sql
-- IN 后面的合集应为一个具体的值
SELECT `name` FROM `student` WHERE `id` IN (1,3);

-- 语句结构
WHERE `${字段名}` IN ${集合}
```

#### 查询空值

```sql
-- 应考虑 NULL 和空字符串两种情况
SELECT `id`,`name` FROM `student` WHERE `name`='' OR `name` IS NULL;

-- 语句结构
WHERE `${字段名}`='' OR `${字段名}` IS NULL
```

### 联表查询

#### 多表连接

```sql
-- 查询多个表的情况，应先从两个表的连接写起
SELECT s.`id` AS '学生id',s.`name` AS '学生姓名',g.`name` AS '年级'
FROM `student` AS s
INNER JOIN `grade` AS g
ON s.grade_id = g.id;

-- 语句结构
SELECT [${表名/表别名}.]`${字段名}` [AS '${字段别名}'],
  ...,
  [${表名/表别名}.]`${字段名}` [AS '${字段别名}']
FROM `${表名}` [[AS] ${表别名}]
${联表方式} JOIN `${表名}` [[AS] ${表别名}]
  ON [${表名/表别名}.]`${字段名}` = [${表名/表别名}.]`${字段名}`;
...
${联表方式} JOIN `${表名}` [[AS] ${表别名}]
  ON [${表名/表别名}.]`${字段名}` = [${表名/表别名}.]`${字段名}`;
[WHERE ${条件}]
```

联表操作包含 7 种 JOIN 类型，详细请查看：
[数据库中的 7 种 join 操作](../join-in-database)

#### 自连接

有这样一张表：

```sql
create table if not exists `category`(
    `category_id` int auto_increment,
    `pid` int,
    `name` varchar(40),
    primary key (`category_id`)
);
```

| category_id | pid | name     |
| ----------- | --- | -------- |
| 2           | 1   | 信息技术 |
| 3           | 1   | 软件开发 |
| 4           | 3   | 数据库   |
| 5           | 1   | 美术设计 |
| 6           | 3   | Web 开发 |
| 7           | 5   | PS 技术  |
| 8           | 2   | 办公信息 |

进行自连接，可将目录的层次关系列出：

```sql
SELECT p.`name` AS '父目录',s.`name` AS '子目录'
FROM `category` AS p,`category` AS s
WHERE p.`category_id`=s.`pid`;
```

结果：

| 父目录   | 子目录   |
| -------- | -------- |
| 软件开发 | 数据库   |
| 软件开发 | Web 开发 |
| 美术设计 | PS 技术  |
| 信息技术 | 办公信息 |

自连接是将自身看作两张完全一样的表进行连接操作。

### 子查询

当要查询的字段在同一张表中，但是筛选条件在其他的表中时，可以使用子查询简化查询语句

```sql
SELECT `id` AS '学生id',`name` AS '学生姓名'
FROM `student`
WHERE `grade_id` IN (
  SELECT `id` from `grade` WHERE `name` IN ('大二','大三')
  );

-- 语句结构
WHERE ${字段名} ${运算} (
  ${子查询}
)
```

{{< notice tip >}}

子查询就是在 `WHERE` 子句中嵌入 `SELETE` 查询

子查询可能会多次遍历表，一般情况下性能会低于联表查询

一般通过联表查询来替换多个子查询以提高性能

正确编写联表查询是高性能的前提，不正确的联表方式可能导致性能降低

{{< /notice >}}

### 分组

```sql
SELECT g.`name` AS '年级', avg(s.`age`) AS '平均年龄' FROM `student` AS s
INNER JOIN grade AS g
ON s.grade_id = g.id
GROUP BY s.grade_id
HAVING 平均年龄 >= 20;

-- 语句结构
SELECT ${聚合函数} FROM `${表名}`
GROUP BY ${字段名}
HAVING ${筛选聚合条件};
```

根据指定的字段名的不同的值进行分组，每组记录根据聚合函数计算结果。

### 排序

```sql
SELECT s.`id`,s.`name`,g.`name`
FROM `student` AS s
INNER JOIN `grade` AS g
ON s.grade_id=g.id
ORDER BY s.`id` DESC;

-- 语句结构
ORDER BY [${表名/表别名}.]`${字段名}` [${ASC/DESC}]
```

| 关键字 | 排序方式 |
| ------ | -------- |
| ACS    | 顺序     |
| DESC   | 逆序     |

### 分页

```sql
SELECT s.`id`,s.`name`,g.`name`
FROM `student` AS s
INNER JOIN `grade` AS g
ON s.grade_id=g.id
ORDER BY s.`id` DESC;
LiMIT 2,5

-- 语句结构
LiMIT ${起始位置},${查询条数}
```

{{< notice tip >}}

对于 `LIMIT x,y`

- x 查询的位置，从第 x 条开始
- y 查询的条数，本次查询 y 条记录

{{< /notice >}}

### 短语顺序

```sql
SELECT [DISTINCT] [${表名/表别名}.]`${字段名}` [[AS] ${字段别名}][,...]
FROM `${表名}` [[AS] ${表别名}][,...]
[${联表方式} JOIN `${表名}` [[AS] ${表别名}] ON [${表名/表别名}.]`${字段名}`=[${表名/表别名}.]`${字段名}`]
[WHERE ${条件语句}]
[GROUP BY [${表名/表别名}.]`${字段名}` [HAVING ${筛选聚合条件}]]
[ORDER BY [${表名/表别名}.]`${字段名}` [ASC/SESC]]
[LiMIT ${起始记录数},${读取记录数}]
[UNION ${查询}]
```

## DCL - Data Control Language

### 用户操作

```sql
CREATE USER 'neko' IDENTIFIED BY '123456';
SET PASSWORD FOR 'neko' = PASSWORD('123456');
RENAME USER 'neko' TO 'neko1';
DROP USER 'neko';

-- 语句结构
CREATE USER '${用户名}' IDENTIFIED BY '${密码}';
SET PASSWORD [FOR '${用户名}'] = PASSWORD('${密码}');
RENAME USER '${旧用户名}' TO '${新用户名}';
DROP USER '${用户名}';
```

### 权限管理

```sql
GRANT ALL PRIVILEGES ON *.* TO 'neko';     -- 授予权限
SHOW GRANTS FOR 'neko';                    -- 查看权限
REVOKE ALL PRIVILEGES FROM *.* TO 'neko';  -- 撤销权限

-- 语句结构
GRANT ${权限列表} ON ${数据库名}.${表名} TO '${用户名}'[@'${主机}'];
SHOW GRANTS FOR ${用户名}[@'${主机}'];
REVOKE ${权限列表} ON ${数据库名}.${表名} FROM '${用户名}'[@'${主机}'];
```

## 知识点

### 表类型

| 表类型   | MyISAM | **InnoDB** | Memory   | CSV          |
| -------- | ------ | ---------- | -------- | ------------ |
| 事务支持 | 不支持 | 支持       | 不支持   | 不支持       |
| 行锁     | 不支持 | 支持       | 不支持   | 不支持       |
| 外键索引 | 不支持 | 支持       | 不支持   | 不支持       |
| 全文索引 | 支持   | 不支持     | 不支持   | 不支持       |
| 占用空间 | 100%   | 200%       | 内存读写 | 用于数据交互 |

### 字符集

| 字符集      | 长度 | 说明                                                  |
| ----------- | ---- | ----------------------------------------------------- |
| latin1      | 1    | 老版本 MySQL 默认字符集                               |
| GBK         | 2    | 支持中文，但是不是国际通用字符集                      |
| utf8mb3     | 3    | 支持中英文混合场景，是国际通用字符集                  |
| **utf8mb4** | 4    | 默认字符集，完全兼容 utf8mb3，可存储 emoji 等特殊字符 |

### 排序规则

| 规则                   | 解释                                                                                 |
| ---------------------- | ------------------------------------------------------------------------------------ |
| **utf8mb4_0900_ai_ci** | Unicode 9.0 规范，不区分音调，不区分大小写，相同字母的大小和音调写在排序时视为相同。 |
| utf8mb4_0900_as_cs     | Unicode 9.0 规范，区分音调，区分大小写，相同字母的大小写和音调在排序时视为不同。     |
| utf8mb4_0900_bin       | Unicode 9.0 规范，通过字符串的二进制进行比较                                         |
| utf8mb4_unicode_ci     | 根据 unicode 编码规则实行排序，准确性高                                              |

### 字段

在生产环境中，每个表通常会有默认的几个字段：

- `id` // 主键
- `version` // 版本，用于乐观锁
- `is_delete` // 伪删除
- `gmt_create` // 记录的创建时间
- `gmt_update` // 记录的更新时间

### 字段类型

#### 数值

| 类型      | 解释             | 长度   |
| --------- | ---------------- | ------ |
| tinyint   | 微型整数         | 1 Byte |
| smallint  | 小型整数         | 2 Byte |
| mediumint | 中小型整数       | 3 Byte |
| **int**   | 整数             | 4 Byte |
| big       | 大型整数         | 8 Byte |
| float     | 单精度浮点数     | 4 Byte |
| double    | 双精度浮点数     | 8 Byte |
| decimal   | 字符串形式浮点数 | -      |

{{< notice tip >}}

int 类型为常用类型

decimal 类型使用字符串形式来存储数值，使用的时候再转化为数值，避免精度问题，一般用于金融领域

数值类型诸如 `int(8)` 的 `Type(M)` 形式，`M` 指的是数据的显示宽度，与零填充有关，与数据的值大小无关

{{< /notice >}}

#### 字符串

| 类型        | 解释           | 长度       |
| ----------- | -------------- | ---------- |
| char        | 固定长度字符串 | 2^8-1      |
| **varchar** | 可变长字符串   | 0 ~ 2^16-1 |
| tinytext    | 微型文本       | 0 ~ 2^8-1  |
| text        | 文本           | 0 ~ 2^16-1 |

{{< notice tip >}}

char / varchar 类型一般用作存储经常读写的变量性质的值

text / tinytext 类型一般用于关联性不强的整体文本，如文章等

能用 varchar 就不用其他的类型

字符串类型的诸如 `varchar(20)` 的 `Type(M)` 形式，`M` 指的是字符床的最大长度

{{< /notice >}}

#### 时间和日期

| 类型          | 解释     | 形式                |
| ------------- | -------- | ------------------- |
| date          | 日期     | yyyy-mm-dd          |
| time          | 时间     | hh:mm:ss            |
| **datetime**  | 日期时间 | yyyy-mm-dd hh:mm:ss |
| **timestamp** | 时间戳   | 1666760950          |
| year          | 年份     | yyyy                |

{{< notice tip >}}

datetime 类型为常用类型

timestamp 类型为 1970-01-01 00:00:00 到现在时间点的**秒数**

{{< /notice >}}

#### NULL

| 类型 | 解释       | 形式   |
| ---- | ---------- | ------ |
| NULL | 无值，未知 | (NULL) |

{{< notice info >}}

不要使用 `NULL` 进行计算，计算结果一律为 `NULL` ，无意义

{{< /notice >}}

### 字段属性

UNSIGNED

- 声明该字段不能为负数
- 只有整数类型能用

ZEROFILL

- 声明该字段位数不足时用 `0` 填充

AUTO_INCREMENT

- 自动在上一条记录的基础上增加，默认为 `1`
- 只有整数类型能用
- 通常用于设置唯一的主键

NOT NULL

- 声明该字段不能为 `NULL`
- 该字段若不赋值则报错

NULL

- 声明该字段允许 `NULL`
- 该字段若不赋值且未设置默认值时则记录为 `NULL`

DEFAULT

- 声明该字段的默认值
- 该字段若不赋值则记录为 `指定的值`

PRIMARY KEY

- 声明该字段为主键
- 一般一张表只有一个
- 一般写在字段声明的最后 `` PRIMARY KEY(`字段`) ``

### 运算操作

#### 算数运算

| 算数运算符 | 含义 |
| ---------- | ---- |
| `+`        | 加   |
| `-`        | 减   |
| `*`        | 乘   |
| `/` `DIV`  | 求商 |
| `%` `MOD`  | 求余 |

#### 比较运算

| 比较运算符 | 含义     |
| ---------- | -------- |
| `=`        | 等于     |
| `<>` `!=`  | 不等于   |
| `>`        | 大于     |
| `>=`       | 大于等于 |
| `<`        | 小于     |
| `<=`       | 小于等于 |

| 比较运算符          | 含义             |
| ------------------- | ---------------- |
| `IS NULL`           | 为 NULL          |
| `IS NOT NULL`       | 不为 NULL        |
| `BETWEEN` x `AND` y | 在 [x, y] 范围中 |
| `LIKE`              | 模糊匹配         |
| `IN`                | 在集合中         |

#### 逻辑运算

| 逻辑运算符 | 含义     |
| ---------- | -------- |
| `AND` `&&` | 与运算   |
| `OR` `⏐⏐`  | 或运算   |
| `NOT` `!`  | 非运算   |
| `XOR`      | 异或运算 |

### 事务

事务是指将一系列数据库操作结合成一个整体，要么同时成功，要么同时失败。

#### ACID 原则

- Atomic - 原子性
  事务中各项操作，要么全做要么全不做，任何一项操作的失败都会导致整个事务的失败
- Consistent - 一致性
  事务结束后系统状态是一致的
- Isolated - 隔离性
  并发执行的事务彼此无法看到对方的中间状态，各种操作彼此互不影响
- Durable - 持久性
  事务完成后所做的改动都会被持久化，即使发生灾难性的失败

#### 隔离所导致的问题

由于事务在操作数据库时会向数据库加锁，不同的锁可对其他事务的读写有着不同的效果。
在数据库中提供了几种事务的隔离级别：

| 隔离级别                    | 级别 | 描述                                                                                                 |
| --------------------------- | ---- | ---------------------------------------------------------------------------------------------------- |
| read uncommitted - 读未提交 | 低   | 允许其他事务读取当前事务修改但未提交的数据                                                           |
| read committed - 读已提交   | 中   | 允许其他事务读取当前事务已经提交的数据，但数据库可能并未操作完成，会发生短时间内两次查询不一致的情况 |
| repeatable read - 可重复读  | 高   | 确保同一事务的多个实例在并发读取数据时，会看到同样的数据行                                           |
| serializable - 序列化       | 最高 | 强制事务排序，解决相互冲突，可能导致大量的超时现象的和锁竞争，效率低下                               |

相应的，也会出现几种问题：

| 隔离级别   | 严重级别 | 描述                                                                                         |
| ---------- | -------- | -------------------------------------------------------------------------------------------- |
| 回滚覆盖   | 高       | 事务失败回滚时，数据被其他事务修改，导致一致性被破坏                                         |
| 脏读       | 高       | 事务读取了其他事务被修改还未提交的数据，该事务可能失败，读取的数据不安全                     |
| 不可重复读 | 中       | 事务读取了其他事务已经提交但数据库未处理完毕的数据，可能导致在该事务中，两次读取的值不同     |
| 提交覆盖   | 低       | 事务读取数据后，其他事务更改了该数据，当前事务在写入时再次更改该数据，导致其他事务的更改失效 |
| 幻读       | 低       | 事务读取后，其他事务向该表增加记录，可能导致该事务中后续读取增加记录                         |

不同的隔离级别可能会导致不同的问题，事务隔离级别越高，并发问题就越小，但是并发处理能力越差；

| 隔离级别                    | 回滚覆盖 | 脏读     | 不可重复读 | 提交覆盖 | 幻读     |
| --------------------------- | -------- | -------- | ---------- | -------- | -------- |
| read uncommitted - 读未提交 | 不会发生 | 可能发生 | 可能发生   | 可能发生 | 可能发生 |
| read committed - 读已提交   | 不会发生 | 不会发生 | 可能发生   | 可能发生 | 可能发生 |
| repeatable read - 可重复读  | 不会发生 | 不会发生 | 不会发生   | 不会发生 | 可能发生 |
| serializable - 序列化       | 不会发生 | 不会发生 | 不会发生   | 不会发生 | 不会发生 |

#### 事务的操作

```sql
-- mysql 默认开启事务并自动提交
SET autocommit = 0;  -- 关闭自动提交

START TRANSACTION;  -- 事务开始
[执行语句]
SAVEPOINT ${检查点名};  -- 生成检查点
[执行语句]
ROLLBACK TO SAVEPOINT ${检查点名};  -- 回滚到检查点
[执行语句]
RELEASE SAVEPOINT ${检查点名};  -- 释放检查点
[执行语句]

-- 事务只能以回滚或提交结束
COMMIT;    -- 提交，完成事务
ROLLBACK;  -- 回滚，撤销事务

SET autocommit = 1;  -- 打开自动提交
```

### 索引

> 索引(Index)是帮助 MySQL 高效获取数据的数据结构.
> 提取句子的主干索引(Index)是一种数据结构。

#### 分类

| 索引分类 | 描述                                       |
| -------- | ------------------------------------------ |
| 主键索引 | 一张表只能有一个，字段中所有的记录不能相同 |
| 唯一索引 | 一张表可以有多个，字段中所有的记录不能相同 |
| 常规索引 | 一张表可以有多个，字段中所有的记录可以相同 |
| 全文索引 | 一般用于字符串，遍历所有的文字             |

#### 索引原则

- 索引不是越多越好
- 不要对经常变动的数据加索引
- 小数据量（十万条以下）不用加索引
- 索引一般家在经常查询的字段上

### 数据库设计

当数据较多且关系较为复杂时，需要进行数据库的设计。

- 糟糕的数据库设计
  - 数据冗余
  - 有较多外键，数据插入和删除都会很麻烦
  - 程序性能差
- 良好的数据库设计
  - 节省空间
  - 能保证数据库完整性
  - 方便开发系统

软件开发中，数据库设计流程：

1. 分析需求
   - 分析业务和数据库处理的需求
2. 概要设计
   - 设计 E-R 关系图

#### 三大范式

为何需要数据规范化？

- 信息重复
- 更新异常
- 插入异常
- 删除异常

##### 第一范式 - 1NF

- 要求：每一个字段均为**不可再分**的原子数据项。

##### 第二范式 - 2NF

- 前提：满足第一范式
- 要求：一张表只描述一件事，每一个字段均与主键**相关**，而不是与主键的**某一部分**相关

##### 第三范式 - 3NF

- 前提：满足第二范式
- 要求：消除传递依赖，每一个字段均与主键**直接相关**，而不能**间接**相关。

{{< notice info >}}

规范与性能时不可兼得的

在严格满足范式规范的情况下，一次查询可能关联很多表，导致性能问题

{{< /notice >}}
