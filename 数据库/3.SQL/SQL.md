# SQL

SQL（Structured Query Language）是数据库领域的通用语言，主要用于管理和操作关系型数据库。

## 1. SQL 语言的五大支柱

| 分类    | 全称                         | 描述                           | 常用命令                              |
| ------- | ---------------------------- | ------------------------------ | ------------------------------------- |
| **DQL** | Data Query Language          | **数据查询**（最核心、最复杂） | `SELECT`                              |
| **DML** | Data Manipulation Language   | **数据操作**（增删改数据）     | `INSERT`, `UPDATE`, `DELETE`          |
| **DDL** | Data Definition Language     | **数据定义**（操作表结构/库）  | `CREATE`, `DROP`, `ALTER`, `TRUNCATE` |
| **TCL** | Transaction Control Language | **事务控制**（维护数据一致性） | `COMMIT`, `ROLLBACK`                  |
| **DCL** | Data Control Language        | **数据控制**（权限管理）       | `GRANT`, `REVOKE`                     |

---

## 2. DQL：查询语法详解 

数据查询语言（DQL - Data Query Language），查询是数据库日常工作占比 80% 的内容。

### 基本语法结构

一个完整的 `SELECT` 语句通常包含以下部分。

**请注意，SQL 的“书写顺序”和数据库引擎的“执行顺序”是完全不同的，理解这一点是进阶的关键。**

**A. 书写顺序 (你写的代码):**

```sql
SELECT DISTINCT <字段列表>
FROM <表名>
JOIN <关联表> ON <关联条件>
WHERE <过滤条件>
GROUP BY <分组字段>
HAVING <分组后过滤条件>
ORDER BY <排序字段>
LIMIT <分页/行数限制>;
```

**B. 执行顺序 (数据库的视角 - Expert Insight):**
数据库引擎并不是先看 `SELECT`，而是先看 `FROM` 也就是数据来源。

1. **FROM / JOIN**: 确定数据来源，生成虚拟表。
2. **WHERE**: 对原始数据进行行级过滤（**此时别名还不能用**）。
3. **GROUP BY**: 将数据划分为组。
4. **HAVING**: 对“分组后”的数据进行过滤（例如：筛选总分大于60的班级）。
5. **SELECT**: 提取需要的列 / 计算聚合函数。
6. **DISTINCT**: 去重。
7. **ORDER BY**: 排序（此时可以使用 `SELECT` 中的别名）。
8. **LIMIT**: 截取结果集。

### 关键语法点解析

* **连接查询 (JOIN)**
* `INNER JOIN`: **交集**。两张表都有匹配数据才返回。
* `LEFT JOIN`: **左侧全集**。左表所有数据都返回，右表没匹配的显示 `NULL`。
* `RIGHT JOIN`: **右侧全集**。
* `FULL JOIN`: **并集**（MySQL不直接支持，需用 `UNION` 模拟）。


* **聚合函数 (Aggregation)**
  常用于统计，必须配合 `GROUP BY` 使用（除非是统计全表）。
* `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `MIN()`

---

## 3. DML：数据操作语法

数据操作语言（DML - Data Manipulation Language），负责数据的“增删改”。

**提示：在生产环境中执行 `UPDATE` 或 `DELETE` 前，永远先写 `WHERE` 子句，甚至先用 `SELECT` 验证一下 `WHERE` 的范围。**

* **INSERT (新增)**

```sql
-- 标准写法
INSERT INTO users (id, name, age) VALUES (1, 'Alice', 25);

-- 批量插入 (性能更好)
INSERT INTO users (name, age) VALUES ('Bob', 30), ('Charlie', 22);

-- 从查询结果插入
INSERT INTO users_backup (name, age) SELECT name, age FROM users;
```


* **UPDATE (修改)**

```sql
UPDATE users 
SET age = 26, status = 'active' 
WHERE id = 1; -- 务必带上 WHERE！
```


* **DELETE (删除)**

```sql
DELETE FROM users 
WHERE status = 'banned'; 
```


> **注意**：`DELETE` 是逐行删除，记录日志，可回滚；`TRUNCATE` (属于DDL) 是直接重置表，速度快但不可回滚。

---

## 4. DDL：数据定义语法

数据定义语言（DDL - Data Definition Language），用于定义“骨架”。

* **CREATE **：创建数据库、表、索引等

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    amount DECIMAL(10, 2),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```


* **ALTER **：修改现有数据库对象

```sql
ALTER TABLE orders ADD COLUMN status VARCHAR(20);
ALTER TABLE orders MODIFY COLUMN amount DECIMAL(12, 2);

```

- **DROP**：删除数据库对象
- **TRUNCATE**：快速删除表中所有数据
- **RENAME**：重命名对象

---

## 5. DCL

数据控制语言（DCL - Data Control Language，用于权限控制：

- **GRANT** - 授予权限
- **REVOKE** - 撤销权限
- **DENY** - 拒绝权限（某些数据库）

## 6. TCL

事务控制语言（TCL - Transaction Control Language），用于管理事务：

- **BEGIN/START TRANSACTION** - 开始事务
- **COMMIT** - 提交事务
- **ROLLBACK** - 回滚事务
- **SAVEPOINT** - 设置保存点

## 7. 其他重要元素

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

## 8. 语法陷阱

1. **NULL 的处理**:

* `NULL` 不等于 `0`，也不等于空字符串 `''`。
* 不要使用 `= NULL`，必须使用 `IS NULL` 或 `IS NOT NULL`。
* 聚合函数（如 `COUNT(col)`）会自动忽略 `NULL` 值，但 `COUNT(*)` 会统计包含 `NULL` 的行。


2. **字符串引用**:

* SQL 标准规定字符串使用单引号 `'value'`。虽然部分数据库（如MySQL）允许双引号，但建议养成用单引号的好习惯，双引号通常用于引用字段名（如 `"User Name"`）。


3. **通配符性能**:

* `LIKE '%abc'`（前置百分号）会导致索引失效，查询变成全表扫描，性能极差。尽量使用 `LIKE 'abc%'`。
