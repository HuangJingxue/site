---
title: RAC
---

```bash
##############################################################
# ????shell??
##############################################################

vi /etc/security/limits.conf
------------------------------------------
#grid & oracle configure shell parameters
grid soft nofile 65536
grid hard nofile 65536
grid soft nproc 16384
grid hard nproc 16384

oracle soft nofile 65536
oracle hard nofile 65536
oracle soft nproc 16384
oracle hard nproc 16384
------------------------------------------
##############################################################
# ????????
##############################################################
vi /etc/sysctl.conf
------------------------------------------
kernel.shmmax = 4294967296
kernel.shmmni = 4096
kernel.shmall = 2097152
kernel.sem = 250 32000 100 128
fs.file-max = 6815744
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
------------------------------------------
???????
/sbin/sysctl -p

##############################################################
#  ??yum?
##############################################################
mkdir /mnt/centos671
mkdir /mnt/centos672
mount -o loop /dev/cdrom /mnt/centos671
mount -o loop /dev/cdrom1 /mnt/centos672
cd /etc/yum.repos.d
mv CentOS-Base.repo CentOS-Base.repo.old

vi CentOS-Base.repo
----------------------------------------------------------
[c6-media]
name=CentOS-6 - Media
baseurl=file:///mnt/centos671
        file:///mnt/centos672
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-Debug-6
enabled=1
----------------------------------------------------------

yum clean all
yum repolist
yum makecache

yum -y --nogpgcheck install libXp
yum -y --nogpgcheck install iscsi-initiator-utils
yum -y --nogpgcheck install gcc
*yum -y --nogpgcheck install compat-libstdc++
yum -y --nogpgcheck install elfutils-libelf-devel
yum -y --nogpgcheck install gcc-c++
yum -y --nogpgcheck install libaio-devel
*yum -y --nogpgcheck install listdc++-devel
*yum -y --nogpgcheck install pdksh

yum -y install elfutils-libelf-devel*
yum -y install compat*

##############################################################
# ??????SELinux
##############################################################
/etc/init.d/iptables status
/etc/init.d/iptables stop
chkconfig --list iptables
chkconfig iptables off

vi /etc/selinux/config
SELINUX=disable

##############################################################
# ??DNS???????????????????????
##############################################################
?/etc/hosts?????????????DNS????
[root@rac1 ~]# yum -y --nogpgcheck install bind
1???named.conf???
[root@rac1 ~]#  vim /etc/named.conf
------------------------------------------------------
[root@asm ~]# cat /etc/named.conf
options {
	listen-on port 53 { any; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { any; };
            };
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
                            };
                };
zone "." IN {
	type hint;
//	file "named.ca";
	file "/dev/null";
                    };
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

2???zone????

[root@rac1 ~]#  vim /etc/named.rfc1912.zones
--------------------------------------------------------
zone "oracle.com" IN {
        type master;
        file "oracle.com.zone";
        allow-update { none; };
};
zone "10.0.136.in-addr.arpa" IN {
        type master;
        file "10.0.136.zone";
        allow-update { none; };
};

3???zone???????
[root@rac1 ~]#  cd /var/named/
[root@rac1 ~]#  cp -p named.localhost oracle.com.zone
[root@rac1 ~]#  cp -p named.localhost 10.0.136.zone
[root@rac1 ~]#  vim /var/named/oracle.com.zone
---------------------------------------------------------------
$TTL 1D
@	IN	SOA	@ oracle.com. (
                                         0	; serial
                                         1D	; refresh
                                         1H	; retry
                                         1W	; expiry
                                         3H )	; minimum
	NS	@
	A	136.0.10.143
rac1	A	136.0.10.143
rac2	A	136.0.10.144
scan	A	136.0.10.211
scan	A	136.0.10.212
scan	A	136.0.10.213

[root@rac1 ~]#  vim /var/named/10.0.136.zone
---------------------------------------------------------------
$TTL    1D
@	IN	SOA	@ oracle.com.  (
                                       0	; Serial
                                       1D	; Refresh
                                       1H	; Retry
                                       1W	; Expire
                                       3H	)    ; Minimum
	NS	@
	A	136.0.10.143
143	PTR	rac1.oracle.com.
144	PTR	rac2.oracle.com.
211	PTR	scan.oracle.com.
212	PTR	scan.oracle.com.
213	PTR	scan.oracle.com.

4????copy????????????
[root@rac1 ~]# cd /var/named/
[root@rac1 named]# chown root.named oracle.com.zone 10.0.136.zone
5???named???
[root@rac1 etc]# service named restart
stopping named?                                             OK
starting named?                                               OK

[root@rac1 etc]# chkconfig named on

6????????DNS?
[root@rac1 ~]# vi /etc/resolv.conf
--------------------------------
search oracle.com
nameserver 136.0.10.11
options rotate
options timeout:2
options attempts:5

##############################################################
# ????IP?
##############################################################
/etc/hosts
-------------------------------------------------------
# Do not remove the following line, or various programs
# that require network functionality will fail.
127.0.0.1       localhost.localdomain localhost
::1             localhost6.localdomain6 localhost6

#public
136.0.10.11 centos1
136.0.10.12 centos2
#priv
172.0.10.11 centos1-priv
172.0.10.12 centos2-priv
#vip
136.0.10.13 centos1-vip
136.0.10.14 centos2-vip
###############################################################################
# ???
###############################################################################
groupadd -g 501 oinstall
groupadd -g 502 dba
groupadd -g 503 oper
groupadd -g 504 asmadmin
groupadd -g 505 asmdba
groupadd -g 506 asmoper

###############################################################################
# ????
###############################################################################
useradd -u 502 -g oinstall -G dba,asmadmin,asmdba,asmoper grid
useradd -u 501 -g oinstall -G dba,oper,asmdba,asmadmin oracle

###############################################################################
# ??????
###############################################################################
passwd grid
passwd oracle

###############################################################################
# ??grid??????
###############################################################################
vi .bashrc
----------------------------------------------------------
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/grid
export ORACLE_OWNER=oracle
export ORACLE_SID=+ASM1 #rac2???ORACLE_SID=+ASM2
export ORACLE_TERM=vt100
export THREADS_FLAG=native
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
export LANG=en_US
alias sqlplus='rlwrap sqlplus'
alias lsnrctl='rlwrap lsnrctl'
alias asmcmd='rlwrap asmcmd'
----------------------------------------------------------
###############################################################################
# ??oracle??????
###############################################################################
vi .bashrc
----------------------------------------------------------
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1
export ORACLE_OWNER=oracle
export ORACLE_SID=orcl1 #rac2???ORACLE_SID=orcl2
export ORACLE_TERM=vt100
export THREADS_FLAG=native
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
export EDITOR=vi
export SQLPATH=/home/oracle
export LANG=en_US
alias sqlplus='rlwrap sqlplus'
alias lsnrctl='rlwrap lsnrctl'
alias rman='rlwrap rman'
alias dgmgrl='rlwrap dgmgrl'
----------------------------------------------------------
###############################################################################
# ???????
###############################################################################
mkdir -p /u01/grid
chown -R grid:oinstall /u01/grid
mkdir -p /u01/app/oracle
chown -R oracle:oinstall /u01/app/
chmod -R 775 /u01/
###############################################################################
# ??grid???????
###############################################################################
su - grid
ssh-keygen -t rsa
ssh-keygen -t dsa
cd .ssh
cat *.pub > authorized_keys

scp authorized_keys grid@136.0.10.12:/home/grid/.ssh/keys_dbs
cat keys_dbs >> authorized_keys
scp authorized_keys grid@136.0.10.11:/home/grid/.ssh/

ssh centos1 date
ssh centos2 date
ssh centos1-priv date
ssh centos2-priv date
--------------------------------------------------------------
###############################################################################
# ??oracle???????
###############################################################################
su - oracle
ssh-keygen -t rsa
ssh-keygen -t dsa
cd .ssh
cat *.pub > authorized_keys

scp authorized_keys oracle@136.0.10.12:/home/oracle/.ssh/keys_dbs
cat keys_dbs >> authorized_keys
scp authorized_keys oracle@136.0.10.11:/home/oracle/.ssh/

ssh centos1 date
ssh centos2 date
ssh centos1-priv date
ssh centos2-priv date

iscsiadm -m discovery -t sendtargets -p 136.0.10.15 -l

?rac1?????????rawdevice:
vi /etc/udev/rules.d/60-raw.rules
---------------------------------------------------------------
ACTION=="add", KERNEL=="sdb1", RUN+="/bin/raw /dev/raw/raw1 %N"
ACTION=="add", KERNEL=="sdb2", RUN+="/bin/raw /dev/raw/raw2 %N"
ACTION=="add", KERNEL=="sdb3", RUN+="/bin/raw /dev/raw/raw3 %N"
ACTION=="add", KERNEL=="sdb5", RUN+="/bin/raw /dev/raw/raw4 %N"
ACTION=="add", KERNEL=="sdb6", RUN+="/bin/raw /dev/raw/raw5 %N"
KERNEL=="raw[1-6]", MODE="0660", GROUP="asmadmin", OWNER="grid"
---------------------------------------------------------------

partprobe /dev/sdb
```
