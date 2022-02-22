---
title: Hive常用命令汇总
date: 2021-09-30 11:19:56
index_img: /img/0a6e3ff0d40e42bc581c88b6cada856825ea325br1-512-512v2_uhq.jpeg
banner_img: /img/Getting-started-with-apache-hive.jpeg
tags: [Hive,大数据]
---

### **Hive数据库操作命令**

#### **查看所有数据库**

```sql
show databases;	
```

<!-- more -->

#### **使用数据库`default`**

```sql
use default;
```

#### **查看数据库信息**

```sql
describe database default;
```

#### **显示当前使用的数据库名称**

```sql
set hive.cli.print.current.db=true;
```

### **创建数据库命令**

```sql
create database example;
```

#### **删除数据库**

删除数据库的时候，不允许删除有数据的数据库，如果数据库里面有数据则会报错。如果要忽略这些内容，则在后面增加`CASCADE`关键字，则忽略报错，删除数据库。

```sql
DROP DATABASE DbName CASCADE(可选);
DROP DATABASE IF EXISTS db_name CASCADE;
```

#### **查看表**

模糊搜索表，其中‘name’可以为正则表达式

```sql
show tables like '*name*';
```

查看已知数据库中的表格

```sql
SHOW TABLES IN DbName;
```

查看表结构信息

```sql
desc formatted table_name;  
desc table_name; 
```

查看分区信息

```sql
show partitions table_name;
```

查看hdfs文件信息

```sql
dfs -ls /user/hive/warehouse/table02;
```

#### **创建表**

创建表完整格式

```sql
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [COMMENT col_comment], ...)]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
```

创建内部表

```sql
create table test_table(
name string comment 'name value',
addr string comment 'addr value'
);
```

创建外部表只需增加`external`关键字

```sql
create table external test_table(
name string comment 'name value',
addr string comment 'addr value'
);
```

创建表格并检查是否冲突

```sql
create table if not exists test_table(
name string comment 'name value',
addr string comment 'addr value'
)
```

**有一个表，创建另一个表**

只是复制了表结构，并不会复制内容

```sql
create table test3 like test2;
```

不需要执行`mapreduce`

**从其他表查询，再创建表**

复制表结构的同时，把`select`内容也复制过来了

```sql
create table test2 as select name,addr from test1;
```

需要执行`mapreduce`

#### **查找表数据**

```sql
select * from employees;
```

#### **命令行中显示列名字段**

```sql
set hive.cli.print.header=true;
```

#### **删除表**

内部表删除，会连同hdfs存储的数据一同删除，而外部表删除，只会删除外部表的元数据信息。

```sql
drop table test_table;
```

