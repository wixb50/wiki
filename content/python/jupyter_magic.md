---
title: "jupyter magic"
date: 2017-07-09 12:30
collection: "jupyterhub"
---

[TOC]

## 介绍 ##

在 IPython 的会话环境中，所有文件都可以通过 %run 命令来当做脚本执行，并且文件中的变量也会随即导入当前命名空间。
即，对于一个模块文件，你对他使用 %run 命令的效果和 from module import * 相同,
这种以 % 开头的命令在 IPython 中被称为魔术命令，用于加强 shell 的功能。

## 常用的魔术命令 ##

常用的魔术命令有：

```
%quickref   显示 IPython 快速参考
%magic  显示所有魔术命令的详细文档
%debug  从最新的异常跟踪的底部进入交互式调试器
%pdb    在异常发生后自动进入调试器
%reset  删除 interactive 命名空间中的全部变量
%run script.py  执行 script.py
%load test.py 导入该段代码的cell中输入，支持http文件
%prun statement 通过 cProfile 执行对 statement 的逐行性能分析
%time statement 测试 statement 的执行时间
%timeit statement   多次测试 statement 的执行时间并计算平均值
%who、%who_ls、%whos  显示 interactive 命名空间中定义的变量，信息级别/冗余度可变
%xdel variable  删除 variable，并尝试清除其在 IPython 中的对象上的一切引用
%bookmark   使用 IPython 的目录书签系统
%cd direcrory   切换工作目录
%pwd    返回当前工作目录（字符串形式）
%env    返回当前系统变量（以字典形式）

!cmd    在系统 shell 执行 cmd
output=!cmd args    执行cmd 并赋值
```


**注意，jupyter notebook 是支持 TAB 键自动补充单词的。**

## 参考 ##
+ [jupyter notebook的安装与使用](http://blog.csdn.net/lee_j_r/article/details/52791228)
+ [27 个Jupyter Notebook的小提示与技巧](http://liuchengxu.org/pelican-blog/jupyter-notebook-tips.html)