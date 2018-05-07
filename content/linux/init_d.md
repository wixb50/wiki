---
title: "init.d"
date: 2018-05-07 09:29
updated: 2018-05-07 09:29
log: "增加init.d和rc.local说明"
collection: "系统启动"
---

[TOC]


## `init.d`服务管理

### 关于`/etc/init.d`

当你查看/etc目录时，你会发现许多rc#.d 形式存在的目录（这里#代表一个指定的初始化级别，范围是0~6）。在这些目录之下，包含了许多对进程进行控制的脚本。这些脚本要么以"K"开头，要么以"S"开头。以K开头的脚本运行在以S开头的脚本之前。这些脚本放置的地方，将决定这些脚本什么时候开始运行。在这些目录之间，系统服务一起合作，就像运行状况良好的机器一样。然而，有时候你希望能在不使用kill 或killall 命令的情况下，能干净的启动或杀死一个进程。这就是/etc/init.d能够派上用场的地方了！

```
如果你在使用Fedora系统，你可以找到这个目录：/etc/rc.d/init.d。实际上无论init.d放在什么地方，它都发挥着相同的作用。
```

```
# init相关命令
/etc/init.d/service [start|stop|reload|restart|force]
```

### 自定义一个init服务(redis为例)

新建/etc/init.d/redisd脚本

```
#!/bin/sh
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=6379
EXEC=/usr/bin/redis-server
CLIEXEC=/usr/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT -a 123456 shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

> 其中chkconfig各字段意义如下：

```
2345：在2、3、4、5级别（级别见下文说明）启动redis
60：启动顺序为60号，S60
10：关闭顺序为10号，K10
当进行chkconfig --add httpd操作时，如果没有指定level那么就会来这个注释中取值
```

执行下列命令

```
chkconfig redisd on
service redisd start # 打开 redis 服务
service redisd stop # 关闭 redis 服务
```

### chkconfig命令

chkconfig命令主要用来更新（启动或停止）和查询系统服务的运行级信息。谨记chkconfig不是立即自动禁止或激活一个服务，它只是简单的改变了符号连接。

使用语法：
```
chkconfig [--add][--del][--list][系统服务] 或 chkconfig [--level <等级代号>][系统服务][on/off/reset]
```

参数用法：
```
--add 　增加所指定的系统服务，让chkconfig指令得以管理它，并同时在系统启动的叙述文件内增加相关数据。
--del 　删除所指定的系统服务，不再由chkconfig指令管理，并同时在系统启动的叙述文件内删除相关数据。
--level<等级代号> 　指定读系统服务要在哪一个执行等级中开启或关毕。
  等级0表示：表示关机
  等级1表示：单用户模式
  等级2表示：无网络连接的多用户命令行模式
  等级3表示：有网络连接的多用户命令行模式
  等级4表示：不可用
  等级5表示：带图形界面的多用户模式
  等级6表示：重新启动
  需要说明的是，level选项可以指定要查看的运行级而不一定是当前运行级。对于每个运行级，只能有一个启动脚本或者停止脚本。当切换运行级时，init不会重新启动已经启动的服务，也不会再次去停止已经停止的服务。

chkconfig --list [name]：显示所有运行级系统服务的运行状态信息（on或off）。如果指定了name，那么只显示指定的服务在不同运行级的状态。
chkconfig --add name：增加一项新的服务。chkconfig确保每个运行级有一项启动(S)或者杀死(K)入口。如有缺少，则会从缺省的init脚本自动建立。
chkconfig --del name：删除服务，并把相关符号连接从/etc/rc[0-6].d删除。
chkconfig [--level levels] name：设置某一服务在指定的运行级是被启动，停止还是重置。
```

运行级文件：

每个被chkconfig管理的服务需要在对应的init.d下的脚本加上两行或者更多行的注释。第一行告诉chkconfig缺省启动的运行级以及启动和停止的优先级。如果某服务缺省不在任何运行级启动，那么使用 - 代替运行级。第二行对服务进行描述，可以用\ 跨行注释。
例如，random.init包含三行：
```
# chkconfig: 2345 20 80
# description: Saves and restores system entropy pool for \
# higher quality random number generation.
```

使用范例：
```
chkconfig --list        #列出所有的系统服务
chkconfig --add httpd        #增加httpd服务
chkconfig --del httpd        #删除httpd服务
chkconfig --level httpd 2345 on        #设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态
chkconfig --list        #列出系统所有的服务启动情况
chkconfig --list mysqld        #列出mysqld服务设置情况
chkconfig --level 35 mysqld on        #设定mysqld在等级3和5为开机运行服务，--level 35表示操作只在等级3和5执行，on表示启动，off表示关闭
chkconfig mysqld on        #设定mysqld在各等级为on，“各等级”包括2、3、4、5等级
```

如何增加一个服务：

1. 服务脚本必须存放在`/etc/ini.d/`目录下；
2. `chkconfig --add servicename`，在chkconfig工具服务列表中增加此服务，此时服务会被在`/etc/rc.d/rcN.d`中赋予K/S入口了；
3. `chkconfig --level 35 mysqld on`，修改服务的默认启动等级


## 关于`/etc/rc.local`

rc.local也是使用的一个脚本。该脚本是在系统初始化级别脚本运行之后再执行的，因此可以安全地在里面添加你想在系统启动之后执行的脚本。常见的情况是你可以再里面添加nfs挂载/mount脚本等。如下：

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

/usr/local/zookeeper-3.4.8/bin/zkServer.sh  start # 启动zookeeper服务

exit 0
```

## 参考文档

+ [Linux中Apache(httpd)安装、配置、加为服务](https://blog.csdn.net/u010297957/article/details/50751656)
+ [Linux-安装-redis-并配置成-service-系统服务](https://github.com/xuxiangwork/Sharing/wiki/Linux-%E5%AE%89%E8%A3%85-redis-%E5%B9%B6%E9%85%8D%E7%BD%AE%E6%88%90-service-%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1)
+ [理解Linux系统/etc/init.d目录和/etc/rc.local脚本](https://blog.csdn.net/acs713/article/details/7322082)