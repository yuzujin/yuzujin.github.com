---
layout: post
title: "在aws ec2实例上搭建shadowsocks服务"
description: ""
category: 
tags: []
---
{% include JB/setup %}

**Shadowsocks**一个可穿透防火墙的快速代理。

-------------

## 创建aws ec2实例

1. 选择t2.micro实例（可以免费使用一年，当然首先你得需要一个aws账号和信用卡信息），镜像选择AMI（实际上是centos系统）

2. 安全策略中打开tcp端口（ssh:22、shadowsocks:8388）

## 服务端安装&配置&启动

1. 安装

   ```
   yum install -y python-setuptools
   easy_install pip
   pip install shadowsocks
   ```
   
2. 配置

   创建配置文件 vi /etc/shadowsocks/config.json
   
   ```
   {
   	"server":"0.0.0.0",
   	"server_port":8388,
   	"local_address":"127.0.0.1",
   	"local_port":1080,
   	"password":"123",
   	"timeout":300,
   	"method":"aes-256-cfb",
   	"fast_open":false,
   	"workers": 1
   }
   ```
   各个字段代表的含义如下：
   
   ```
   server: 服务器监听地址
   server_port: 服务器端口
   local_address: 本地监听地址
   local_port: 本地端口
   password: 密码
   timeout: 超时时间
   method: 加密算法
   fast_open: 使用TCP_FASTOPEN，true/false
   workers: 工作线程数，Unix/Linux可用
   ```
   
3. 启动
   `ssserver -c /etc/shadowsocks.json -d start`

## 客户端安装配置
1. windows安装shadowsocks客户端

   -**下载地址**：https://github.com/shadowsocks/shadowsocks-windows/releases
   
   -**服务器地址**：aws 实例公用ip
   
   -**服务器端口**：ssserver启动的端口
   
   -**密码**：shadowsocks配置的密码(123)
   
2. chrome安装switchysharp插件

   代理协议：socks5 代理服务器：127.0.0.1 代理端口：1080
   
## 访问youtube
www.youtube.com 成功。

参考链接：http://www.cnblogs.com/wwlzz/p/4321998.html