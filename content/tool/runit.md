---
title: "runit"
date: 2018-05-30 13:31
collection: "服务管理工具"
---

[TOC]


## `runit`服务管理

### 关于`runit`

runit是一个用于服务监控的UNIX软件，它提供以下两种服务：

+ 当服务器启动的时候启动定义好的服务。
+ 监控运行的服务，当服务发生意外中断的时候，自动重启服务。

### 安装与环境搭建

首先确保系统安装了runit，大多数Linux版本的软件仓库里都包哈了runit包。例如，如果你的系统是基于Debian的，则可以使用下面的命令进行安装：
```
$ apt-get install runit
```
如果是centos，则可以使用yum进行安装，但是默认情况下centos软件仓库里并没有runit，所以需要先配置相应的仓库：
```
$ curl -s https://packagecloud.io/install/repositories/imeyer/runit/script.rpm.sh | sudo bash
$ sudo yum install runit-2.1.1-7.el7.centos.x86_64
```
运行以下命令来检查是否已经安装了runit并且系统已经运行了runit。
```
$ ps -ef | grep runsvdir
# 输出如下
root 2783 1 0 15:34 ? 00:00:00 runsvdir -P /etc/service log:
```
runsvdir其实是一套组件，这些组件可以满足用户的各种需求，核心组件包括了runsvdir，runsv， chpst，svlogd以及sv。

### 服务创建

注意输出结果中的`runsvdir -P /etc/service log:.......`， 它的意思是runsvdir会监控/etc/service/目录下的文件，这些文件用于配置被监控的服务。

被监控的服务是通过在/etc/service目录下创建子目录，并添加可执行脚本run来实现的。

当runsvdir发现新的配置文件时，它就会自动启动一个`runsv`进程来管理这个配置的服务。

确保存在/etc/service，如果不存在，则使用mkdir创建相应目录：
```
$ mkdir /etc/service
```

#### 新建一个服务
```
$ mkdir /etc/service/template
$ touch /etc/service/template/run && chmod +x /etc/service/template/run
```
配置服务启动,编辑`/etc/service/template/run`
```
#!/bin/sh -e
exec 2>&1
exec chpst -u USER COMMAND
```
这个脚本首先将标准错误输出流输出到标准输出流，然后执行chpst命令。chpst命令用来指定使用哪个用户执行命令。由于run脚本默认被root用户执行，通过chpst可以将run配置为普通用户来执行。通过man命令可以查看chpst的更多信息。

当runsvdir检查到/etc/service目录下包含一个新的目录时，runsvdir会启动一个runsv进程来执行和监控run脚本。通过man命令查看runsv的更多信息。

#### 配置服务日志输出(可选)
```
$ mkdir /etc/service/template/log
$ touch /etc/service/template/log/run && chmod +x /etc/service/template/log/run
```
编辑`/etc/service/template/log/run`
```
#!/bin/sh
exec chpst -u USER svlogd -tt LOGDIR
```
上面的脚本使用chpst启动一个svlogd守护进程，该进程将日志信息写到LOGDIR目录中。使用man命令获取更多关于svlodg的信息。

当runsvdir在/etc/service/目录中发现新的配置时，它会继续查找子目录log，如果找到了则启动runsv进程来执行和监控log目录下的run脚本。


### 管理服务

+ 检查服务的状态
```
sv status example
```

+ 停止服务
```
sv stop example
```
停止服务之后不会再输出日志信息，也不会再自动重启。

+ 重启服务
```
sv restart example
```

## 参考文档

+ [runit 快速入门](https://segmentfault.com/a/1190000006644578)
