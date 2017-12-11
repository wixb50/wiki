---
title: "docker iptables"
date: 2017-16-26 16:32
collection: "Docker"
---

[TOC]

## 使用iptables限制docker网络访问

1. 以下的限制只对docker0生效，即默认的`docker run`启动的容器；
2. 如果容器启动的时候指定了`--net`则需要限制对应的network网关；
3. 如果需要限制`docker swarm`启动的services，则需要对`docker_gwbridge`进行限制；

### 常用iptables命令

+ 规则展示及筛选

```
iptables -L # 所有规则
iptables -L -v # 所有规则(详细信息)
iptables -L INPUT # 指定chain的规则
iptables -L --line-numbers # 显示line number所有规则
```

+ 删除规则

```
iptables -D INPUT 3 # 通过line number删除规则
iptables -D INPUT -m conntrack --ctstate INVALID -j DROP # 通过规则删除
iptables -F # 清空iptables规则
iptables -F INPUT # 删除指定chain的所有规则
```

### 限制docker容器访问外部网络  

+ 禁止容器访问局域网

```
iptables -I FORWARD -i docker0 -d 192.168.0.0/16 -j DROP
# 允许LAN网络连接容器(否则会被上一条拒绝)
iptables -I FORWARD -i docker0 -d 192.168.0.0/16 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

+ 限制绑定到指定network的容器(bridge)网络访问

```
# 获取网关名称
DOCKER_NET="my_network_name"
SUB_NET=`docker network inspect --format='{{range .IPAM.Config}}{{.Subnet}}{{end}}' $DOCKER_NET`
NET_GW=`ip route show $SUB_NET | cut -d\  -f3`
# 设置访问白名单
WHITE_LIST=(192.168.202.203 192.168.202.204)
#WHITE_LIST=(`cat white_list.txt`) # 或者从文件读取
iptables -I FORWARD -i $NET_GW -d 192.168.0.0/16 -j DROP
for line in "${WHITE_LIST[@]}"
do
  iptables -I FORWARD -i $NET_GW -d $line -j ACCEPT
done
```

+ 限制docker swarm启动的容器网络访问

```
NET_GW="docker_gwbridge"
# 设置访问白名单同上
```

### 限制外部对docker容器的访问

+ 限制部分ip访问容器

仅公开访问http和https。
两个管理IP被授予访问所有正在运行的容器。
两个局域网IP被授予访问所有正在运行的容器。
其他一切都将被拒绝。

```
#!/usr/bin/env bash

# Usage:
# timeout 10 docker_iptables.sh
#
# Use the builtin shell timeout utility to prevent infinite loop (see below)

if [ ! -x /usr/bin/docker ]; then
    exit
fi

# Check if the PRE_DOCKER chain exists, if it does there's an existing reference to it.
iptables -C FORWARD -o docker0 -j PRE_DOCKER

if [ $? -eq 0 ]; then
    # Remove reference (will be re-added again later in this script)
    iptables -D FORWARD -o docker0 -j PRE_DOCKER
    # Flush all existing rules
    iptables -F PRE_DOCKER
else
    # Create the PRE_DOCKER chain
    iptables -N PRE_DOCKER
fi

# Default action
iptables -I PRE_DOCKER -j DROP

# Docker Containers Public Admin access (insert your IPs here)
iptables -I PRE_DOCKER -i eth0 -s 192.184.41.144 -j ACCEPT
iptables -I PRE_DOCKER -i eth0 -s 120.29.76.14 -j ACCEPT

# Docker Containers Restricted LAN Access (insert your LAN IP range or multiple IPs here)
iptables -I PRE_DOCKER -i eth1 -s 192.168.1.101 -j ACCEPT
iptables -I PRE_DOCKER -i eth1 -s 192.168.1.102 -j ACCEPT

# Docker internal use
iptables -I PRE_DOCKER -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -I PRE_DOCKER -i docker0 ! -o docker0 -j ACCEPT
iptables -I PRE_DOCKER -m state --state RELATED -j ACCEPT
iptables -I PRE_DOCKER -i docker0 -o docker0 -j ACCEPT

# Docker container named www-nginx public access policy
WWW_IP_CMD="/usr/bin/docker inspect --format='{{.NetworkSettings.IPAddress}}' www-nginx"
WWW_IP=$($WWW_IP_CMD)

# Double check, wait for docker socket (upstart docker.conf already does this)
while [ ! -e "/var/run/docker.sock" ]; do echo "Waiting for /var/run/docker.sock..."; sleep 1; done

# Wait for docker web server container IP
while [ -z "$WWW_IP" ]; do echo "Waiting for www-nginx IP..."; WWW_IP=$($WWW_IP_CMD); done

# Insert web server container filter rules
iptables -I PRE_DOCKER -i eth0 -p tcp -d $WWW_IP --dport 80  -j ACCEPT
iptables -I PRE_DOCKER -i eth0 -p tcp -d $WWW_IP --dport 443 -j ACCEPT

# Finally insert the PRE_DOCKER table before the DOCKER table in the FORWARD chain.
iptables -I FORWARD -o docker0 -j PRE_DOCKER
```

## 参考资料

+ [Disable access to LAN from docker container](https://stackoverflow.com/questions/27207397/disable-access-to-lan-from-docker-container)
+ [docker-restricting-container-access-with-iptables](http://rudijs.github.io/2015-07/docker-restricting-container-access-with-iptables/)


