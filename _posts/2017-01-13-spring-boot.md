---
layout: post
title: "Spring Boot入门"
description: ""
category: 
tags: []
---
{% include JB/setup %}

## Why Spring Boot?

---

Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。

多年以来，Spring IO平台饱受非议的一点就是大量的XML配置以及复杂的依赖管理。Spring Boot并不是要成为Spring IO平台里面众多“Foundation”层项目的替代者。Spring Boot的目标不在于为已解决的问题域提供新的解决方案，而是为平台带来另一种开发体验，从而简化对这些已有技术的使用。

Spring Boot，甚至可以说整个Spring生态系统都使用到了Groovy编程语言。Boot所提供的众多便捷功能，都是借助于Groovy强大的MetaObject协议、可插拔的AST转换过程以及内置的依赖解决方案引擎所实现的。在其核心的编译模型之中，Boot使用Groovy来构建工程文件，所以它可以使用通用的导入和样板方法（如类的main方法）对类所生成的字节码进行装饰（decorate）。这样使用Boot编写的应用就能保持非常简洁，却依然可以提供众多的功能。

## What's Spring Boot?

---

从最根本上来讲，Spring Boot就是一些库的集合，它能够被任意项目的构建系统所使用。简便起见，该框架也提供了命令行界面，它可以用来运行和测试Boot应用。框架的发布版本，包括集成的CLI（命令行界面），可以在Spring仓库中手动下载和安装。一种更为简便的方式是使用Groovy环境管理器（Groovy enVironment Manager，GVM），它会处理Boot版本的安装和管理。Boot及其CLI可以通过GVM的命令行gvm install springboot进行安装。

要进行打包和分发的工程会依赖于像Maven或Gradle这样的构建系统。为了简化依赖图，Boot的功能是模块化的，通过导入Boot所谓的“starter”模块，可以将许多的依赖添加到工程之中。为了更容易地管理依赖版本和使用默认配置，框架提供了一个parent POM，工程可以继承它。Spring Boot工程的样例POM文件定义如下所示：

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>

    	<groupId>com.example</groupId>
    	<artifactId>myproject</artifactId>
    	<version>1.0.0-SNAPSHOT</version>

    	<!-- Inherit defaults from Spring Boot -->
    	<parent>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-starter-parent</artifactId>
        	<version>1.0.0.RC1</version>
    	</parent>

    	<!-- Add typical dependencies for a web application -->
    	<dependencies>
        	<dependency>
            	<groupId>org.springframework.boot</groupId>
            	<artifactId>spring-boot-starter-web</artifactId>
        	</dependency>
        	<dependency>
            	<groupId>org.springframework.boot</groupId>
            	<artifactId>spring-boot-starter-actuator</artifactId>
        	</dependency>
    	</dependencies>

    	<repositories>
        	<repository>
            	<id>spring-snapshots</id>
            	<url>http://repo.spring.io/libs-snapshot</url>
        	</repository>
    	</repositories>

    	<pluginRepositories>
        	<pluginRepository>
            	<id>spring-snapshots</id>
            	<url>http://repo.spring.io/libs-snapshot</url>
        	</pluginRepository>
    	</pluginRepositories>

    	<build>
        	<plugins>
            	<plugin>
                	<groupId>org.springframework.boot</groupId>
                	<artifactId>spring-boot-maven-plugin</artifactId>
            	</plugin>
        	</plugins>
    	</build>
	</project>
	

## How to use?

---

**使用CLI构建**

Spring Boot在刚刚公开宣布之后就将一个样例发布到了Twitter上，它目前成为了最流行的一个应用样例。它的全部描述如程序清单1.2所示，一个非常简单的Groovy文件可以生成功能强大的以Spring为后端的web应用。

	@RestController
	class App {
		@RequestMapping("/")
		String home() {
    		"hello"
    	}
	}
	
这个应用可以通过spring run App.groovy命令在Spring Boot CLI中运行。Boot会分析文件并根据各种“编译器自动配置（compiler auto-configuration）”标示符来确定其意图是生成Web应用。然后，它会在一个嵌入式的Tomcat中启动Spring应用上下文，并且使用默认的8080端口。

Boot能够自动确定类所需的功能，这一点使其成为了强大的快速应用开发工具。当应用在Boot CLI中执行时，它们在使用内部的Groovy编译器进行构建，这个编译器可以在字节码生成的时候以编码的方式探查并修改类。通过这种方式，使用CLI的开发人员不仅可以省去定义默认配置，在一定程度上甚至可以不用定义特定的导入语句，它们可以在编译的过程中识别出来并自动进行添加。除此之外，当应用在CLI中运行时，Groovy内置的依赖管理器，“Grape”，将会解析编译期和运行时的类路径依赖，与Boot编译器的自动配置机制类似。这种方式不仅使得框架更加对用户友好，而且能够让不同版本的Spring Boot与特定版本的来自于Spring IO平台的库相匹配，这样一来开发人员就不用关心如何管理复杂的依赖图和版本结构了。另外，它还有助于快速原型的开发并生成概念原型的工程代码。

**非CLI构建**

对于不是使用CLI构建的工程，Boot提供了许多的“starter”模块，它们定义了一组依赖，这些依赖能够添加到构建系统之中，从而解析框架及其父平台所需的特定类库。例如，spring-boot-starter-actuator依赖会引入一组基本的Spring项目，从而实现应用的快速配置和即时可用。关于这种依赖，值得强调的一点就是当开发Web应用，尤其是RESTful Web服务的时候，如果包含了spring-boot-starter-web依赖，它就会为你提供启动嵌入式Tomcat容器的自动化配置，并且提供对微服务应用有价值的端点信息，如服务器信息、应用指标（metrics）以及环境详情。除此之外，如果引入spring-boot-starter-security模块的话，actuator会自动配置Spring Security，从而为应用提供基本的认证以及其他高级的安全特性。它还会为应用结构引入一个内部的审计框架，这个框架可以用来生成报告或其他的用途，比如开发认证失败的锁定策略。

为了阐述在Java Maven工程中，如何快速地使Spring Web工程准备就绪，如下所示：

	package com.infoq.springboot;

	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
	import org.springframework.web.bind.annotation.*;

	@RestController
	@EnableAutoConfiguration
	public class Application {

		@RequestMapping("/")
		public String home() {
    		return "Hello";
    	}

		public static void main(String[] args) {
    		SpringApplication.run(Application.class, args);
    	}
	}
	
在Application类上的@EnableAutoConfiguration注解会告知Boot要采用一种特定的方式来对应用进行配置。这种方法会将其他样板式的配置均假设为框架默认的约定，因此能够聚焦于如何尽快地使应用准备就绪以便运行起来。Application类是可运行的，因此，当我们以Java Application的方式运行这个类时，就能启动该应用及其嵌入式的容器，这样也能实现即时地开发。

为了发布版本而构建工程时，Boot的Maven和Gradle插件可以嵌入（hook）到这些构建系统的打包过程中，以生成可执行的“胖jar包（fat jar）”，这种jar包含了工程的所有依赖并且能够以可运行jar的方式执行。使用Maven打包Boot应用只需运行mvn package命令，与之类似，使用Gradle时，执行gradle build命令将会在构建的目标地址下生成可运行的jar。

## Use in MicroService

---

Boot对Spring应用的开发进行了简化，提供了模块化方式导入依赖的能力，强调了开发RESTful Web服务的功能并提供了生成可运行jar的能力，这一切都清晰地表明在开发可部署的微服务方面Boot框架是一个强大的工具。


[Spring Boot Reference Guide](http://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/reference/htmlsingle/)

