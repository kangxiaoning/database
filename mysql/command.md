常用命令
======

# 1. 参数

```
--查询自动提交设置
show variables like 'AUTOCOMMIT'

--设置自动提交
SET AUTOCOMMIT = 1;

--查看权限
show grants for username@xxx.xxx.xxx.xxx;

--client设置字符集
set names utf8;

--添加主键
alter table table_name add primary key(col_a,col_b,col_c);

--timestamp提取日期
select count(*) from table_name where date(reporttime)='2017-11-15';

--查询db size
SELECT table_schema "DB Name",
        ROUND(SUM(data_length + index_length) / 1024 / 1024, 1) "DB Size in MB" 
FROM information_schema.tables 
GROUP BY table_schema;

--查询table size
SELECT TABLE_NAME "Table Name",
        ROUND((data_length + index_length) / 1024 / 1024, 1) "Table Size in MB" 
FROM information_schema.tables 
where table_schema="videocommunity"
order by 2 desc;
```