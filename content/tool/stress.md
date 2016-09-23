---
title: "stress"
date: 2016-09-13 09:10
collection: "其它"
---

## 介绍

顾名思义，stress是用来测试服务器的负载情况的压力测试工具。

## 使用

+ Stress参数说明

```
-? 显示帮助信息
-v 显示版本号
-q 不显示运行信息
-n，--dry-run 显示已经完成的指令执行情况
-t --timeout N 指定运行N秒后停止
   --backoff N 等待N微妙后开始运行
-c --cpu 产生n个进程 每个进程都反复不停的计算随机数的平方根
-i --io  产生n个进程 每个进程反复调用sync()，sync()用于将内存上的内容写到硬盘上
-m --vm n 产生n个进程,每个进程不断调用内存分配malloc和内存释放free函数
   --vm-bytes B 指定malloc时内存的字节数 (默认256MB)
   --vm-hang N 指示每个消耗内存的进程在分配到内存后转入休眠状态，与正常的无限分配和释放内存的处理相反，这有利于模拟只有少量内存的机器
-d --hadd n 产生n个执行write和unlink函数的进程
   --hadd-bytes B 指定写的字节数，默认是1GB
   --hadd-noclean 不要将写入随机ASCII数据的文件Unlink

时间单位可以为秒s，分m，小时h，天d，年y，文件大小单位可以为K，M，G
```

+ 产生13个cpu进程4个io进程1分钟后停止运行

```
$ stress -c 13 -i 4 --verbose --timeout 1m
```

## 参考

+ [Linux压力测试软件Stress使用指南](http://www.hi-linux.com/2016/06/27/Linux%E5%8E%8B%E5%8A%9B%E6%B5%8B%E8%AF%95%E8%BD%AF%E4%BB%B6Stress%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/)
