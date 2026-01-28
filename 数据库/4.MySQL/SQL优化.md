# SQL优化

## **1. limit分页优化**

当偏移量特别大时，limit效率会非常低。结合`order by`使用。

```sql
select id from A order by id limit 90000,10;
select id from A order by id  between 90000 and 90010;
```

## **2.利用limit 1 、top 1 取得一行**

有些业务逻辑进行查询操作时（如某一字段`DESC`，取最大)

可以使用`limit 1`或者`top 1`来终止[数据库索引]继续扫描整个表或索引。

```sql
SELECT id FROM A LIKE 'abc%' limit 1;
```

## **3. 任何情况都不要用 select \*  ，用具体的字段列表替换"\*"**

## **4. 批量插入优化**

```sql
INSERT into person(name,age) values
('A',24),
('B',24),
('C',24),
...
```

## **5.like语句的优化**

反例：`SELECT id FROM A WHERE name like '%abc%';`

正例：`SELECT id FROM A WHERE name like 'abc%';`

由于abc前面用了`%`，该查询走**全表查询**，除非必要，否则不要在关键词前加`%`

## **6.where使用or的优化**

通常使用`union all` 或 `union`替换`or`会得到更好的效果，`where`中使用了`or`，导致**索引失效**。

反例：`SELECT id FROM A WHERE num = 10 or num = 20;`

正例：`SELECT id FROM A WHERE num = 10 union all SELECT id FROM A WHERE num=20;`

## **7.where中使用 IS NULL 或 IS NOT NULL 的优化**

`where`中使用`IS NULL` 或`IS NOT NULL` 判断，导致**索引失效**，全表查询。

**优化成num上设置默认值0**，确保表中num没有null值，`IS NULL`使用率极高，应注意**避免全表扫描。**

```sql
SELECT id FROM A WHERE num=0;
```

## **8.where中对表达式操作的优化**

不要在`where`中的`=`左边，进行函数、算数或其他表达式运算，会导致**索引失效**。

反例：`SELECT id FROM A WHERE datediff(day,createdate,'2019-11-30')=0;`

**正例：**`SELECT id FROM A WHERE createdate>='2019-11-30' and createdate<'2019-12-1';`

反例：`SELECT id FROM A WHERE year(addate) <2020;`

**正例：**`SELECT id FROM A where addate<'2020-01-01'`

## **9. 尽量用 union all 替换 union**

`union`和`union all`差异：`union`将结果集合并后再进行唯一性过滤，这会加大资源消耗及延迟。 不在乎重复结果集时，尽量使用`union all`而不是`union`。

## **10.Inner join 和 left join、right join、子查询**

推荐使用`inner join`连接

- `SELECT A.id,A.name,B.id,B.name FROM A LEFT JOIN B ON A.id =B.id;`
- `SELECT A.id,A.name,B.id,B.name FROM A,B WHERE A.id = B.id;`

尽量用**`join`来替换子查询**

- 反例：`Select* from A where exists (select * from B where id>=3000 and A.uuid=B.uuid);`
- 正例：`Select* from A inner join B ON A.uuid=B.uuid where b.uuid>=3000;`

使用`JOIN`时候，用小结果驱动大结果。

- `left join` 左边表结果尽量小，如果有条件应该放到左边先处理，`right join`同理反向。

## **11.联合索引的最左前缀原则**

在复合索引中，MySQL 只能使用索引的最左前缀部分。

例如：

```sql
CREATE INDEX idx_name_age ON employees(name, age);
SELECT * FROM employees WHERE name = 'John';
```

这里 MySQL 会使用 `idx_name_age` 索引，因为 `name` 是索引的最左前缀。

但如果查询条件是：

```sql
SELECT * FROM employees WHERE age = 30;
```

MySQL 不会使用 `idx_name_age` 索引，因为 `age` 不是索引的最左前缀。