---
layout: post
title: "查看Apache的connection数"
description: ""
category: 
tags: []
---
{% include JB/setup %}

下面介绍一下查看Apache的连接数和当前的连接数以及IP访问次数的常用命令。

查看了连接数和当前的连接数：

```
netstat -ant | grep $ip:80 | wc -l 
netstat -ant | grep $ip:80 | grep EST | wc -l 
```

查看IP访问次数:

```
netstat -nat|grep ":80"|awk '{print $5}' |awk -F: '{print $1}' | sort| uniq -c|sort -n
```

查看当前连接情况：

```
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}' 
watch "netstat -n | awk '/^tcp/ {++S[\$NF]} END {for(a in S) print a, S[a]}'"
```

返回结果示例：

```  
SYN_RECV 30           #表示正在等待处理的请求数；
ESTABLISHED 1597      #表示正常数据传输状态；
FIN_WAIT1 51          #表示server端主动要求关闭tcp连接；
FIN_WAIT2 504         #表示客户端中断连接；
TIME_WAIT 1057        #表示处理完毕，等待超时结束的请求数；
LAST_ACK 5            #关闭一个TCP连接需要从两个方向上分别进行关闭，双方都是通过发送FIN来表示单方向数据的关闭，当通信双方发送了最后一个FIN的时候，发送方此时处于LAST_ACK状态，当发送方收到对方的确认（Fin的Ack确认）后才真正关闭整个TCP连接；
```

ESTABLISHED 表示正在通信，TIME_WAIT 表示主动关闭，CLOSE_WAIT 表示被动关闭

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/connection.gif)

一般不到万不得已的情况也不会去查看网络状态，如果服务器出了异常，百分之八九十都是下面两种情况：

1. 服务器保持了大量TIME_WAIT状态
2. 服务器保持了大量CLOSE_WAIT状态

因为linux分配给一个用户的文件句柄是有限的，而TIME_WAIT和CLOSE_WAIT两种状态如果一直被保持，那么意味着对应数目的通道就一直被占着，一旦达到句柄数上限，新的请求就无法被处理了，接着就是大量Too Many Open Files异常，tomcat崩溃。。。
