---
title: "Docker"
date: 2015-11-08 12:34
updated: 2016-03-07 23:39
collection: "虚拟化"
---

[Docker](https://www.docker.com/) Build, Ship, Run; An open platform for distributed applications for developers and sysadmins

## 前言 ##

下面5个链接是2014年的博客, 基于0.10.0, 当前是1.9.x (有些已经失效)

* [Docker 1 -- 开始](http://blog.tankywoo.com/docker/2014/05/08/docker-1-start.html)
* [Docker 2 -- 关于Dockerfile](http://blog.tankywoo.com/docker/2014/05/08/docker-2-dockerfile.html)
* [Docker 3 -- 自建Docker Registry](http://blog.tankywoo.com/docker/2014/05/08/docker-3-docker-registry.html)
* [Docker 4 -- 总结](http://blog.tankywoo.com/docker/2014/05/08/docker-4-summary.html)
* [Docker 网络桥接](http://blog.tankywoo.com/2014/12/22/docker-bridge-network.html)

入门参考:

* [Docker - Get Started](http://docs.docker.com/linux/started/)
* [Docker - Docs](https://docs.docker.com/)

其实以前入门时, 有一个Interactive commandline tutorial, 我觉得挺好的, 但是现在没了.

那个时候docker官网的域名还是docker.io, 现在已经把docker.com拿下了, 变化还是挺大的.

## 遇到的问题 ##

## loop module ##

Gentoo下安装docker后启动报错, 看日志:

> level=info msg="Listening for HTTP on unix (/var/run/docker.sock)"
> level=error msg="There are no more loopback devices available."
> level=fatal msg="Error starting daemon: error initializing graphdriver: loopback mounting failed"

原因是没有加载`loop`模块. 在`/etc/conf.d/modules` 中加入模块:

	$ cat /etc/conf.d/modules
	# You can define a list modules for a specific kernel version,
	# a released kernel version, a main kernel version or just a list.
	# The most specific versioned variable will take precedence.
	#modules_2_6_23_gentoo_r5="ieee1394 ohci1394"
	#modules_2_6_23="tun ieee1394"
	#modules_2_6="tun"
	#modules_2="ipv6"
	#modules="ohci1394"
	
	# You can give modules a different name when they load - the new name
	# will also be used to pick arguments below.
	#modules="dummy:dummy1"
	
	# Give the modules some arguments if needed, per version if necessary.
	# Again, the most specific versioned variable will take precedence.
	#module_ieee1394_args="debug"
	#module_ieee1394_args_2_6_23_gentoo_r5="debug2"
	#module_ieee1394_args_2_6_23="debug3"
	#module_ieee1394_args_2_6="debug4"
	#module_ieee1394_args_2="debug5"
	
	# You should consult your kernel documentation and configuration
	# for a list of modules and their options.
	modules="loop"

然后加载模块: `modprobe loop`

## 内核支持 ##

一个是iptables nat表, 编译内核时忘了打开nat支持了, 导致docker启动不了(docker开启不修改iptables选项应该就没事了).

打开内核选项:

	CONFIG_IP_NF_NAT

[can't initialize iptables table `nat'`](https://forums.gentoo.org/viewtopic-t-1009770.html?sid=7822e8eefcdb28edcedf9db7526b7b1e)

安装完docker顺便发现有如下提示:

	* Messages for package app-emulation/docker-1.10.0:
	
	*   CONFIG_IP_NF_TARGET_MASQUERADE:     is not set when it should be.
	*   CONFIG_CGROUP_HUGETLB:      is not set when it should be.
	*   CONFIG_CGROUP_NET_PRIO:     is not set when it should be.
	* Please check to make sure these options are set correctly.
	* Failure to do so may cause unexpected problems.

就顺便一起重新编译内核了.

官方也提供了内核参数检查的脚本 [docker/contrib/check-config.sh](https://github.com/docker/docker/blob/master/contrib/check-config.sh)

## 一些链接 ##

* [Docker —— 从入门到实践](https://www.gitbook.com/book/yeasy/docker_practice/details)
* [国内daocloud的一些镜像](https://dashboard.daocloud.io/packages) 国内现在没找到什么好的仓库, 这个算一个, 就是镜像略少
