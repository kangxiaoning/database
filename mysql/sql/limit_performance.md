limit 性能问题
=============

# 1. 问题

limit 是常用的分页功能，然而使用不当，页数的增加会是查询越来越慢。

参考[MySQL ORDER BY / LIMIT performance: late row lookups](https://explainextended.com/2009/10/23/mysql-order-by-limit-performance-late-row-lookups/)


# 2. 解决方案

解决方法是改写SQL。

## 2.1 原始SQL

```sql
SELECT *
FROM news
WHERE cat_id = 4
ORDER BY id DESC
LIMIT 150000, 10;
```
## 2.2 改写SQL

有where条件的，把where子句也写到子查询里即可。

```sql
SELECT l.id, value, LENGTH(stuffing) AS len
FROM (
	SELECT  id
	FROM    t_limit
	ORDER BY id
	LIMIT 150000, 10
	) o
JOIN t_limit l
ON l.id = o.id
ORDER BY l.id;
```
