---
title: DBA的日常工作
---

```bash

~~~~~~~~模拟pga无法分配造成ORA-04030异常~~~~~~~~~~
conn / as sysdba
定义对象类型
CREATE OR REPLACE TYPE pga_type AS OBJECT
(id NUMBER,
name char(2000)
);
/
定义表类型
create or replace type t_pga_table as table of pga_type;
/
向表类型变量持续填充数据
create or replace procedure proc_test_pga as
v_pga t_pga_table := t_pga_table();
begin
loop
v_pga.extend();
v_pga(v_pga.count) := pga_type(1, 'pga_test');
--dbms_output.put_line(v_pga(v_pga.count).name);
end loop;
end;
/

获得当前sys用户的sid
select sid from v$mystat where rownum=1;
sid
----
159

执行procedure

SQL>exec proc_test_pga;

ERROR at line 1:
ORA-04030: out of process memory when trying to allocate 16396 bytes (koh-kghu call ,pl/sql vc2)

select * from v$process_memory
where pid=
(select pid from v$process where addr=
(select paddr from v$session where sid=159));

PID SERIAL# CATEGORY ALLOCATED USED MAX_ALLOCATED
----- ---------- --------------- ---------- ---------- -------------
15 2 SQL 164240 123016 562608
15 2 PL/SQL 26312 21156 258324
15 2 Freeable 131072 0
15 2 Other 1185704 4294005349

```
