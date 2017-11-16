---
title: "pip"
date: 2017-11-16 13:30
collection: "Tools"
---

[TOC]

## 修改PyPI源##

### 命令方式：针对一次使用，临时修改

+ easy_install

```
easy_install -i http://e.pypi.python.org/simple requests
```

+ pip

```
pip install requests -i http://e.pypi.python.org/simple
```

*注：1. 源路径要包含/simple部分；2. 使用pip时-i参数应放在install xxx的后面*

### 修改（若没有，则创建）easy_install/pip的配置文件

+ easy_install：在`~/.pydistutils.cfg`配置文件中写入如下内容：

```
[easy_install]
index_url = http://e.pypi.python.org/simple
```

+ pip：在`~/.pip/pip.conf`配置文件中写入：

```
[global]
index-url = http://e.pypi.python.org/simple
```