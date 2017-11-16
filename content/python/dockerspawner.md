---
title: "DockerSpawner"
date: 2016-06-27 13:30
collection: "Jupyterhub"
---

[TOC]

## 介绍 ##

jupyter 是把 IPython 和 Python 解释器剥离后的产物，将逐渐替代 IPython 独立发行。jupyter 可以和 Python 之外的 程序结合，提供新的、强大的服务。详见：[IPython 与 jupyter 相关概念介绍](http://blog.windrunner.info/python/jupyter.html)

## 安装 ##

安装Python3

```
$ sudo apt-get install python3-pip
```

安装npm/nodejs

```
$ sudo apt-get install npm nodejs-legacy
```

安装代理模块

```
$ npm install -g configurable-http-proxy
```

安装Jupyterhub

```
$ pip3 install jupyterhub
```

升级或者安装jupyter notebook

```
$ pip3 install --upgrade notebook
```

生成配置文件

```
$ jupyterhub --generate-config
```

编辑配置文件`jupyterhub_config`

```
$ sudo vim jupyterhub_config.py
# 配置如下
c.JupyterHub.spawner_class = 'dockerspawner.DockerSpawner'
c.LocalAuthenticator.create_system_users = False
c.Authenticator.whitelist = {'int'} # 可以登陆的用户，需要同步PAM用户
c.Authenticator.admin_users = {'int'}
c.Spawner.notebook_dir = '~/work' # 指定jupyter容器的工作目录
c.JupyterHub.hub_ip = '172.17.0.1'
c.DockerSpawner.container_image = 'myhexinjupyterhub/singleuser' # 使用自己的配置好的镜像,基于jupyterhub/singleuser构建的增加zipline包
```

安装dockerspawner模块

```
$ git clone https://github.com/jupyterhub/dockerspawner.git
$ cd dockerspawner
$ pip3 install -r requirements.txt
$ python setup.py install
```

运行，登陆后自动生成用户的基于docker的jupyter notebook

```
jupyterhub --no-ssl
```

## 参考 ##
+ [Installation of Jupyterhub on remote server](https://github.com/jupyterhub/jupyterhub/wiki/Installation-of-Jupyterhub-on-remote-server)
+ [DockerSpawner Github](https://github.com/jupyterhub/dockerspawner)