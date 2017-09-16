---
title: sqlBasic
tags:
  - SQL
categories: Database
date: 2016-08-14 23:13:07
keywords: sql基础
description:
---
sql语句基础。
<!--more-->
## 前言
SQL语言指结果查询语言，用于执行查询的语法，但是也包含了用于更新、插入、和删除记录的语法。主要由：数据定义语言（Data Definition Language，DDL），数据操作语言（Data Manipulation Language，DML），数据控制语言（DataControl Language，DCL）三者构成。

## DDL
DDL用于定义和管理SQL数据库中所有对象的语言，它是我们有能力创建或修改数据库中的相关对象（数据库，表等等）已经他们之间的关系。常用的相关重要语句包括：
+ Create Database：创建数据库
+ Alter Database:修改数据库
+ Create Table：创建数据库表
+ Alter Table：更新表结构
+ Drop Table：删除表
+ Create Index：创建索引
+ Drop Index：删除索引
其他还有对存储结构，事务，视图等的相关操作。

## DML
DML是用于操作数据库对象中所包含的数据的语言，其操作对象是记录（record）。其操作方法包括：
+ Insert语句：用于插入新的记录，可以进行批量操作。
+ Delete语句：用于删除表中的记录，可批量操作。
+ Update语句：更新表中的记录，可批量操作。
与DDL不同的是，DML是对记录进行操作，而DDL是对数据库中的对象进行操作。两者的操作对象不同。

## DCL
DCL是提供对数据库操作权限的相关操作，比如说数据库授权，数据库角色等等，其操作方法主要有两个：
+ Grant：赋予权限。
+ Revoke：撤回权限。
## Example
实例基于MySql实现。

### 数据库定义
#### 定义数据库
``` sql
drop database if exists DATABASE_Name;
create database DATABASE_NAME default character set utf8 ;
```
首先判断数据库系统中是否存在同名数据库，如果存在则删除（通常不直接删除数据库）。
创建语句表示：创建了一个名为DATABASE_NAME的数据库，其字符存储的格式为utf8。 
我们创建一个名为Test的数据库：

``` sql
create database Test default character set utf8 ;
```
#### 定义表结构
定义表结构由DDL实现，主要是：创建表，修改表，删除表。

``` sql
CREATE TABLE TABLE_NAME ( id int PRIMARY KEY AUTO_INCREMENT, num1 int not null);
```
上面代码创建了一张表，表中有两个属性，分别为整形自增的组建，`PRIMARY KEY`表示主键，`AUTO_INCREMENT`表示该属性自增。
当创建表之后，可以通过`desc TABLE_NAME;`，`describe TABLE_NAME`或者`show columns from TABLE_NAME`来查看表结构。
如果需要查看数据库里所有的表时，可以通过`show tables`来查询。以下创建两张表，person表和department表。

``` sql
CREATE TABLE person(id int PRIMARY KEY AUTO_INCREMENT, name varchar(20) not null);
CREATE TABLE department(id int PRIMARY KEY AUTO_INCREMENT, name varchar(20) not null);
```
当我们定义完表后，还可以通过相关DDL语言进行表结构的修改。

``` sql
ALTER TABLE TABLE_NAME add COLUMNS_NAME COLUMNS_TYPE;
```
以上代码用于修改表的结构，可以通过`ALTER`命令来对表的字段进行增删改。
+ 增加字段 ` add COLUMNS_NAME COLUMNS_TYPE;` 。
+ 删除字段 `drop COLUMNS_NAME;`。
+ 修改字段名称 `change old_COLUMNS_NAME new_COLUMNS_NAME COLUMNS_TYPE`（修改字段名称时，需要重新制定字段的类型）。
+ 修改字段的类型 `modify COLUMNS_NAME new_COLUMNS_TYPE`.

目前数据库里有两张表，分别为`person`和`department`表。我们可以通过修改表的结构来关联二者。
我们为person表添加指向department表的外键。
首先，先在person表添加一个存储外键的字段`dp_fk`,其数据类型需要与department的主键相同。

``` sql
ALTER TABLE person add dp_fk int;
```
然后引入关联。由于如果表的主键需要为其他表的外键，则该主键必须是唯一的，因此需要为`department`表的主键添加唯一约束。

``` sql
alter table department change id id int UNIQUE;
ALTER TABLE person add CONSTRAINT fk_p_dp foreign key(dp_fk) REFERENCES department(id) on delete restrict on update restrict;
```
_注意：_
+ _如果需要添加一个外键时，主表必须存在一个字段与外表的唯一主键的数据类型完全相同。_
+ _`on`后边主要是当外表的记录被删除时，主表的更新策略，分别有
+ >`cascade`：在外表update/delete记录时，同时也在主表进行update/delete相关联的记录。
+ >`restrict`：如果主表有记录匹配的记录时，不允许外表对其主键进行update/delete操作（但是非主表外键的其他字段可以）。
+ >`set null`：置空处理。即当外表的记录被删除时，将主表上的匹配的记录主键置为空值，因此在该模式下，主表的外键字段不能设置为`not null`。
+ >`No action`：同`restrict`。

对表的字段创建索引
索引是数据表里所有记录引用指针。一下是三种创建数据库索引的方式：

``` sql
-- 第一种直接创建
CREATE INDEX INDEX_NAME ON TABLE_NAME(COLUNMN_NAME)；
--第二种，通过修改表结构的方式
ALTER TABLE TABLE_NAME ADD INDEX INDEX_NAME ON(COLUMN_NAME)；
--第三种方式，创建表的时候创建索引。
CREATE TABLE TABLE_NAME(id int not null AUTO_INCREMENT, INDEX INDEX_NAME (id);
```
在person表为字段`name`添加索引，

``` sql
create index personIndex on Person(name(length));
```
删除索引

``` sql
DROP INDEX index_name ON table
```
_不同表中的索引名称可以相同。_
 
--------------------------------------------------------------------------
###### 博文原创，欢迎装载！
###### 转载请附上原文链接：http://xuhuiqiang.cn/2016/08/14/sqlBasic/
--------------------------------------------------------------------------
