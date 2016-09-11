---
title: "docker tips"
date: 2016-08-05 18:32
collection: "docker"
---

## 安装 ##

```
curl -sSL https://get.docker.com/ | sh
```

官方也提供了内核参数检查的脚本 [docker/contrib/check-config.sh](https://github.com/docker/docker/blob/master/contrib/check-config.sh)

## dockerv1.12

+ https://yq.aliyun.com/articles/57576
+ engine自带swarm
+ 离线deb包: https://apt.dockerproject.org/repo/pool/main/d/docker-engine/  

## 搭建docker私有仓库：

+ http://www.cnblogs.com/lienhua34/p/4922130.html  
+ http://dockone.io/question/757  
+ http://www.cnblogs.com/baxk/p/5225533.html (good)   
+ [自己搭建registry v2](http://www.zimug.com/317.html) (OK)
+ [私有仓库web-ui](https://github.com/kwk/docker-registry-frontend)
+ [vmware的开源私有仓库](https://github.com/vmware/harbor)

## docker stats 异常

+ https://github.com/docker/docker/issues/18420
+ https://github.com/docker/docker/issues/251

## docker代理

+ https://docs.docker.com/engine/admin/systemd/#http-proxy
+ [Ubuntu 15.04及后续版本Docker的代理设置](http://www.jianshu.com/p/ab38818ecd83)

## Dockerfile

+ [Dockerfile文件详解](https://hujb2000.gitbooks.io/docker-flow-evolution/content/cn/basis/dockerfiledetail.html)

## docker工具

+ [docker搭建dns服务器](http://www.coderli.com/config-dnsmasq-using-docker/)

## Docker二进制安装方法

建议使用离线deb安装.

+ 最新官方版:https://docs.docker.com/engine/installation/binaries/    
+ 实验成功:http://www.cnblogs.com/jackluo/p/5441888.html (OK)

## 保持docker一直运行

使用`sleep infinity`命令启动容器
```
docker run -d --name images-name your_image sleep infinity
```

## 更改docker数据存储目录 ##

docker的默认存储路径为`/var/lib/docker`，但是有时候想更改目录：

+ Ubuntu/Debian:编辑`/etc/default/docker`文件，更改`DOCKER_OPTS="-dns 8.8.8.8 -dns 8.8.4.4 -g /mnt"`为目录.
+ Fedora/Centos: 编辑`/etc/sysconfig/docker`文件，更改参数为`other_args="-g /var/lib/testdir"`.


## Docker容器内不能访问外网

编辑系统网络配置

```
$ sudo gedit /etc/NetworkManager/NetworkManager.conf
```
注释以下行后重启

```
#dns=dnsmasq

$ sudo /etc/init.d/network-manager restart
```
