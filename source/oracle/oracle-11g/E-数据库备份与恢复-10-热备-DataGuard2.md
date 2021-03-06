---
title: DataGuard
---

```bash
在从中进行不完全恢复，找回主库中的误操作：
SQL> alter database recover managed standby database disconnect from session until change 1298235;

将数据exp/imp

从库继续向前recover
SQL> alter database recover managed standby database disconnect from session;

查看从库中归档日志的已用情况：
SQL> select sequence#,applied from v$archived_log;

关闭从库的管理恢复
SQL> alter database recover managed standby database cancel;

在从库中启用延迟恢复
SQL> alter database recover managed standby database disconnect from session delay 360;
##########################################################################################
主库传输日志的风格和从库接收日志的风格
SQL> select ASYNC_BLOCKS,AFFIRM from v$archive_dest where DEST_ID=2;

ASYNC_BLOCKS AFF
------------ ---
       61440 NO
网络IO异步；磁盘IO异步

修改主库向从库传输日志的风格：网络IO同步；磁盘IO同步
alter system set log_archive_dest_2='service=aux1srv sync affirm valid_for=(online_logfiles,primary_role) db_unique_name=aux1';
##########################################################################################
查看主库的保护模式（属性）和保护级别（即时状态）
SQL> select database_role,protection_mode,protection_level from v$database;

DATABASE_ROLE	 PROTECTION_MODE      PROTECTION_LEVEL
---------------- -------------------- --------------------
PRIMARY 	 MAXIMUM PERFORMANCE  MAXIMUM PERFORMANCE

修改主库的保护模式
ALTER DATABASE SET STANDBY DATABASE TO MAXIMIZE {AVAILABILITY | PERFORMANCE | PROTECTION};

最高可用模式
ALTER DATABASE SET STANDBY DATABASE TO MAXIMIZE AVAILABILITY;

SQL> alter system switch logfile;
SQL> alter system checkpoint;

SQL> select database_role,protection_mode,protection_level from v$database;

DATABASE_ROLE	 PROTECTION_MODE      PROTECTION_LEVEL
---------------- -------------------- --------------------
PRIMARY 	 MAXIMUM AVAILABILITY MAXIMUM AVAILABILITY

最大保护模式：
ALTER DATABASE SET STANDBY DATABASE TO MAXIMIZE PROTECTION;

SQL> select database_role,protection_mode,protection_level from v$database;

DATABASE_ROLE	 PROTECTION_MODE      PROTECTION_LEVEL
---------------- -------------------- --------------------
PRIMARY 	 MAXIMUM PROTECTION   MAXIMUM PROTECTION
##########################################################################################
角色转换(switchover):主库先变从库；从库再变主库
为主变从做准备:
1.standby log
SQL> alter database add standby logfile '/u01/app/oracle/oradata/db01/redo04.log' size 52428800;
SQL> alter database add standby logfile '/u01/app/oracle/oradata/db01/redo05.log' size 52428800;
SQL> alter database add standby logfile '/u01/app/oracle/oradata/db01/redo06.log' size 52428800;

2.创建standby log归档路径
mkdir -p /home/oracle/arc_db01_dest3/

alter system set log_archive_dest_3='location=/home/oracle/arc_db01_dest3/ valid_for=(standby_logfiles,standby_role) db_unique_name=db01';

3.打开备库数据文件自动管理
SQL> alter system set standby_file_management=auto scope=spfile;

4.启动路径转换参数
alter system set db_file_name_convert='/u01/app/oracle/oradata/aux1/','/u01/app/oracle/oradata/db01/' scope=spfile;
alter system set log_file_name_convert='/u01/app/oracle/oradata/aux1/','/u01/app/oracle/oradata/db01/' scope=spfile;

5.配置监听程序
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
      (ADDRESS = (PROTOCOL = TCP)(HOST = 172.25.0.10)(PORT = 7788))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = cctv)
      (ORACLE_HOME = /u01/app/oracle/product/11.2.0/db_1)
      (SID_NAME = cctv)
    )
    (SID_DESC =
      (GLOBAL_DBNAME = db01)
      (ORACLE_HOME = /u01/app/oracle/product/11.2.0/db_1)
      (SID_NAME = db01)
    )
  )

查看角色转换状态：
SQL> select switchover_status from v$database;

SWITCHOVER_STATUS
--------------------
SESSIONS ACTIVE

开始角色转换：
ALTER DATABASE COMMIT TO SWITCHOVER TO PHYSICAL STANDBY WITH SESSION SHUTDOWN;
启动数据库：
SQL> startup
查看数据库角色：
SQL> select database_role,protection_mode,protection_level from v$database;

DATABASE_ROLE	 PROTECTION_MODE      PROTECTION_LEVEL
---------------- -------------------- --------------------
PHYSICAL STANDBY MAXIMUM PROTECTION   MAXIMUM PROTECTION
～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～～
为从变主做准备:
1.从库的online log要有归档位置
alter system set log_archive_dest_1='location=/home/oracle/arc_aux1_dest1/ valid_for=(online_logfiles,primary_role) db_unique_name=aux1';

2.从库要为主库远程归档
alter system set log_archive_dest_2='service=db01srv sync affirm valid_for=(online_logfiles,primary_role) db_unique_name=db01';

3.从库要配置服务命名
db01srv =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.25.0.10)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = db01)
    )
  )

4.查看角色转换状态
SQL> select switchover_status from v$database;

SWITCHOVER_STATUS
--------------------
SESSIONS ACTIVE

开始角色转换：
ALTER DATABASE COMMIT TO SWITCHOVER TO PRIMARY WITH SESSION SHUTDOWN;
打开数据库：
SQL> alter database open;
##########################################################################################
```
