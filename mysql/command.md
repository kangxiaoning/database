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
```