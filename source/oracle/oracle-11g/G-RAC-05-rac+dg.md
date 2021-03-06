---
title: RAC
---

```bash
1.集群数据库要归档
SQL> archive log list

在所有节点都需要正常停库
node1 & node2
SQL> shutdown immediate
在node1启动数据库到mount状态
SQL> startup mount
SQL> alter database archivelog;
SQL> alter database open;
SQL> archive log list
在node2打开数据库
SQL> startup

2.修改rac的本地存档参数
alter system set log_archive_dest_1='location=use_db_recovery_file_dest sync affirm valid_for=(online_logfiles,primary_role) db_unique_name=racdb';

alter system set log_archive_dest_state_1='enable';

3.启用远程归档
alter system set log_archive_dest_2='service=aux1srv valid_for=(online_logfiles,primary_role) db_unique_name=aux1';

alter system set log_archive_dest_state_2='enable';

4.在rac打开force logging
alter database force logging;

5.在rac打开dataguard开关
alter system set log_archive_config='dg_config=(racdb,aux1)';

6.为从库准备口令文件
scp orapwracdb2 oracle@172.25.0.10:$ORACLE_HOME/dbs/orapwaux1

7.为从库准备参数
vi $ORACLE_HOME/dbs/initaux1.ora
--------------------------------------------------------------------------------------
*.audit_file_dest='/u01/app/oracle/admin/aux1/adump'
*.compatible='11.2.0.4.0'
*.control_files='/u01/app/oracle/oradata/aux1/control01.ctl'
*.db_block_size=8192
*.db_domain=''
*.db_name='racdb'
*.db_recovery_file_dest='/u01/app/oracle/fast_recovery_area'
*.db_recovery_file_dest_size=4621074432
*.diagnostic_dest='/u01/app/oracle'
*.log_archive_config='dg_config=(racdb,aux1)'
*.memory_target=842006528
*.open_cursors=300
*.processes=150
*.remote_login_passwordfile='exclusive'
undo_tablespace='UNDOTBS1'
db_unique_name=aux1
log_archive_dest_3='location=/home/oracle/arc_aux1_dest3/ valid_for=(standby_logfiles,standby_role) db_unique_name=aux1'
standby_file_management=auto
db_file_name_convert='+DB/racdb/datafile/','/u01/app/oracle/oradata/aux1/','+DB/racdb/tempfile/','/u01/app/oracle/oradata/aux1/'
log_file_name_convert='+DB/racdb/onlinelog/','/u01/app/oracle/oradata/aux1/'
--------------------------------------------------------------------------------------
8.创建相关目录
mkdir -p /u01/app/oracle/admin/aux1/adump
mkdir -p /u01/app/oracle/oradata/aux1
mkdir -p /u01/app/oracle/fast_recovery_area
mkdir -p /home/oracle/arc_aux1_dest3

9.使用spfile启动实例到nomount
export ORACLE_SID=aux1
sqlplus / as sysdba
create spfile from pfile;
startup nomount

10.在从库配置监听程序，并启动监听
vi $ORACLE_HOME/network/admin/listener.ora
--------------------------------------------------------------------------------------
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = aux1)
      (ORACLE_HOME = /u01/app/oracle/product/11.2.0/db_1)
      (SID_NAME = aux1)
    )
  )

LISTENER =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.25.0.10)(PORT = 1521))
  )
--------------------------------------------------------------------------------------
##############
lsnrctl start

11.在rac（node1 & node2）配置连接从库的服务命名：
vi $ORACLE_HOME/network/admin/tnsnames.ora
--------------------------------------------------------------------------------------
aux1srv =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.25.0.10)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = aux1)
    )
  )
--------------------------------------------------------------------------------------
12.在rac（node1 & node2）测试与从库的网络连接：
sqlplus sys/Oracle11g@aux1srv as sysdba

13.在node1启动rman复制从库
rman target / auxiliary sys/Oracle11g@aux1srv
RMAN> duplicate target database for standby from active database;

14.在从库增加standby log
alter database add standby logfile '/u01/app/oracle/oradata/aux1/redo07.log' size 50m;
alter database add standby logfile '/u01/app/oracle/oradata/aux1/redo08.log' size 50m;

15.打开从库
SQL> alter database recover managed standby database disconnect from session;
SQL> alter database recover managed standby database cancel;
SQL> alter database open;

16.在rac(node1 & node2)切换日志
select thread#,sequence#,status from v$log where thread#=1;
select thread#,sequence#,status from v$log where thread#=2;

17.在从库验证归档出现
SQL> select thread#,sequence#,applied from v$archived_log;

18.在从库启动日志应用
SQL> alter database recover managed standby database using current logfile disconnect from session;

19.在从库查看日志应用状态
SQL> select thread#,sequence#,applied from v$archived_log;
```

