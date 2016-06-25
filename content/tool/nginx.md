---
title: "Nginx"
date: 2016-01-07 21:11
updated: 2016-04-18 08:50
collection: "Web服务器"
log: "增加access_log的变量问题"
---

[TOC]

[Nginx Beginner's Guide](http://nginx.org/en/docs/beginners_guide.html)

## 指令 ##


### 核心命令 ###

见[Core functionality](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)

* `worker_processes`: (number | auto) 表示Nginx开启的子进程数


### server_name ###

见[Server names](http://nginx.org/en/docs/http/server_names.html)


### location ###

具体见官方文档 [ngx_http_core_module - location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location), 讲得非常详细。

基本语法如下:

	location [ = | ~ | ~* | ^~  ] uri { ...  }

主要分两种:

* prefix string, 普通的前缀字符串匹配。包括 `=`, `^~`
* regular expression, 正则匹配, 包括 `~*`, `~`

Nginx寻找匹配路径的逻辑:

> To find location matching a given request, nginx first checks locations defined using the prefix strings (prefix locations). Among them, the location with the longest matching prefix is selected and remembered. Then regular expressions are checked, in the order of their appearance in the configuration file. The search of regular expressions terminates on the first match, and the corresponding configuration is used. If no match with a regular expression is found then the configuration of the prefix location remembered earlier is used.

> If the longest matching prefix location has the “^~” modifier then regular expressions are not checked.

> Also, using the “=” modifier it is possible to define an exact match of URI and location. If an exact match is found, the search terminates.

即，会先寻找prefix string的**最长匹配**的路径并**记录**下来，然后再在正则路径中以**配置的顺序**寻找，如果正则路径有匹配，则使用正则location; 否则使用prefix string中之前记录的匹配location。

但是有两个例外，`^~`的作用就是不继续尝试做正则匹配，即如果最长匹配到这个location, 则停止继续尝试正则的location；还有一个是精确匹配`=`。

具体例子, 假设域名是a.com:

	location = / {
		[ configuration A ]
		# 精确匹配 (exact match)
		# 只匹配 a.com/
		# 优先级最高
	}

	location / {
		[ configuration B ]
		# 普通的prefix string匹配, 优先级最低
	}

	location /documents/ {
		[ configuration C ]
		# 普通的prefix string匹配, 优先级最低
	}

	location ^~ /images/ {
		[ configuration D ]
		# prefix string匹配, 如果匹配上, 则不再尝试去做正则匹配
	}

	location ~* \.(gif|jpg|jpeg)$ {
		[ configuration E ]
		# 正则匹配, 忽略大小写 (case-insensitive)
	}

	location ~ \.(PNG|TXT)$ {
		[ configuration E ]
		# 正则匹配, 大小写敏感 (case-sensitive)
	}

所以按优先级来说:

1. `=` exact match
2. `^~` prefix string match
3. `~*`/`~` re match
4. common prefix string match

最后，还有几个问题:

1. 如Mac OS X系统，默认是HFS+文件系统，是大小写不敏感的, 做location匹配时是忽略大小写。
2. 因为如`=`是做精确匹配后就停止，所以效率高。如，如果访问 '/' 非常频繁，可以考虑配置为 `= /`

但是，针对第二点，涉及到一个 `internal redirect` 的问题，具体看[index指令](http://nginx.org/en/docs/http/ngx_http_index_module.html#index)

> It should be noted that using an index file causes an internal redirect, and the request can be processed in a different location. For example, with the following configuration:
> 
> 	location = / {
> 		index index.html;
> 	}
> 
> 	location / {
> 		...
> 	}
> 
> a “/” request will actually be processed in the second location as “/index.html”.

如果配置为目录，会导致与与其不符的情况，这块需要注意。

本地编译Nginx打开debug选项，然后`error_log`配置为`debug`级别:

	error_log /var/log/nginx/test.error.log debug;

可以看到实际寻找location的顺序和一些行为:

	2016/04/12 18:47:27 [debug] 24990#0: *6 test location: "/"
	2016/04/12 18:47:27 [debug] 24990#0: *6 using configuration "=/"
	2016/04/12 18:47:27 [debug] 24990#0: *6 http cl:-1 max:1048576
	...
	2016/04/12 18:47:27 [debug] 24990#0: *6 open index "/opt/test/2015/index.html"
	2016/04/12 18:47:27 [debug] 24990#0: *6 internal redirect: "/index.html?"
	2016/04/12 18:47:27 [debug] 24990#0: *6 rewrite phase: 1
	2016/04/12 18:47:27 [debug] 24990#0: *6 test location: "/"
	2016/04/12 18:47:27 [debug] 24990#0: *6 test location: "images/"
	2016/04/12 18:47:27 [debug] 24990#0: *6 test location: ~ "\.(gif|jpg|jpeg|txt)$"
	2016/04/12 18:47:27 [debug] 24990#0: *6 using configuration "/"
	...
	2016/04/12 18:47:27 [debug] 24990#0: *6 http filename: "/opt/test/index.html"

比如我同时配置`= /` 和 `/`, root分别是/opt/test/2015/和/opt/test/, 访问根预期是在前者找, 但实际是在后者找了。而index默认是有值的。

其它参考:

* [Understanding Nginx Server and Location Block Selection Algorithms](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms)
* [nginx配置location总结及rewrite规则写法](http://seanlook.com/2015/05/17/nginx-location-rewrite/)
* [Nginx location 配置踩坑过程分享](https://blog.coding.net/blog/tips-in-configuring-Nginx-location)
* [How to mak nginx to stop processing other rules and serve a specific location?](http://stackoverflow.com/questions/10699755/how-to-mak-nginx-to-stop-processing-other-rules-and-serve-a-specific-location)


### root vs alias ###

注意 `alias` 配置的路径需要带上结束的**slash**, 否则做路径拼接会出错导致找不到资源。

另外，官方文档推荐使用`root`。

* [Nginx — static file serving confusion with root & alias](http://stackoverflow.com/questions/10631933/nginx-static-file-serving-confusion-with-root-alias)
* [nginx目录设置 alias 和 root](http://www.wkii.org/nginx-set-directory-alias-and-root.html)
* [nginx虚拟目录(alias与root的区别)](http://blog.sina.com.cn/s/blog_6c2e6f1f0100l92h.html)


### HTTP Basic Authentication ###

见 [ngx_http_auth_basic_module](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)，相关指令:

* `auth_basic`: 任意字符串(开启) 或 off (关闭, 默认值)
* `auth_basic_user_file`: 用户名:密码 列表文件

创建用户密码对的文件:

	$ htpasswd -c /path/to/auth/file <username>

注意: `-c` 是生成用户密码对并(覆盖)写入文件; 不加`-c`是基于现有文件做append。所以加-c要注意别覆盖已有文件了。

`htpasswd` 命令Gentoo下在`app-admin/apache-tools`包里。

参考:

* [How To Set Up HTTP Authentication With Nginx On Ubuntu 12.10](https://www.digitalocean.com/community/tutorials/how-to-set-up-http-authentication-with-nginx-on-ubuntu-12-10)
* [How To Set Up Password Authentication with Nginx on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-nginx-on-ubuntu-14-04)
* [How To Set Up Basic HTTP Authentication With Nginx on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-set-up-basic-http-authentication-with-nginx-on-centos-7)


### access_log的变量问题 ###

因为习惯把nginx日志按网站名存放, 即/var/log/nginx/www.xxx.com.log，所以考虑可以改为直接使用`server_name`:

	access_log  /var/log/nginx/$server_name.log;

类似与`proxy_pass`用变量配置域名，变量在运行时会做解析。所以:

nginx会在每次请求过来时打开日志文件，然后在请求结束时关闭文件(strace看)；而不使用变量情况下，会在进程起来时就一直打开文件(lsof看)。前者是非常消耗资源。(TODO 解决方案?)

还有另外一个，就是配置变量后，是每次靠worker process打开，所以日志的权限需要和worker process一样，否则无法写入日志；而后者是master process打开的日志文件，所以日志文件的权限是root也可以。


## 其它 ##

### 中文域名 ###

中文域名之所以可以解析/访问, 并不是说dns, nginx等都可以识别, 而是再浏览器层面会做一个转码.

这个转码技术是[Punycode](https://en.wikipedia.org/wiki/Punycode), 将Unicode码转为ASCII码.

其实不光是中文域名, 只要支持的Unicode编码的域名, 都可以经过Punycode转码.

这种域名一般称为[IDN](https://en.wikipedia.org/wiki/Internationalized_domain_name) (Internationalized domain name).

相关的rfc规定一般转为`xn--<puny>.<suffix>`这样的格式. 如`中国.com`, 转码后是`xn--fiqs8s.com`

	$ python -c 'import sys;print sys.argv[1].decode("utf-8").encode("idna")' "中国.com"
	xn--fiqs8s.com

也可以安装额外的工具, 如Gentoo下`net-dns/libidn`, [参考](http://serverfault.com/questions/335073/how-can-i-convert-an-idn-to-punicode-in-bash).

浏览器也可以直接看到, 如Chrome下点击地址栏最左边的"View site information".

相应的在nginx的配置中, server_name不能直接写中文域名, 应该写解析后的ascii编码的英文域名, 如:

	server
	{
		listen			80;
		server_name		xn--fiqs8s.com;
	}


## 链接 ##

* [如何正确配置Nginx+PHP](http://huoding.com/2013/10/23/290)
* [实战Nginx与PHP（FastCGI）的安装、配置与优化](http://ixdba.blog.51cto.com/2895551/806622)
* [Pitfalls and Common Mistakes](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/)

书籍:

* [实战Nginx](https://book.douban.com/subject/4251875/) 花了几个小时看完，还是有些收获，给7分
