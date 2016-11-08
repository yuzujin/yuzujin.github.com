---
layout: post
title: "Apache和Nginx的rewrite功能浅析"
description: ""
category: 
tags: []
---
{% include JB/setup %}


## Nginx Rewrite规则相关指令

**Nginx Rewrite规则相关`指令`有**:

if        #条件判断

rewrite   #跳转

set       #对变量进行了赋值操作

return    #该指令将结束执行直接返回http状态码到客户端

**Nginx Rewrite规则相关`标记`有**：

redirect   #返回302临时重定向 地址栏会显示跳转后的地址

last       #相当于Apache里的[L]标记，表示完成rewrite（重新将rewrite后的地址在server标签中执行）

break      #终止匹配, 不再匹配后面的规则（将rewrite后的地址在当前location标签中执行）

permanent  #返回301永久重定向 地址栏会显示跳转后的地址

**一个简单的Nginx Rewrite规则语法如下**:

```
rewrite ^/b/(.*)\.html /play.php?video=$1 break;
如果加上if语句，示例如下： 
if (!-f $request_filename)
{ rewrite ^/img/(.*)$ /site/$host/images/$1 last;      }
```

**Nginx rewrite模块环境变量**

$args, 请求中的参数;

$content_length, HTTP请求信息里的"Content-Length";

$content_type, 请求信息里的"Content-Type";

$document_root, 针对当前请求的根路径设置值;

$document_uri, 与$uri相同;

$host, 请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名;

$limit_rate, 对连接速率的限制;

$request_method, 请求的方法，比如"GET"、"POST"等;

$remote_addr, 客户端地址;

$remote_port, 客户端端口号;

$remote_user, 客户端用户名，认证用;

$request_filename, 当前请求的文件路径名

$request_body_file

$request_uri, 请求的URI，带查询字符串;

$query_string, 与$args相同;

$scheme, 所用的协议，比如http或者是https，比如rewrite  ^(.+)$  $scheme://example.com$1  redirect;

$server_protocol, 请求的协议版本，"HTTP/1.0"或"HTTP/1.1";

$server_addr, 服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费);

$server_name, 请求到达的服务器名;

$server_port, 请求到达的服务器端口号;

$uri, 请求的URI，可能和最初的值有不同，比如经过重定向之类的。

> 这些变量可以用在rewrite规则里，也可以打印日志的时候用

## Apache Rewrite 规则相关指令

**Apache rewrite规则相关指令有**：

RewriteEngine on #打开rewirte功能

RewriteCond      #相当于nginx if

RewriteRule      #相当于nginx rewrite命令

**Apache rewrite规则相关`标记`有**：

1.  R[=code](force redirect)           #强制外部重定向, 强制在替代字符串加上http://thishost[:thisport]/前缀重定向到外部的URL.如果code不指定，将用缺省的302 HTTP状态码。
2.  F(force URL to be forbidden)       #禁用URL,返回403HTTP状态码。
3.  G(force URL to be gone)            #强制URL为GONE，返回410HTTP状态码。
4.  P(force proxy)                     #强制使用代理转发。
5.  L(last rule)                       #表明当前规则是最后一条规则，停止分析以后规则的重写。
6.  N(next round)                      #重新从第一条规则开始运行重写过程。
7.  C(chained with next rule)          #与下一条规则关联,如果规则匹配则正常处理，该标志无效，如果不匹配，那么下面所有关联的规则都跳过。
8.  T=MIME-type(force MIME type)       #强制MIME类型
9.  NS (used only if no internal sub-request)  #只用于不是内部子请求
10. NC(no case)                        #不区分大小写
11. QSA(query string append)           #追加请求字符串
12. NE(no URI escaping of output)      #不在输出转义特殊字符,例如：RewriteRule /foo/(.*) /bar?arg=P1\%3d$1 [R,NE] #将能正确的将/foo/zoo转换成/bar?arg=P1=zed
13. S=num(skip next rule(s))           #跳过num条规则
14. E=VAR:VAL(set environment variable)#设置环境变量
15. PT(pass through to next handler)   #传递给下一个处理, 例如：

```
    RewriteRule ^/abc(.*) /def$1 [PT] #将会交给/def规则处理
    Alias /def /ghi
``` 

**一个简单的Apache rewrite规则语法如下**：

```
RewriteEngine on
RewriteCond %{HTTP_HOST} ^haha.test.com [NC]
RewriteRule ^/(.*) http://hehe.test.com/ [L]
```

**Apache rewrite模块环境变量**

第一部分: HTTP headers 部分参数

参数名称: HTTP_USER_AGENT
样例参考值: Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9.0.8) Gecko/2009032609 Firefox/3.0.8
说明: 相当于PHP中的服务器参数: $_SERVER["HTTP_USER_AGENT"]

参数名称: HTTP_REFERER
样例参考值: http://www.test.cn/test.php
说明: 相当于PHP中的服务器参数: _SERVER["HTTP_REFERER"]

参数名称: HTTP_COOKIE
样例参考值: ZDEDebuggerPresent=php,phtml,php3
说明: 相当于PHP中的服务器参数: $_SERVER["HTTP_COOKIE"]

参数名称: HTTP_FORWARDED
样例参考值: 如果使用代理服务器的话会是代理服务器的IP地址, 本地不容易搭环境测试出值来.
说明: 相当于PHP中的服务器参数: $_SERVER["HTTP_FORWARDED"]

参数名称: HTTP_HOST
样例参考值: www.test.com
说明: 相当于PHP中的服务器参数: $_SERVER["HTTP_HOST"]

参数名称: HTTP_PROXY_CONNECTION
样例参考值: 网络连接代理方面的信息. 和HTTP_FORWARDED参数一样. 本地不容易搭环境测试出值来.
说明: PHP中貌似未提供这样的服务器信息值. 如果又的话可能等值于: $_SERVER["HTTP_PROXY_CONNECTION"]

参数名称: HTTP_ACCEPT
样例参考值: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
说明: 相当于PHP中的服务器参数: $_SERVER["HTTP_ACCEPT"]

第二部分: server internals 部分参数

参数名称: DOCUMENT_ROOT
样例参考值: C:/webRoot/t
说明: 相当于PHP中的服务器参数: $_SERVER["DOCUMENT_ROOT"]

参数名称: SERVER_ADMIN
样例参考值: youemailaddress@gmail.com
说明: 相当于PHP中的服务器参数: $_SERVER["SERVER_ADMIN"]

参数名称: SERVER_NAME
样例参考值: www.test.com
说明: 相当于PHP中的服务器参数: $_SERVER["SERVER_NAME"]

参数名称: SERVER_ADDR
样例参考值: 127.0.0.1
说明: 相当于PHP中的服务器参数: $_SERVER["SERVER_ADDR"]

参数名称: SERVER_PORT
样例参考值: 80
说明: 相当于PHP中的服务器参数: $_SERVER["SERVER_PORT"]

参数名称: SERVER_PROTOCOL
样例参考值: HTTP/1.1
说明: 相当于PHP中的服务器参数: $_SERVER["SERVER_PROTOCOL"]

参数名称: SERVER_SOFTWARE
样例参考值: Apache/2.2.11 (Win32) PHP/5.2.9-1
说明: 相当于PHP中的服务器参数: $_SERVER["SERVER_SOFTWARE"]

第三部分: connection & request 部分参数

参数名称: REMOTE_ADDR
样例参考值: 127.0.0.1 正在浏览当前页面用户的 IP 地址。
说明: 相当于PHP中的服务器参数: $_SERVER["REMOTE_ADDR"]

参数名称: REMOTE_HOST
样例参考值: 127.0.0.1 正在浏览当前页面用户的主机名。反向域名解析基于该用户的 REMOTE_ADDR
说明: 相当于PHP中的服务器参数: $_SERVER["REMOTE_HOST"]

参数名称: REMOTE_PORT
样例参考值: 2574 (变化的值)用户连接到服务器时所使用的端口
说明: 相当于PHP中的服务器参数: $_SERVER["REMOTE_PORT"]

参数名称: REMOTE_USER
样例参考值: 空
说明: PHP 好像未提供相关的$_SERVER值.

参数名称: REMOTE_IDENT
样例参考值: 空
说明: PHP 好像未提供相关的$_SERVER值.

参数名称: REQUEST_METHOD
样例参考值: GET
说明: 相当于PHP中的服务器参数: $_SERVER["REQUEST_METHOD"]

参数名称: SCRIPT_FILENAME
样例参考值: C:/webRoot/t/share77.html
说明: 相当于PHP中的服务器参数: $_SERVER["SCRIPT_FILENAME"]

参数名称: PATH_INFO
样例参考值: 空
说明: 相当于PHP中的服务器参数: $_REQUEST["PATH_INFO"]

参数名称: QUERY_STRING
样例参考值: a=b&c=d&e=f
说明: 相当于PHP中的服务器参数: $_SERVER["QUERY_STRING"]

参数名称: AUTH_TYPE
样例参考值: 空 当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个是认证的类型
说明: 相当于PHP中的服务器参数: $_SERVER["AUTH_TYPE"]

第四部分: date and time 部分参数

参数名称: TIME_YEAR
样例参考值: 2009
说明: 服务器获取当前的年份值

参数名称: TIME_MON
样例参考值: 04
说明: 服务器获取当前的月份值

参数名称: TIME_DAY
样例参考值: 22
说明: 服务器获取当前的日值

参数名称: TIME_HOUR
样例参考值: 16
说明: 服务器获取当前时间的小时

参数名称: TIME_MIN
样例参考值: 26
说明: 服务器获取当前时间的分钟

参数名称: TIME_SEC
样例参考值: 34
说明: 服务器获取当前时间的秒

参数名称: TIME_WDAY
样例参考值: 3
说明: 服务器获取当天是星期几, 从星期日-星期六, 数字从 0-6

参数名称: TIME
样例参考值: 20090422162634
说明: 服务器获取当前的时间, 格式为: 年月日时分秒

第四部分: specials 部分参数

参数名称: API_VERSION
样例参考值: 20051115:21
说明: apache 的 API 版本信息.

参数名称: THE_REQUEST
样例参考值: GET /share77.html HTTP/1.1
说明: 浏览器发给服务器的请求值. 不包括其他的头信息.

参数名称: REQUEST_URI
样例参考值: /share77.html
说明: 浏览器请求的资源信息.

参数名称: REQUEST_FILENAME
样例参考值: C:/webRoot/t/share77.html
说明: 被请求的资源的在磁盘的物理地址.

参数名称: IS_SUBREQ
样例参考值: false
说明: 如果是 sub-request 则显示为 true, 否则为 false.

参数名称: HTTPS
样例参考值: off
说明: 如果连接使用 SSL/TLS 模式, 则值为on , 否则值为off, 这个参数比较安全, 即使未载入 mod_ssl 模块时.

## Apache和Nginx实例对比

**Apache Rewrite**

RewriteRule ^/(test1|test2)/$ /yzj/index.php?name=$1 [L]
RewriteRule ^/test/$ /yzj/test.php [L]
RewriteRule ^/(test)_([a-zA-Z]+)/$ /yzj/index.php?name=$1_$2 [L] 
RewriteRule ^/test([0-9]*)/$ /yzj/test.php?id=$1 [L]

**Nginx Rewrite**

rewrite ^/(test1|test2)/$ /yzj/index.php?name=$1 last;
rewrite ^/test/$ /yzj/test.php last;
rewrite ^/(test)_([a-zA-Z]+)/$ /zl/index.php?name=$1_$2 last;
rewrite ^/test([0-9]*)/$ /yzj/test.php?id=$1 last;

## Apache与Nginx Rewrite规则的一些功能相同或类似的指令、标记对应关系


Apache的RewriteCond指令对应Nginx的if指令；

Apache的RewriteRule指令对应Nginx的rewrite指令；

Apache的[R]标记对应Nginx的redirect标记；

Apache的[P]标记对应Nginx的last标记；

Apache的[R,L]标记对应Nginx的redirect标记；

Apache的[P,L]标记对应Nginx的last标记；

Apache的[PT,L]标记对应Nginx的last标记；