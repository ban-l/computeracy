# LIMIT

在 MySQL 中，`LIMIT` 子句用于限制查询结果返回的行数。

它通常用于分页查询或仅获取部分结果。以下是 `LIMIT` 子句的使用示例：

### 1. 基本用法

```sql
SELECT * FROM employees LIMIT 10;
```

- 该查询会返回 `employees` 表中的前 10 行。

### 2. 指定偏移量

```sql
SELECT * FROM employees LIMIT 5, 10;
```

- 该查询会从第 6 行开始（偏移量为 5），返回接下来的 10 行。
- 第一个参数 `5` 是偏移量，表示跳过前 5 行。
- 第二个参数 `10` 是返回的行数。

### 3. 分页查询

假设每页显示 20 条记录，查询第 3 页的数据：

```sql
SELECT * FROM employees LIMIT 40, 20;
```

- 偏移量计算公式：`(page_number - 1) * page_size`
- 这里 `(3 - 1) * 20 = 40`，所以偏移量为 40。
- 返回第 41 到 60 行的数据。

### 4. 结合 `ORDER BY` 使用

```sql
SELECT * FROM employees ORDER BY salary DESC LIMIT 5;
```

- 该查询会返回 `employees` 表中工资最高的 5 名员工。

### 5. 仅获取前 N 行

```sql
SELECT * FROM employees LIMIT 3;
```

- 该查询会返回 `employees` 表中的前 3 行。

### 6. 使用 `LIMIT` 和 `WHERE` 子句

```sql
SELECT * FROM employees WHERE department = 'Sales' LIMIT 5;
```

- 该查询会返回 `employees` 表中部门为 `Sales` 的前 5 名员工。

### 7. 使用 `LIMIT` 和 `OFFSET` 关键字

```sql
SELECT * FROM employees LIMIT 10 OFFSET 20;
```

- 该查询会从第 21 行开始（偏移量为 20），返回接下来的 10 行。
- `OFFSET 20` 表示跳过前 20 行。
- `LIMIT 10` 表示返回 10 行。

### 总结

- `LIMIT` 子句用于限制查询结果的行数。
- 可以单独使用 `LIMIT`，也可以结合 `OFFSET` 使用。
- 常用于分页查询或仅获取部分结果。