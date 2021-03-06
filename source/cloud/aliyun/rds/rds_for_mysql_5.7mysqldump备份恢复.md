---
title: rds_for_mysql_5.7 mysqldump备份恢复
---

> RDS for MySQL Mysqldump 常见问题和处理https://help.aliyun.com/knowledge_detail/41732.html?spm=5176.11065259.1996646101.searchclickresult.265d1a5eWljLy8


```bash
[root@sh_01 ~]# booboo < ways_datafull_201808151223_1808151223.sql
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1840 (HY000) at line 24: @@GLOBAL.GTID_PURGED can only be set when @@GLOBAL.GTID_EXECUTED is empty.


[root@sh_01 ~]# grep DEFINER ways_datafull_201808151223_1808151223.sql 
CREATE DEFINER=`taoh`@`%` FUNCTION `find_in_area`(targetStr varchar(2000), findStr varchar(2000)) RETURNS varchar(1000) CHARSET utf8
[root@sh_01 ~]#  awk '{ if (index($0,"GTID_PURGED")) { getline; while (length($0) > 0) { getline; } } else { print $0 } }' ways_datafull_201808151223_1808151223.sql | grep -iv 'set @@' > your_revised.sql


grep DEFINER ways_datafull_201808151223_1808151223.sql 
awk '{ if (index($0,"GTID_PURGED")) { getline; while (length($0) > 0) { getline; } } else { print $0 } }' ways_datafull_201808151223_1808151223.sql | grep -iv 'set @@' > your_revised.sql
```
 

## mysqldump 常用demo

```sql
# 导出库 jacky 下的 teacher 表，包括表上的触发器，导出字符集是 utf8mb4
mysqldump —no-defaults -hxxx.mysql.aliyun.com -uuser_name -P3306 -ppass_word —set-gtid-purged=off —default-character-set=utf8mb4 —hex-blob —single-transaction —result-file=jacky_teacher.sql jacky teacher
# -p 选项指定密码，密码和选项间不要有空格
# -P 选项指定实例的端口
# -h 选项指定 RDS 实例的 URL 地址
# -u 选项指定使用的数据库用户
# 也可以使用下面的方式进行导出
mysqldump —no-defaults -hxxx.mysql.aliyun.com -uuser_name -P3306 -ppass_word —set-gtid-purged=off —default-character-set=utf8mb4 —hex-blob —single-transaction jacky teacher > jacky_teacher.sql

导出库 jacky，包括存储过程和函数，不包括 lock tables 和 unlock tables 语句
mysqldump —no-defaults -hxxx.mysql.rds.aliyuncs.com -uuser_name -ppass_word -P3306 —set-gtid-purged=off —hex-blob —single-transaction —routines —skip-add-locks —result-file=jacky.sql jacky
# —routines — 导出库涉及的存储过程和函数
# —skip-add-locks — 输出中不包括 lock tables table_name write; 和 unlock tables; 语句

# 导出库 jacky，包括存储过程、函数、触发器、事件，不包括数据
mysqldump —no-defaults -hxxx.mysql.rds.aliyuncs.com -uuser_name -ppass_word -P3306 —set-gtid-purged=off —hex-blob —single-transaction —routines —events —no-data —result-file=jacky.sql jacky
# 触发器选项 —triggers 默认开启，因此不需要指定
# —events — 导出库涉及的定时事件（计划任务）
# —no-data — 不导出数据

# 导出库 jacky，不包括 库、表创建语句，不包括 drop table 语句
mysqldump —no-defaults -hxxx.mysql.rds.aliyuncs.com -uuser_name -ppass_word -P3306 —set-gtid-purged=off —hex-blob —single-transaction —no-create-db —no-create-info —skip-add-drop-table —result-file=jacky.sql jacky
# —no-create-db  — 输出中不包括库的创建语句
# —no-create-info — 输出中不包括表的创建语句
# —skip-add-drop-table — 输出中不包括表的删除语句

# 导出库 jacky，jerry，jason，不包括表 jacky.teacher, jason.orders, jerry.sales
mysqldump —no-defaults -hxxx.mysql.aliyun.com -uuser_name -P3306 -ppass_word —set-gtid-purged=off —hex-blob —single-transaction —result-file=jacky_jerry_jason.sql —ignore-table=jacky.teacher —ignore-table=jason.orders —ignore-table=jerry.sales —databases jacky jerry jason
# —ignore-table — 指定不进行导出的表
# —databases — 指定要进行导出的数据库名称

# 导出库 jacky，包括存储过程和函数，尽量兼容 SQL SERVER 语法
mysqldump —no-defaults -hxxx.mysql.aliyun.com -uuser_name -P3306 -ppass_word —set-gtid-purged=off —compatible=mssql —routines —hex-blob —single-transaction —result-file=jacky_mssql.sql jacky
# —compatible=mssql — 增加对 SQL SERVER 的语法兼容性
```
