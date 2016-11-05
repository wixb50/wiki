---
title: "nfs"
date: 2016-11-05 16:50
collection: "File服务器"
---

[TOC]

## 配置使用

### 服务器安装配置

```
apt install nfs-kernel-server
```
编辑/etc/exports文件，添加：
```
/srv/homes (rw)
```

### 客户端安装配置

```
apt install nfs-common
```
客户端挂载服务器共享资源：
```
mount -o ro 10.100.0.30:/srv/homes /mnt/share
或者
mount -o rw 10.100.0.30:/srv/homes /mnt/share
```
自动挂载编辑/etc/fstab：
```
# nfs
/svr/homes /mnt/share nfs defaults 0 0
```


参考：

+ [debian NFS服务配置](http://openwares.net/linux/debian-nfs-setup.html)
