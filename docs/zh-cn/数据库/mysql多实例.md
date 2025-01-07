````
1.MySQL多实例介绍

1.1.什么是MySQL多实例

MySQL多实例就是在一台机器上开启多个不同的服务端口（如：3306,3307），运行多个MySQL服务进程，通过不同的socket监听不同的服务端口来提供各自的服务:；

1.2.MySQL多实例的特点有以下几点

1：有效利用服务器资源，当单个服务器资源有剩余时，可以充分利用剩余的资源提供更多的服务。

2：节约服务器资源

3：资源互相抢占问题，当某个服务实例服务并发很高时或者开启慢查询时，会消耗更多的内存、CPU、磁盘IO资源，导致服务器上的其他实例提供服务的质量下降；

1.3.部署mysql多实例的两种方式

第一种是使用多个配置文件启动不同的进程来实现多实例，这种方式的优势逻辑简单，配置简单，缺点是管理起来不太方便；

第二种是通过官方自带的mysqld\\_multi使用单独的配置文件来实现多实例，这种方式定制每个实例的配置不太方面，优点是管理起来很方便，集中管理；

1.4.同一开发环境下安装两个数据库，必须处理以下问题

配置文件安装路径不能相同

数据库目录不能相同

启动脚本不能同名

端口不能相同

socket文件的生成路径不能相同

2.Mysql多实例安装部署

2.1.部署环境

阿里云centos 7.2(防火墙已关闭)

2.2.安装mysql软件版本

2.2.1.免编译二进制包

mysql-5.6.21-linux-glibc2.5-x86\\_64.tar.gz

2.3.解压和迁移

\[root\@panshi-mysql56 ~]#tar -xvf mysql-5.6.21-linux-glibc2.5-x86\\_64.tar.gz

\[root\@panshi-mysql56 ~]#mv mysql-5.6.21-linux-glibc2.5-x86\\_64 /usr/local/mysql

2.4.创建mysql用户

\[root\@panshi-mysql56 ~]#groupadd -g 27 mysql

\[root\@panshi-mysql56 ~]#useradd -u 27 -g mysql mysql

2.5.创建相关目录

\[root\@panshi-mysql56 ~]#mkdir -p /data/mysql/{mysql\\_3306,mysql\\_3307,mysql\\_3308,mysql\\_3309,mysql\\_3310,mysql\\_3311}

\[root\@panshi-mysql56 ~]#mkdir /data/mysql/mysql\\_3306/{data,log,tmp}

\[root\@panshi-mysql56 ~]#mkdir /data/mysql/mysql\\_3307/{data,log,tmp}

\[root\@panshi-mysql56 ~]#mkdir /data/mysql/mysql\\_3308/{data,log,tmp}

\[root\@panshi-mysql56 ~]#mkdir /data/mysql/mysql\\_3309/{data,log,tmp}

\[root\@panshi-mysql56 ~]#mkdir /data/mysql/mysql\\_3310/{data,log,tmp}

\[root\@panshi-mysql56 \~]#mkdir /data/mysql/mysql\\_3311/{data,log,tmp}

2.6.更改目录权限

\[root\@panshi-mysql56 ~]#chown -R mysql\:mysql /data/mysql/

\[root\@panshi-mysql56 ~]#chown -R mysql\:mysql /usr/local/mysql/

2.7. 添加环境变量

\[root\@panshi-mysql56 ~]#echo 'export PATH=\$PATH:/usr/local/mysql/bin' >> /etc/profile

\[root\@panshi-mysql56 ~]#source /etc/profile

2.8.复制my.cnf文件到etc目录

cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf

2.9.修改my.cnf（在一个文件中修改即可）

\[root\@panshi-mysql56 \~]# vim /etc/my.conf

\[client]\

port=3306\

socket=/tmp/mysql.sock

\[mysqld\\_multi]\

mysqld = /usr/local/mysql /bin/mysqld\\_safe\

mysqladmin = /usr/local/mysql /bin/mysqladmin\

log = /data/mysql/mysqld\\_multi.log

\[mysqld]\

user=mysql\

basedir = /usr/local/mysql\

sql\\_mode=NO\\_ENGINE\\_SUBSTITUTION,STRICT\\_TRANS\\_TABLES

\[mysqld3306]\

mysqld=mysqld\

mysqladmin=mysqladmin\

datadir=/data/mysql/mysql\\_3306/data\

port=3306\

server\\_id=3306\

socket=/tmp/mysql\\_3306.sock\

log-output=file\

slow\\_query\\_log = 1\

long\\_query\\_time = 1\

slow\\_query\\_log\\_file = /data/mysql/mysql\\_3306/log/slow\.log\

log-error = /data/mysql/mysql\\_3306/log/error.log\

binlog\\_format = mixed\

log-bin = /data/mysql/mysql\\_3306/log/mysql3306\\_bin\

query\\_cache\\_type=1

\[mysqld3307]\

mysqld=mysqld\

mysqladmin=mysqladmin\

datadir=/data/mysql/mysql\\_3307/data\

port=3307\

server\\_id=3307\

socket=/tmp/mysql\\_3307.sock\

log-output=file\

slow\\_query\\_log = 1\

long\\_query\\_time = 1\

slow\\_query\\_log\\_file = /data/mysql/mysql\\_3307/log/slow\.log\

log-error = /data/mysql/mysql\\_3307/log/error.log\

binlog\\_format = mixed\

log-bin = /data/mysql/mysql\\_3307/log/mysql3307\\_bin

query\\_cache\\_type=1 #开启缓存

default-time-zone = '+08:00' #shiqu

\[mysqld3308]\

mysqld=mysqld\

mysqladmin=mysqladmin\

datadir=/data/mysql/mysql\\_3308/data\

port=3308\

server\\_id=3308\

socket=/tmp/mysql\\_3308.sock\

log-output=file\

slow\\_query\\_log = 1\

long\\_query\\_time = 1\

slow\\_query\\_log\\_file = /data/mysql/mysql\\_3308/log/slow\.log\

log-error = /data/mysql/mysql\\_3308/log/error.log\

binlog\\_format = mixed\

log-bin = /data/mysql/mysql\\_3306/log/mysql3306\\_bin\

query\\_cache\\_type=1

default-time-zone = '+08:00'

\[mysqld3309]\

mysqld=mysqld\

mysqladmin=mysqladmin\

datadir=/data/mysql/mysql\\_3309/data\

port=3309\

server\\_id=3309\

socket=/tmp/mysql\\_3309.sock\

log-output=file\

slow\\_query\\_log = 1\

long\\_query\\_time = 1\

slow\\_query\\_log\\_file = /data/mysql/mysql\\_3309/log/slow\.log\

log-error = /data/mysql/mysql\\_3309/log/error.log\

binlog\\_format = mixed\

log-bin = /data/mysql/mysql\\_3309/log/mysql3309\\_bin

query\\_cache\\_type=1

default-time-zone = '+08:00'

\[mysqld3310]\

mysqld=mysqld\

mysqladmin=mysqladmin\

datadir=/data/mysql/mysql\\_3310/data\

port=3310\

server\\_id=3310\

socket=/tmp/mysql\\_3310.sock\

log-output=file\

slow\\_query\\_log = 1\

long\\_query\\_time = 1\

slow\\_query\\_log\\_file = /data/mysql/mysql\\_33010/log/slow\.log\

log-error = /data/mysql/mysql\\_3310/log/error.log\

binlog\\_format = mixed\

log-bin = /data/mysql/mysql\\_3310/log/mysql3310\\_bin

query\\_cache\\_type=1

default-time-zone = '+08:00'

\[mysqld3311]

mysqld=mysqld

mysqladmin=mysqladmin

datadir=/data/mysql/mysql\\_3311/data

port=3311

server\\_id=3311

socket=/tmp/mysql\\_3311.sock

log-output=file

slow\\_query\\_log = 1

long\\_query\\_time = 1

slow\\_query\\_log\\_file = /data/mysql/mysql\\_33011/log/slow\.log

log-error = /data/mysql/mysql\\_3311/log/error.log

binlog\\_format = mixed

log-bin = /data/mysql/mysql\\_3311/log/mysql3311\\_bin

query\\_cache\\_type=1

default-time-zone = '+08:00'

2.12. 初始化数据库

初始化3306数据库

\[root\@panshi-mysql56 ~]#/usr/local/mysql/scripts/mysql\\_install\\_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql\\_3306/data --defaults-file=/etc/my.cnf

初始化3307数据库

\[root\@panshi-mysql56 ~]#/usr/local/mysql/scripts/mysql\\_install\\_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql\\_3307/data --defaults-file=/etc/my.cnf

初始化3308数据库

\[root\@panshi-mysql56 ~]#/usr/local/mysql/scripts/mysql\\_install\\_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql\\_3308/data --defaults-file=/etc/my.cnf

初始化3309数据库

\[root\@panshi-mysql56 ~]#/usr/local/mysql/scripts/mysql\\_install\\_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql\\_3309/data --defaults-file=/etc/my.cnf

初始化3310数据库

\[root\@panshi-mysql56 ~]#/usr/local/mysql/scripts/mysql\\_install\\_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql\\_3310/data --defaults-file=/etc/my.cnf

初始化3311数据库

\[root\@panshi-mysql56 ~]#/usr/local/mysql/scripts/mysql\\_install\\_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql\\_3311/data --defaults-file=/etc/my.cnf

检查数据库是否初始化成功

执行初始化后出现”OK”

2.12.4. 查看数据库是否初始化成功（2）

查看3306数据库

\[root\@mysql \~]# cd /data/mysql/mysql\\_3306/data

\[root\@mysql data]# ls

auto.cnf ibdata1 ib\\_logfile0 ib\\_logfile1 mysql mysql.pid performance\\_schema test

查看3307数据库

\[root\@mysql \~]# cd /data/mysql/mysql\\_3307/data

\[root\@mysql data]# ls

auto.cnf ibdata1 ib\\_logfile0 ib\\_logfile1 mysql mysql.pid performance\\_schema test

```

同样方法查询其他实例

2.13.设置启动文件

[root@panshi-mysql56 ~]#cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql

2.14.mysqld\_multi进行多实例管理

启动全部实例：/usr/local/mysql/bin/mysqld\_multi start

查看全部实例状态：/usr/local/mysql/bin/mysqld\_multi report

启动单个实例：/usr/local/mysql/bin/mysqld\_multi start 3306

停止单个实例：/usr/local/mysql/bin/mysqld\_multi stop 3306

查看单个实例状态：/usr/local/mysql/bin/mysqld\_multi report 3306

2.14.1.启动全部实例

[root@panshi-mysql56 ~]# /usr/local/mysql/bin/mysqld\_multi start

[root@panshi-mysql56 ~]# /usr/local/mysql/bin/mysqld\_multi report

Reporting MySQL servers

MySQL server from group: mysqld3306 is running

MySQL server from group: mysqld3307 is running

MySQL server from group: mysqld3308 is running

MySQL server from group: mysqld3309 is running

MySQL server from group: mysqld3310 is running

MySQL server from group: mysqld3311 is running

2.15.查看启动进程 netstat -lnap |grep mysql

[root@panshi-mysql56 ~]#netstat -lnap |grep mysql

2.16.修改密码

mysql的root用户初始密码是空，所以需要登录mysql进行修改密码，下面以3306为例：

无密码登陆：mysql -S /tmp/mysql\_3306.sock

set password for root@'localhost'=password('123456');

flush privileges;

下次登录：

[root@mysql ~]# mysql -S /tmp/mysql\_3306.sock -p

Enter password:

2.17.新建用户及授权

一般新建数据库都需要新增一个用户，用于程序连接，这类用户只需要insert、update、delete、select权限。

新增一个用户，并授权如下：

mysql>grant select,delete,update,insert on \*.\* to <username>@'%' identified by '<password>';

```xxxxxxxxxx 同样方法查询其他实例2.13.设置启动文件[root@panshi-mysql56 ~]#cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql2.14.mysqld\_multi进行多实例管理启动全部实例：/usr/local/mysql/bin/mysqld\_multi start查看全部实例状态：/usr/local/mysql/bin/mysqld\_multi report启动单个实例：/usr/local/mysql/bin/mysqld\_multi start 3306停止单个实例：/usr/local/mysql/bin/mysqld\_multi stop 3306查看单个实例状态：/usr/local/mysql/bin/mysqld\_multi report 33062.14.1.启动全部实例[root@panshi-mysql56 ~]# /usr/local/mysql/bin/mysqld\_multi start[root@panshi-mysql56 ~]# /usr/local/mysql/bin/mysqld\_multi reportReporting MySQL serversMySQL server from group: mysqld3306 is runningMySQL server from group: mysqld3307 is runningMySQL server from group: mysqld3308 is runningMySQL server from group: mysqld3309 is runningMySQL server from group: mysqld3310 is runningMySQL server from group: mysqld3311 is running2.15.查看启动进程 netstat -lnap |grep mysql[root@panshi-mysql56 ~]#netstat -lnap |grep mysql2.16.修改密码mysql的root用户初始密码是空，所以需要登录mysql进行修改密码，下面以3306为例：无密码登陆：mysql -S /tmp/mysql\_3306.sockset password for root@'localhost'=password('123456');flush privileges;下次登录：[root@mysql ~]# mysql -S /tmp/mysql\_3306.sock -pEnter password:2.17.新建用户及授权一般新建数据库都需要新增一个用户，用于程序连接，这类用户只需要insert、update、delete、select权限。新增一个用户，并授权如下：mysql>grant select,delete,update,insert on \*.\* to <username>@'%' identified by '<password>';
````