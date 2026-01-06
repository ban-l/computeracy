# SQL

SQL（结构化查询语言）是一个功能丰富的语言，主要用于管理和操作关系型数据库。

它包含以下几个主要组成部分：

## **1. 数据定义语言（DDL - Data Definition Language）**
用于定义和修改数据库结构：
- **CREATE** - 创建数据库、表、索引等
- **ALTER** - 修改现有数据库对象
- **DROP** - 删除数据库对象
- **TRUNCATE** - 快速删除表中所有数据
- **RENAME** - 重命名对象

## **2. 数据操作语言（DML - Data Manipulation Language）**
用于操作数据库中的数据：
- **SELECT** - 查询数据
- **INSERT** - 插入新数据
- **UPDATE** - 更新现有数据
- **DELETE** - 删除数据
- **MERGE/UPSERT** - 合并数据（插入或更新）

## **3. 数据控制语言（DCL - Data Control Language）**
用于权限控制：
- **GRANT** - 授予权限
- **REVOKE** - 撤销权限
- **DENY** - 拒绝权限（某些数据库）

## **4. 事务控制语言（TCL - Transaction Control Language）**
用于管理事务：
- **BEGIN/START TRANSACTION** - 开始事务
- **COMMIT** - 提交事务
- **ROLLBACK** - 回滚事务
- **SAVEPOINT** - 设置保存点

## **5. 数据查询语言（DQL - Data Query Language）**
主要就是 **SELECT** 语句及其各种子句：
- **FROM** - 指定数据源
- **WHERE** - 过滤条件
- **GROUP BY** - 分组
- **HAVING** - 分组后过滤
- **ORDER BY** - 排序
- **JOIN** - 连接多个表
- **UNION/INTERSECT/EXCEPT** - 集合操作

## **6. 其他重要元素**

**子句和操作符：**
- 比较运算符（=, <>, >, < 等）
- 逻辑运算符（AND, OR, NOT）
- 聚合函数（COUNT, SUM, AVG, MAX, MIN）
- 字符串函数、数学函数、日期函数等

**高级特性：**
- **视图（VIEW）** - 虚拟表
- **索引（INDEX）** - 提高查询性能
- **存储过程（Stored Procedures）**
- **触发器（Triggers）**
- **游标（Cursors）**
- **窗口函数（Window Functions）**
