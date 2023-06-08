---
layout:       post
title:        "运维日常"
author:       "Kaapo"
header-style: text
catalog:      true
tags:
    - Linux
    - 运维
---

- curl -I -X HEAD mpapi.timeforest-w.cn -x http://172.16.41.175:80
- 查看访问量前几位的ip

```
awk '{print $1}' /usr/local/nginx/logs/access.log |sort -n|uniq -c|sort -nr|head -n 10
```

- 查看https并发连接数以及状态

```
netstat  -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

- 统计文件夹下边的文件的个数(包含子文件夹)
-

```
ls -lR|grep "^-"| grep ".java"|wc -l
```

- 统计文件夹下边的文件夹的个数(包含子文件夹)
-

```
ls -lR|grep "^d"| grep ".java"|wc -l
```

- 查看当前每个ip的连接数

```
netstat -lnp |awk  '/^tcp/ { print $5}'|awk -F “:” '{print $1}'|sort|uniq -c|sort -nr
```

- 统计nginx访问量

```
awk '$9 == 200 {print $7}' /usr/local/nginx/logs/access.log|sort|uniq -c|sort -nr
```

- 测试网页返回值

```
curl -o /dev/null -s -w %{http_code} www.linux.com
```

- 查找包含字段的文件

```
grep -r '192.167.1.1' /etc --include *.conf

递归搜索/etc 目录下包含 ip 的 conf 后缀文件
# grep -r[-r递归搜索] '192.167.1.1' /etc --include *.conf

排除搜索 bak 后缀的文件
# grep -r '192.167.1.1' /opt --exclude *.bak

排除来自 file 中的文件
# grep -r '192.167.1.1' /opt --exclude-from file
```

- 过滤掉空行和#号

```
sed '/^#/d;/^$/d' /etc/httpd/conf/httpd.conf
grep -E -v "^$|^#" /etc/httpd/conf/httpd.conf
awk '! /^#/ && ! /^$/{print $0}'  /etc/httpd/conf/httpd.conf
awk '! /^#|^$/' /etc/httpd/conf/httpd.conf
awk '/^[^#]|"^$"/' /etc/httpd/conf/httpd.conf
```

- tee命令，把后边的输入写到  /etc/docker/daemon.json中，直到遇到EOF位置

```
[root@localhost ~]# tee /etc/docker/daemon.json << EOF
> {
>   "registry-mirrors": ["https://m0vaijvu.mirror.aliyuncs.com"]
> }
> EOF
```

- 自动多级目录

```
1，新建样式
2，将样式作用在多级目录
3, 将样式作用在文本
4，自动生成目录
```

- yum

```
#搜索匹配特定字符的rpm包
yum search firofox
#搜索包含特定文件的rpm包
yum provides firefox
fuser -m -v -k /mnt/databak/
```

- 日志相关

```
last 查看当前登录和过去登录的用户信息   last命令默认读取/var/log/wtmp文件数据
lastlog  查看所有用户最后一次登录时间   lastlog命令默认读取/var/log/lastlog文件内容
/var/log/message	系统启动后的信息和错误日志，是Red Hat Linux中最常用的日志之一 
/var/log/secure	与安全相关的日志信息 
/var/log/maillog	与邮件相关的日志信息 
/var/log/cron	与定时任务相关的日志信息 
/var/log/spooler	与UUCP和news设备相关的日志信息 
/var/log/boot.log	守护进程启动和停止相关的日志消息
```

- 系统：

```
# uname -a   # 查看内核/操作系统/CPU信息 
# cat /etc/issue 
# cat /etc/redhat-release # 查看操作系统版本 
# cat /proc/cpuinfo  # 查看CPU信息 
# hostname   # 查看计算机名 
# lspci -tv   # 列出所有PCI设备 
# lsusb -tv   # 列出所有USB设备 
# lsmod    # 列出加载的内核模块 
# env    # 查看环境变量
```

- 资源：

```
# free -m   # 查看内存使用量和交换区使用量 
# df -h    # 查看各分区使用情况 
# du -sh <目录名>  # 查看指定目录的大小 
# grep MemTotal /proc/meminfo # 查看内存总量 
# grep MemFree /proc/meminfo # 查看空闲内存量 
# uptime   # 查看系统运行时间、用户数、负载 
# cat /proc/loadavg  # 查看系统负载
```

- 磁盘和分区：

```
# mount | column -t  # 查看挂接的分区状态 
# fdisk -l   # 查看所有分区 
# swapon -s   # 查看所有交换分区 
# hdparm -i /dev/hda  # 查看磁盘参数(仅适用于IDE设备) 
# dmesg | grep IDE  # 查看启动时IDE设备检测状况
```

- 网络：

```
# ifconfig   # 查看所有网络接口的属性 
# iptables -L   # 查看防火墙设置 
# route -n   # 查看路由表 
# netstat -lntp   # 查看所有监听端口 
# netstat -antp   # 查看所有已经建立的连接 
# netstat -s   # 查看网络统计信息
```

- 进程：

```
# ps -ef   # 查看所有进程 
# top    # 实时显示进程状态
```

- 用户：

```
# w    # 查看活动用户 
# id <用户名>   # 查看指定用户信息 
# last    # 查看用户登录日志 
# cut -d: -f1 /etc/passwd # 查看系统所有用户 
# cut -d: -f1 /etc/group # 查看系统所有组 
# crontab -l   # 查看当前用户的计划任务
```

- 服务：

```
# chkconfig –list  # 列出所有系统服务 
# chkconfig –list | grep on # 列出所有启动的系统服务
```

- 程序：

```
# rpm -qa   # 查看所有安装的软件包
```

- linux添加路由

```
route add -net 172.18.77.0/24 gw 10.18.93.1
[root@localhost ~]# cat /etc/sysconfig/network-scripts/route-em2
172.18.77.0/24 via 10.18.93.1  dev em2
ifdown em2 ;ifup  em2
```

- mysqldump

```
mysqldump -R -E -ndt -i shopdata -u root -p --default-character-set=utf8 > shopdata_func_proc.sql
mysqldump -uroot -p db_mxh_time > db_max_time.sql
mysql -uroot -p --comment --default-character-set=utf8 db_mxh_time < /tmp/db/func_proc.sql
总结
-d 结构(--no-data:不导出任何数据，只导出数据库表结构)

-t 数据(--no-create-info:只导出数据，而不添加CREATE TABLE 语句)

-n (--no-create-db:只导出数据，而不添加CREATE DATABASE 语句）

-R (--routines:导出存储过程以及自定义函数)

-E (--events:导出事件)

--triggers (默认导出触发器，使用--skip-triggers屏蔽导出)

-B (--databases:导出数据库列表，单个库时可省略）

--tables 表列表（单个表时可省略）

①同时导出结构以及数据时可同时省略-d和-t
②同时不导出结构和数据可使用-ntd
③只导出存储过程和函数可使用-R -ntd
④导出所有(结构&数据&存储过程&函数&事件&触发器)使用-R -E(相当于①，省略了-d -t;触发器默认导出)
⑤只导出结构&函数&事件&触发器使用 -R -E -d
```

- https证书

```
cd /usr/local/src/
yum install -y certbot
cd /usr/local/src/
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
./certbot-auto certonly --standalone --email 992565438@qq.com  -d www.esenlin.cn
yum install -y bind-utils
dig www.esenlin.cn
dig -t www.esenlin.cn
 ./certbot-auto certonly --standalone --email 992565438@qq.com  -d www.esenlin.cn
service nginx stop
service httpd stop
./certbot-auto certonly --standalone --email 992565438@qq.com  -d www.esenlin.cn
./certbot-auto certonly --standalone --email 992565438@qq.com  -d timef.esenlin.cn
./certbot-auto certonly --standalone --email 992565438@qq.com  -d api.esenlin.cn
./certbot-auto certonly --standalone --email 992565438@qq.com  -d tinfo.esenlin.cn
```

- 数据库升级

```
参考http://www.jb51.net/article/77646.htm和https://www.2cto.com/database/201709/683175.html
service mysql stop
cp -ar /usr/local/mysql  /usr/local/mysql.bak    #备份basedir，datadir,my.cnf  启动脚本
tar -zxvf /usr/local/src/mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz  -C /usr/local/
rm -rf mysql/var/*
cp -ra mysql.bak/var/   mysql/
history 
service mysqld start
/usr/local/mysql/bin/mysql_upgrade  -uroot -proot
ps aux |grep mysql
```

- rsync

```
rsync -avL --delete --progress  --exclude-from=/root/exclude.list      /home/   /root/backup/
```

- find

```
‘-atime +n/-n’ : 访问或执行时间大于/小于n天的文件
‘-ctime +n/-n’ : 写入、更改inode属性（例如更改所有者、权限或者链接）时间大于/小于n天的文件
‘-mtime +n/-n’ : 写入时间大于/小于n天的文件
‘-type filetype’   filetype 包含了 f, b, c, d, l, s 等。
'-size +50M/-50M '  按照大小查找
find ./ -mtime  +30|xargs rm -rf
```

去文件夹下最老的两个文件：

```
ls -ltr |grep '^-'|head -2
```

统计文件加下的文件个数

```
ls -lR|grep "^-"|grep ".java" |wc -l
```

Linux下批量Kill多个进程

```
ps -ef|grep php|grep -v grep|cut -c 9-15|xargs kill -9

管道符"|"用来隔开两个命令，管道符左边命令的输出会作为管道符右边命令的输入。下面说说用管道符联接起来的

几个命令：
"ps - ef"是linux 里查看所有进程的命令。这时检索出的进程将作为下一条命令"grep mcfcm_st"的输入。
"grep mcfcm_st"的输出结果是，所有含有关键字"mcfcm_st"的进程，这是Oracle数据库中远程连接进程的共同特点。
"grep -v grep"是在列出的进程中去除含有关键字"grep"的进程。
"cut -c 9-15"是截取输入行的第9个字符到第15个字符，而这正好是进程号PID。
"xargs kill -9"中的xargs命令是用来把前面命令的输出结果（PID）作为"kill -9"命令的参数，并执行该令。


"kill -9"会强行杀掉指定进程，这样就成功清除了oracle的所有远程连接进程。其它类似的任

务，只需要修改"grep php"中的关键字部分就可以了。
```

df -h 卡住

```
strace df -h
umount -l  /卡住的目录
```

准确查看系统启动时间

```
date -d "$(awk -F. '{print $1}' /proc/uptime) second ago" +"%Y-%m-%d %H:%M:%S"
```

sed 替换

```
sed ‘s/test/zcx/g’ ./test1.dat
sed中使用变量
#!/bin/bash
name=tomas
sed -i 's/rose/'"${name}"'/g' b.txt

```

