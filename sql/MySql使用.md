# MySql使用

## 基本使用

### 数据库操作

- 查看数据库：`show databases;`

- 创建数据库：`create database if not exists 数据库名 default charaset utf8 collate utf8_general_ci;`

- 使用mysqladmin创建数据库：

  > 普通用户，你可能需要特定的权限来创建或者删除 MySQL 数据库；所以我们这边使用root用户登录，root用户拥有最高权限，可以使用 mysql mysqladmin 命令来创建数据库。
  >
  > `mysqladmin -uroot -p create 数据库名`

- 删除数据库：`drop database 数据库名;`或使用`mysqladmin -uroot -p drop 数据库名`

- 选择数据库：`use 数据库名;`

### 数据类型

mysql大概支持三类：数值、日期/时间和字符串（字符）类型。

#### 数值类

|     类型     |                   大小                   |  范围（有符号）   | 范围（无符号） |      用途       |
| :----------: | :--------------------------------------: | :---------------: | :------------: | :-------------: |
|   TINYINT    |                  1 字节                  |  (-2^7^，2^7^-1)  |                |    小整数值     |
|   SMALLINT   |                  2 字节                  | (-2^15^，2^15^-1) |                |    大整数值     |
|  MEDIUMINT   |                  3 字节                  | (-2^23^，2^23^-1) |                |    大整数值     |
| INT或INTEGER |                  4 字节                  | (-2^31^，2^31^-1) |                |    大整数值     |
|    BIGINT    |                  8 字节                  | (-2^63^，2^63^-1) |                |   极大整数值    |
|    FLOAT     |                  4 字节                  |                   |                | 单精度 浮点数值 |
|    DOUBLE    |                  8 字节                  |                   |                | 双精度 浮点数值 |
|   DECIMAL    | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 |  依赖于M和D的值   | 依赖于M和D的值 |     小数值      |

#### 日期类

| 类型      | 大小 (字节) | 范围                                                         | 格式                | 用途                     |
| --------- | ----------- | ------------------------------------------------------------ | ------------------- | ------------------------ |
| DATE      | 3           | 1000-01-01/9999-12-31                                        | YYYY-MM-DD          | 日期值                   |
| TIME      | 3           | '-838:59:59'/'838:59:59'                                     | HH:MM:SS            | 时间值或持续时间         |
| YEAR      | 1           | 1901/2155                                                    | YYYY                | 年份值                   |
| DATETIME  | 8           | 1000-01-01 00:00:00/9999-12-31 23:59:59                      | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| TIMESTAMP | 4           | 1970-01-01 00:00:00/2038  结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |

#### 字符串类

| 类型       | 大小                | 用途                            |
| ---------- | ------------------- | ------------------------------- |
| CHAR       | 0-255字节           | 定长字符串                      |
| VARCHAR    | 0-65535 字节        | 变长字符串                      |
| TINYBLOB   | 0-255字节           | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | 0-255字节           | 短文本字符串                    |
| BLOB       | 0-65535字节         | 二进制形式的长文本数据          |
| TEXT       | 0-65535字节         | 长文本数据                      |
| MEDIUMBLOB | 0-16 777 215字节    | 二进制形式的中等长度文本数据    |
| MEDIUMTEXT | 0-16 777 215字节    | 中等长度文本数据                |
| LONGBLOB   | 0-4 294 967 295字节 | 二进制形式的极大文本数据        |
| LONGTEXT   | 0-4 294 967 295字节 | 极大文本数据                    |

- CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。

- BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。
- BLOB 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 BLOB 类型：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。它们区别在于可容纳存储范围不同。
- 有 4 种 TEXT 类型：TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。对应的这 4 种 BLOB 类型，可存储的最大长度不同，可根据实际情况选择。

### 数据表操作

- 创建数据表：`create table if not exist 表名 (字段 字段类型 约束，);`
- 删除数据表：`drop table 表名;`

### 增删改查

- 插入数据：`insert into table_name (字段1,字段2) values(值1，值2);`
- 查询数据：`select column1,column2 from table_name [where] [limit n][offset m];`
  - limit n：返回的记录数；
  - offset m：select语句开始时的偏移量；
- 更新数据：`update table_name set name="zhan" where id=10; `
- 删除数据：`delete from table_name where id=10;`

### 其他语法

- like "%"：模糊查询
- union [all|distinct]：链接两个以上的select语句的查询结果
- order by 字段  [desc]:排序
- group by :`select coulmn,function(column_name) from table_name where column_name operator value group by column_name;`分组统计



