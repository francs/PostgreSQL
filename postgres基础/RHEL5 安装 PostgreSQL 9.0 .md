RHEL5 安装 PostgreSQL 9.0 
====================

**作者**：谭峰(francs) 

**日期**：2010-07-31
-------------------

 以下是第一次在红帽子五上安装POSTGRESQL笔记, 该篇日志描述了在Red Hat Enterprise 5上安装Postgresql-9.0beat3的详细安装过程， 操作系统安装步骤已省略。

## 环境
虚拟机 :   Vmware Workstation 5

操作系统： Red Hat Enterprise 5

数据库：   postgresql-9.0beta3


## 修改操作系统参数
```
su - root
vi /etc/sysctl.conf
kernel.shmmni = 4096
kernel.sem = 501000 6412800000 501000 12800
fs.file-max = 767246
net.ipv4.ip_local_port_range = 1024 65000
net.core.rmem_default = 1048576
net.core.rmem_max = 1048576
net.core.wmem_default = 262144
net.core.wmem_max = 262144
net.ipv4.tcp_tw_recycle=1 
net.ipv4.tcp_max_syn_backlog=4096 
net.core.netdev_max_backlog=10000
vm.overcommit_memory=0
net.ipv4.ip_conntrack_max=655360
```
sysctl -p 生效

## 修改 /etc/security/limits.conf 文件

```
*  soft    nofile  131072
*  hard    nofile  131072
*  soft    nproc   131072
*  hard    nproc   131072
*  soft    core    unlimited
*  hard    core    unlimited
*  soft    memlock 50000000
*  hard    memlock 50000000
```

## 修改  /etc/pam.d/login 文件

``` 
session required pam_limits.so
```

## 添加用户和组

```
# groupadd postgresql
# useradd -g postgresql postgresql
# passwd postgres
```
## 创建目录 

```
mkdir -p /opt/pg_root/data
mkdir -p /opt/pg_root/log
touch /opt/pg_root/log/pgsql.log

cd /usr/local
chown -R postgres:postgres pg_root 
ln -sf /opt/pg_root    /database/pgdata/pg_root
```

## 配置环境变量

```
su - postgres
vi .bash_profile
export DATE=`date +"%Y%m%d%H%M"`

export PGPORT=1921
export PGDATA=/database/pgdata/tbs1/pg_root/data
export PGHOME=/opt/pgsql_9beta3
export PATH=/database/pgdata/tbs1/pg_root/bin:$PATH

export LD_LIBRARY_PATH=export DATE=`date +"%Y%m%d%H%M"`
export MANPATH=$PGHOME/share/man:$MANPATH
export LANG=en_US.utf8
alias rm='rm -i'
alias ll='ls -lh'
```

## 解压并编译

解压安装包到 /opt/postgresql-9.0beta3：
```
# tar xvf postgresql-9.0beta3.tar.gz
```

进行安装配置
```
cd /opt/postgresql-9.0beta3
./configure --prefix=/opt/pg_root --with-pgport=1921 --with-segsize=8 --with-wal-segsize=64 --with-wal-blocksize=64 --with-perl --with-python --with-openssl --with-pam --with-ldap --with-libxml --with-libxslt --enable-thread-safety
```

注意：Configure 脚本详细选项可以查看帮助：./configure --help，如查看版本信息：`./configure -V ` ， 如果报缺少相应包，则 yum 安装相应包即可。


## gmake

```
# gmake
```

这个过程比较长，大概在二十分钟以上 没有任何问题的话，我们可以看到最后一句提示信息
`“All of PostgreSQL successfully made. Ready to install.”`

## 开始安装

```
# gmake install-world
```

成功安装后能看到最后一句提示信息"PostgreSQL installation complete."


## 初始化数据库(PSQL用户)

```
root# su - postgres
initdb -D /opt/pg_root/data  -E UTF8 --locale=C -U postgres -W
```

注意: initdb脚本详细选项可以查看 `initdb --help`, 例如查看initdb脚本版本命令如下 `initdb -V`。


命令执行完后提示

```
Success. You can now start the database server using:

    postgres -D /usr/local/pgsql/data
or
    pg_ctl -D /usr/local/pgsql/data -l logfile start
```
    
## 启动数据库
```
pg_ctl -D /database/pgdata/pg_root/data-l /database/pgdata/pg_root/log/pgsql.log start
```

## 关闭数据库 (参考步骤，这步只是为了显示关闭数据库命令)

```
pg_ctl  -D /database/pgdata/pg_root/data
```

## 创建数据库用户
```
CREATE ROLE usera LOGIN
  ENCRYPTED PASSWORD 'usera'
   nosuperuser inherit nocreatedb nocreaterole;
```

## 创建数据库
```
createdb  mydb -O usera
```
 
## 登陆测试
```
psql -dmydb -Uusera
psql (9.0beta3)
Type "help" for help.

oup=> select version();
                                                       version                                                        
----------------------------------------------------------------------------------------------------------------------
 PostgreSQL 9.0beta3 on x86_64-unknown-linux-gnu, compiled by GCC gcc (GCC) 4.1.2 20080704 (Red Hat 4.1.2-46), 64-bit
(1 row)
```
