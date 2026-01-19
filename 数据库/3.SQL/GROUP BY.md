# GROUP BY

分组，把具有相同数据值的行 ，放在同一组中。

### 1. **非聚合列必须包含在GROUP BY中**

- **严格模式限制**：若启用`ONLY_FULL_GROUP_BY`模式（MySQL 5.7+默认），`SELECT`中的非聚合列（如未使用`SUM`、`COUNT`等）必须出现在`GROUP BY`子句中，否则报错。
- **非严格模式风险**：未启用时，MySQL可能返回任意值，导致数据不一致。

**示例错误**：

```sql
SELECT name, age, SUM(sales) FROM orders GROUP BY name;
-- age未在GROUP BY中，严格模式下报错。
```

------

### 2. **正确放置子句顺序**

确保子句顺序为：

`WHERE` → `GROUP BY` → `HAVING` → `ORDER BY` → `LIMIT`

**错误示例**：

```sql
SELECT department, COUNT(*) FROM employees
GROUP BY department
WHERE hire_date > '2020-01-01'; -- WHERE在GROUP BY后，语法错误！
```

------

### 3. **性能优化与索引**

- **索引优化**：为`GROUP BY`的列添加索引（尤其是多列分组时），避免全表扫描和临时表排序。
- **避免复杂表达式**：如`GROUP BY YEAR(date_column)`可能导致索引失效，可考虑生成列或预处理。

------

### 4. **HAVING vs. WHERE**

- **WHERE**：在分组前过滤**行**，**不可使用聚合函数**。
- GROUP BY
- **HAVING**：在分组后过滤**组**，**可使用聚合函数**。

**示例**：

```sql
SELECT department, AVG(salary)
FROM employees
WHERE salary > 3000  -- 分组前过滤行
GROUP BY department
HAVING AVG(salary) > 5000;  -- 分组后过滤组
```

------

### 5. **表达式与函数**

允许使用表达式或函数分组，但可能影响性能：

```sql
SELECT DATE(order_date), SUM(amount)
FROM orders
GROUP BY DATE(order_date);  -- 按日期分组
```

------

### 6. **WITH ROLLUP 的汇总行**

`WITH ROLLUP`生成分层汇总，但会产生`NULL`值表示汇总行：

```sql
SELECT department, role, SUM(sales)
FROM employees
GROUP BY department, role WITH ROLLUP;
-- 使用COALESCE(department, '总计')处理NULL
```

------

### 7. **列别名与执行顺序**

`GROUP BY`可使用`SELECT`中的别名（因执行顺序在`SELECT`之后）：

```sql
SELECT YEAR(date) AS year, COUNT(*)
FROM events
GROUP BY year;  -- 使用别名year
```

------

### 8. **多列分组的顺序**

分组顺序影响结果，`GROUP BY col1, col2`与`col2, col1`分组逻辑不同。

------

### 9. **NULL值的处理**

所有`NULL`值会被归为同一组，需注意数据逻辑。

------

### 10. **JOIN后的GROUP BY**

确保连接后的数据唯一性，避免错误聚合：

```sql
SELECT users.id, users.name, SUM(orders.amount)
FROM users
LEFT JOIN orders ON users.id = orders.user_id
GROUP BY users.id;  -- 正确：按用户ID分组
```

------

### 11. **SQL模式兼容性**

- 检查`sql_mode`是否包含`ONLY_FULL_GROUP_BY`，避免环境差异导致错误。

- 通过以下命令查看模式：

  ```sql
  SELECT @@sql_mode;
  ```

------

### 12. **DISTINCT与GROUP BY的选择**

- 仅需去重时，`DISTINCT`可能更高效。
- 需要聚合计算时，使用`GROUP BY`。