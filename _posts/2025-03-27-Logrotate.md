---
layout: post
title: Logrotate手册
subtitle: 
cover-img: 
thumbnail-img: /assets/img/logrotate.jpeg
share-img: 
tags: [install]
author: Jamie
---

{: .box-success}
logrotate是一个日志文件管理工具，用于分割日志，删除旧的日志，并生成新的日志，起到”转储”作用，为了节省硬盘空间

## 配置介绍

### 安装

> linux默认安装logrotate

```bash
# 主要的配置文件
/etc/logrotate.conf
# 目录下的所有的文件都会被主动的读入config中，如果该文件夹下的某些参数没有配置，则直接使用conf文件内的配置
/etc/logrotate.d/

cat /etc/cron.daily/logrotate


#!/bin/sh
/usr/sbin/logrotate /etc/logrotate.conf >/dev/null 2>&1
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```

### 命令

```shell
# 验证日志切割文件
logrotate -d /etc/logrotate.d/mariadb
# 手动执行一次日志切割
logrotate -f -d /etc/logrotate.d/mariadb
```

### 可选命令

```bash
logrotate [OPTION...] <configfile>
-d --debug : debug模式，测试配置文件是否有误
-f --force : 强制转储文件
-m --mail=command : 压缩日志后，将日志发送到指定邮箱
-s --state=statefile : 使用指定状态文件
-v --verbose : 显示转储过程
```

### 状态查看

```bash
cat /var/lib/logrotate/logrotate.status

logrotate state -- version 2
"/var/log/yum.log" 2022-6-27-21:28:23
"/var/log/firewalld" 2022-1-19-3:0:0
"/var/log/drms-analysis-waj/drms-analysis-waj.log" 2022-6-27-21:28:23
"/var/log/chrony/*.log" 2022-1-13-3:0:0
"/var/log/wtmp" 2022-6-27-21:28:23
```

## 切割介绍

- 第一次执行完rotate(轮转)之后，原本的messages会变成messages.1，而且会制造一个空的messages给系统来储存日志；
- 第二次执行之后，messages.1会变成messages.2，而messages会变成messages.1，又造成一个空的messages来储存日志！
- 如果仅设定保留三个日志（即轮转3次）的话，那么执行第三次时，则 messages.3这个档案就会被删除，并由后面的较新的保存日志所取代！

### 配置参数

```conf
# cat /etc/logrotate.conf

# 以下是默认值，如果其他文件设置了改值，就以其他文件的设定为主

weekly    // 默认每一周执行一次rorate轮转工作
rotate 4  // 保留多少个日志文件，默认保留4个，0指没有备份
create    // 自动创建新的日志文件，与原来的文件有相同的权限
dateext   // 切割后的日志文件以当前日期格式结尾，如xx.log-20220123，如果注释掉，切割是按照数字递增的xx.log-1
compress  // 是够通过gzip压缩转储以后的日志文件，如果不需要，注释掉

include /etc/logrotate.d   // 将目录下的所有文件都加载进来

/var/log/wtmp {        //仅针对wtmp日志文件设置的参数
monthly                //每月一次的切割
minsize 1M             //文件大小超过1M后才会切割
create 0664 root utmp  //指定新建日志的文件权限及所属用户和组
rotate 1               //只保留一个日志
}
```

> 单个文件参数列表

```shell
compress   //通过gzip压缩转储以后的日志
nocompress //不过gzip压缩处理
copytruncate //先拷贝再清空，有时间差，可能会丢失部分日志
nocopytruncate 
create mode owner group //轮转时指定创建新文件的属性，如create 077 nobody nobody
nocreate  //不建立新的日志文件
delaycompress //和compress一起使用，转存的日志文件到下一次转储才压缩
nodelaycompress //覆盖delaycompress选项，转储同时压缩
missignok  //如果日志丢失，不报错继续滚动 下一个日志
errors address //转储时的错误信息发送到指定的Email地址
ifempty //及时日志文件为空也做轮转
notifempty //日志为空时不轮转
mail address //转储的日志文件发送到指定的Email
nomail //转储时不发送日志文件
olddir directory //转储后的日志文件放入指定的目录，必须和当前日志在同一个系统
noolddir //转储后日志文件和当前日志文件放在同一目录下

sharedscripts //运行postrotate脚本，作用是在所有日志都轮转后统一执行一次脚本，如果没有配置这个，那么每个日志轮转后都会执行一次这个脚本
prerotate //在logrotate转储值之前需要指定的命令，例如修改文件的属性等动作；必须独立成行
postrotate //在logrotate转储之后需要执行的命令，例如重新启动，必须独立成行

daily //指定转储周期为每天
weekly //指定转储周期为每周
monthly //指定转储周期为每月
rotate count //指定日志文件删除之前的转储次数，0指没有备份，5指5个备份
dateext //使用当前日期作为命名格式
dateformat .%s //配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，必须配合dateext使用，只支持 %Y %m %d %s 这四个参数
size 100M //日志到达100M就会转储
```

### 示例

> nginx切割日志

```shell
[root@master-server ~]# vim /etc/logrotate.d/nginx
/usr/local/nginx/logs/*.log {
daily
rotate 7
missingok
notifempty
dateext
sharedscripts
postrotate
    if [ -f /usr/local/nginx/logs/nginx.pid ]; then
        kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
    fi
endscript
}
```

> syslog
> 

```shell
/var/log/cron
/var/log/maillog
/var/log/messages
/var/log/secure
/var/log/spooler
{
    missingok
    sharedscripts
    postrotate
        /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
    endscript
}
```

> chrony

```shell
/var/log/chrony/*.log {
    missingok
    nocreate
    sharedscripts
    postrotate
        /usr/bin/chronyc cyclelogs > /dev/null 2>&1 || true
    endscript
}
```

> firewalld

```shell
/var/log/firewalld {
    weekly
    missingok
    rotate 4
    copytruncate
    minsize 1M
}
```