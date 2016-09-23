---
title: "siege"
date: 2016-09-23 09:10
collection: "其它"
---

## 介绍

 linux下siege 网站压力测试机

## 使用

+ 安装

```
sudo apt-get install siege
```

+ 参数说明

```
-c NUM 设置并发的用户（连接）数量，比如-c10,设置并发10个连接。默认的连接数量可以到~/.siegerc中查看，指令为concurrent = x，前面咱们已经调整了默认并发连接为50。
-r NUM(repetitions)，重复数量，即每个连接发出的请求数量，设置这个的话，就不需要设置-t了。对应.siegerc配置文件中的reps = x指令
-t NUM(time)，持续时间，即测试持续时间，在NUM时间后结束，单位默认为分，比如-t10，那么测试时间为10分钟，-t10s，则测试时间为10秒钟。对应.siegerc中的指令为time = x指令
-b (benchmark),基准测试，如果设置这个参数的话，那么delay时间为0。在.siegerc中咱们修改为默认开启。
-f url.txt (file),这是url列表文件。对应.siegerc配置文件中的file = x指令
```

+ 测试结果分析

```
$ siege -c 500 -r 50 -f url
** SIEGE 2.67
** Preparing 500 concurrent users for battle.
The server is now under siege..      done.
Transactions:                  25000 hits           #意思是总共完成了25000次测试
Availability:                 100.00 %              #测试的有效性100%
Elapsed time:                  65.52 secs           #用时65.52秒
Data transferred:              83.65 MB             #传输了83.65MB数据
Response time:                  0.57 secs           #响应时间
Transaction rate:             381.56 trans/sec      #每秒传输381.56次
Throughput:                     1.28 MB/sec         #数据吞吐量每秒1.28MB
Concurrency:                  216.02                #实际并发访问
Successful transactions:       21707                #成功的传输
Failed transactions:               0                #失败的传输
Longest transaction:            5.83                #每次传输所花最长时间
Shortest transaction:           0.00                #每次传输所花最短时间
```

+ 使用示例

```
$ siege -c 20 -t 10 http://www.baidu.com
```
过一会百度就把你ip封了。

## 参考

+ [siege 网站压力测试](https://github.com/erasin/notes/blob/master/linux/soft/siege.md)
