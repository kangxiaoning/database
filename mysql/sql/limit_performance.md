limit 性能问题
=============

# 1. 问题

limit 是常用的分页功能，然而使用不当，页数的增加会使查询越来越慢，例如`limit 150000, 10`
比`limit 20,10`慢很多。

This task can be reformulated like this: take the last 150,010 rows in id order
and return the first 10 of them

参考 [MySQL ORDER BY / LIMIT performance: late row lookups](https://explainextended.com/2009/10/23/mysql-order-by-limit-performance-late-row-lookups/)

【总结】

1. 基础

- **index key**: 创建index的字段。
- **table pointer**: 唯一标识一行的值（例如primary key）。
- **row lookup**: 通过index利用table pointer获取对应table数据的过程。 
- **late row lookup**: 先从index提取需要的index key，再对这些index key执行row lookup。

2. 问题原因

对不需要的数据进行了row lookup，过程如下。

- 对limit子句的offset + row_numbers行都执行了row lookup
- 丢弃offset行，返回row_numbers行
- 这里可以看出随着offset增大，这个操作的耗时会越来越长，时间复杂度O(n)

例如`ORDER BY id DESC LIMIT 150000, 10`相当于执行**150010**次row lookup，然后丢弃
150000行，返回10行。

# 2. 解决方案

解决方法是减少不必要的row lookup，这是cost比较高的操作，并且前面执行了row lookup后面又
丢弃，做了很多无用功。

可以按如下步骤操作，这个过程也称为**late row lookup**。

- 通过index过滤出需要的table pointer
- 只对这些table pointer执行row lookup
- 由于第一步时间复杂度是O(log<sup>n</sup>)，第二步基本是O(1)，因此整个操作的时间复杂度也是
O(log<sup>n</sup>)

这样就避免了做无用功，既然不需要这些行，那也就不需要对这些行执行row lookup了。

## 2.1 原始SQL

```sql
SELECT *
FROM news
WHERE cat_id = 4
ORDER BY id DESC
LIMIT 150000, 10;
```
## 2.2 改写SQL

通过改写sql，在row lookup前过滤出有效table pointer，有where条件的，把where子句也写到子查询里即可。

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
