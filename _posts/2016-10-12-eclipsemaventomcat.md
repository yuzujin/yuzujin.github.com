---
layout: post
title: "配置eclipse&maven&tomcat基础开发环境"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 安装eclipse

本次实验下载的是eclipse MARS release，版本，该版本已经集成里maven插件。


### 安装Java

根据系统版本（本文系统基于ios）选择合适的jdk版本下载安装，安装过程中会出现两次安装提示，第一次是安装jdk，第二次是安装jre。

安装完成后, 在.bash_profile中设置jdk环境变量：JAVA_HOME，修改Path变量

```
JAVA_HOME="`/usr/libexec/java_home`"
PATH=$JAVA_HOME/bin:$PATH
```
修改完成后，不要忘记运行source .bash_profile，使其生效

查看java是否安装配置成功，运行java -version

```
java version "1.8.0_91"
Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

配置jre路径：

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/_images/java.png)


### 安装maven

下载apache-maven-3.3.9-bin.tar.gz安装包，解压即可使用。

打开.bash_profile，配置maven环境变量：MAVEN_HOME

```
M3_HOME=/Users/test/app/maven3.3.9
PATH=$JAVA_HOME/bin:$M3_HOME/bin:$PATH
```

修改完成后，不要忘记运行source .bash_profile，使其生效

接下来在maven根目录下创建本地仓库目录：repo，同时打开maven下的conf/settings.xml，修改本地仓库路径：

```
<localRepository>/Users/test/app/maven3.3.9/repo</localRepository>
```

关闭保存

然后再eclipse中配置maven路径，如下图：

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/_images/maven.png)

配置settings.xml文件：

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/_images/maven2.png)

至此maven配置完成。


### 安装tomcat

下载apache-tomcat-8.0.33.tar.gz安装包，解压即可使用

打开eclipse配置：

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/_images/tomcat.png)

点击finish，配置完成。




