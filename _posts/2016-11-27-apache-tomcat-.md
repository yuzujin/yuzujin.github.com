---
layout: post
title: "Apache Tomcat 安全配置"
description: ""
category: 
tags: []
---
{% include JB/setup %}

## telnet管理端口保护

*配置内容*

	修改默认的 8005 管理端口为不易猜测的端口（大于 1024）；
	修改 SHUTDOWN指令为其他字符串；

*示例(server.xml)*

	<Server port="8520" shutdown="dangerous">
	

## ajp连接端口保护

*配置内容*

	修改默认的 ajp 8009 端口为不易冲突的大于1024 端口；
	通过 iptables 规则限制ajp 端口访问的权限仅为线上机器(可选)；
	
*示例(server.xml)*

	<Connector port="8521" protocol="AJP/1.3" />
    
## 禁用管理端

*配置内容*
	
	删除默认的{Tomcat安装目录}/conf/tomcat-users.xml 文件，重启 tomcat后将会自动生成新的文件；
	删除{Tomcat 安装目录}/webapps 下默认的所有目录和文件；
	将 tomcat 应用根目录配置为 tomcat 安装目录以外的目录；
	热部署可以关掉，防止被黑客注入木马程序
	
*示例*

    <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
    
## 降权启动

*配置内容*

	tomcat 启动用户权限必须为非 root 权限，尽量降低 tomcat 启动用户的目录访问权限；
	如需直接对外使用 80端口，可通过普通账号启动后，配置 iptables规则进行转发；
	
*示例*
    
    无
   
## 文件列表访问控制

*配置内容*

	1.conf/web.xml 文件中default 部分 listings 的配置必须为 false；
	
*示例*

	<init-param> 
		<param-name>listings</param-name> 
		<param-value>false</param-value> 
	</init-param>
	
## WEB-INF 目录访问控制

*配置内容*

	Tomcat web 目录里的 WEB-INF 目录默认为禁止访问，但是当tomcat 组合其他webserver 一起使用，并且其他 webserver 和tomcat 指向同一目录时，需要在其他webserver 配置文件里添加禁止访问/WEB-INF 目录
	
*示例*

	Apache：
	<Directory "WEB-INF">
		Order allow,deny
		Deny from all
	</Directory>
	
	Nginx：
	location ^~ /WEB-INF
	{
 		deny all;
	}
	
	Lighttpd:
	$HTTP["url"] =~ "^/WEB-INF/" {
 		url.access-deny = ( "" )
	}

## 版本信息隐藏

*配置内容*

	修改conf/web.xml，重定向 403、404 以及 500等错误到指定的错误页面(全局)；
	
	也可以通过修改应用程序目录下的WEB-INF/web.xml下的配置进行错误页面的重定向（应用级别）；

*示例*

	<error-page>
		<error-code>403</error-code>
		<location>/forbidden.jsp</location>
	</error-page>
	<error-page>
		<error-code>404</error-code>
		<location>/notfound.jsp</location>
	</error-page>
	<error-page>
		<error-code>500</error-code>
		<location>/systembusy.jsp</location>
	</error-page>
	
## Server header重写

*配置内容*

	在 HTTP Connector 配置中加入 server 的配置；
	
	当 tomcat HTTP 端口直接提供 web服务时此配置生效，加入此配置，将会替换 http 响应Server header部分的默认配置，默认是Apache-Coyote/1.1
	
*示例(conf/server.xml)*
	
	<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               server="webserver" />
               
## 访问限制

*配置内容*

	通过配置，限定访问的 ip 来源
	通过配置信任ip 的白名单，拒绝非白名单 ip的访问，此配置主要是针对高保密级别的系统，一般产品线不需要；
	Tomcat供了两个参数供你配置：RemoteHostValve 和RemoteAddrValve，前者用于限制主机名，后者用于限制IP地址。
	重启生效
	
*示例(conf/server.xml新增一条)*

    全局设置（对Tomcat下所有应用生效）
    <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
    	<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="68.147.18.138,68.135.165.*" deny="*.*.*.*"/>
    	#<Valve className="org.apache.catalina.valves.RemoteHostValve" allow="abc.com" deny=""/>
    </Host>
     
    
    局部设置 (仅对具体的应用生效)
	<Context path="/test" docBase="/home/work/tomcat/webapps/test" debug="0" reloadable="false" crossContext="true">
	<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="68.147.18.138,68.135.165.*" deny="*.*.*.*"/>
	</Context>
	
## 起停脚本权限回收

*配置内容*

	去除其他用户对 Tomcat的 bin 目录下shutdown.sh、startup.sh、catalina.sh 的可执行权限；
	
*示例*

	chmod -R 744 tomcat/bin/* 
	
## 访问日志格式规范

*配置内容*

	开启 Tomcat默认访问日志中的Referer 和User-Agent记录

*示例（conf/server.xml）*

	全局设置
	<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
		<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="localhost_access_log." suffix=".txt" pattern="%h %l %u %t %r %s %b %{Referer}i %{User-Agent}i %D" resolveHosts="false"/>
	</Host>
	局部设置
	参考上面的访问限制小节