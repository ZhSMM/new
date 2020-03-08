# sql

## 简介

sql是一门结构化查询语言，用来访问和操作数据库系统（SQL大小写不敏感）。SQL 语句用于取回和更新数据库中的数据。SQL 可与数据库程序协同工作，比如 MS Access、DB2、Informix、MS SQL Server、Oracle、Sybase 以及其他数据库系统。

RDBMS：关系型数据库管理系统，是SQL的基础，也是关系型数据库的基础，包括MySql、Oracle。RDBMS中的数据存储在称为表（tables)的数据库对象中。表是相关的数据项的集合，由行和列组成。

## 分类

主要分为SQL DML和DDL：

### SQL DML

SQL DML：数据操作语言，主要包括增删改查，即

- select：查询；
- update：更新；
- delete：删除；
- insert into：插入。

SQL DDL：数据定义语言，用于操作数据库、表、索引等，即

- create database：创建数据库；
- alter database：修改数据库；
- create table：创建数据表；
- alter table：变更数据表；
- drop table：删除数据表；
- create index：创建索引；
- drop index：删除索引。