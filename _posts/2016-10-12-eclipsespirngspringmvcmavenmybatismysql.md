---
layout: post
title: "在Eclipse上搭建Spirng+SpringMVC+Maven+Mybatis+MySQL项目"
description: ""
category: 
tags: []
---
{% include JB/setup %}

本文中所使用到的各个软件版本如下：

|Software   |   Version            |
|:---------:|:--------------------:|
|Spring     |3.2.9.RELEASE         | 
|Maven      |3.3.9                 |
|Mybatis    |3.2.1                 |
|MySQL      |5.1.49                |
|Java       |1.8.0                 |
|Eclipse    |Mars.2 Release (4.5.2)|
|Tomcat     |8.0.33                |

下面详细讲述使用eclipse进行Spring+SpringMVC+Maven+Mybatis+MySQL项目搭建过程，包括搭建步骤和配置步骤。

## 创建数据库和表


```
CREATE TABLE `t_user` (
  `USER_ID` int(11) NOT NULL AUTO_INCREMENT,
  `USER_NAME` char(30) NOT NULL,
  `USER_PASSWORD` char(10) NOT NULL,
  PRIMARY KEY (`USER_ID`),
  KEY `IDX_NAME` (`USER_NAME`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8

INSERT INTO t_user (USER_ID, USER_NAME, USER_PASSWORD) VALUES (1, 'test1', '123456');
INSERT INTO t_user (USER_ID, USER_NAME, USER_PASSWORD) VALUES (2, 'test2', '123456');
```


## 创建Maven工程

1. 选择工程

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/1.png)

2. 选择快速框架

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/2.png)

   下一步
   
3. 填写输出包名、group id、artifact id

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/3.png)


4. 创建好的目录结构如下：

![image](https://raw.githubusercontent.com/yuzujin/yuzujin.github.com/master/images/4.png)

5. 添加包依赖，编辑pom.xml文件

```
<project xmlns="http://maven.apache.org/POM/4.0.0" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.test.global.mobile.tools</groupId>
  <artifactId>test</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>war</packaging>
  
  <properties>
      <druid-version>1.0.14</druid-version>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <jdk.version>1.8</jdk.version>
      <!-- spring版本号 -->
      <spring.version>3.2.9.RELEASE</spring.version>
      <!-- log4j日志文件管理包版本 -->
      <slf4j.version>1.6.6</slf4j.version>
      <log4j.version>1.2.17</log4j.version>
      <!-- junit版本号 -->
      <junit.version>4.8.2</junit.version>
      <!-- mybatis版本号 -->
      <mybatis.version>3.2.1</mybatis.version>
   </properties>
      
   <dependencies>
      <!-- spring -->
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-core</artifactId>
          <version>${spring.version}</version>
      </dependency>
      
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-beans</artifactId>
          <version>${spring.version}</version>
      </dependency>
      
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-web</artifactId>
          <version>${spring.version}</version>
      </dependency>
      
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>${spring.version}</version>
      </dependency>
      
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>${spring.version}</version>
      </dependency>
      
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context-support</artifactId>
          <version>${spring.version}</version>
      </dependency>
      
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-aop</artifactId>
          <version>${spring.version}</version>
      </dependency>

      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-aspects</artifactId>
          <version>${spring.version}</version>
      </dependency>
      
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-tx</artifactId>
          <version>${spring.version}</version>
      </dependency>

      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-jdbc</artifactId>
          <version>${spring.version}</version>
      </dependency>
      
      <!-- common -->
      <dependency>
          <groupId>commons-httpclient</groupId>
          <artifactId>commons-httpclient</artifactId>
          <version>3.1</version>
      </dependency>

      <!--mybatis依赖 -->
      <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis</artifactId>
          <version>${mybatis.version}</version>
      </dependency>

      <!-- mybatis/spring包 -->
      <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis-spring</artifactId>
          <version>1.2.0</version>
      </dependency>

      <!-- log4j -->
      <dependency>
          <groupId>log4j</groupId>
          <artifactId>log4j</artifactId>
          <version>${log4j.version}</version>
      </dependency>
      
      <dependency>
          <groupId>commons-lang</groupId>
          <artifactId>commons-lang</artifactId>
          <version>2.6</version>
      </dependency>
      
      <!-- slf4j -->
      <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-api</artifactId>
          <version>${slf4j.version}</version>
      </dependency>
      
      <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-log4j12</artifactId>
          <version>${slf4j.version}</version>
      </dependency>
      
      <!-- gson -->
      <dependency>
          <groupId>com.google.code.gson</groupId>
          <artifactId>gson</artifactId>
          <version>2.2.4</version>
      </dependency>
      
      <!-- json -->
      <dependency>
          <groupId>net.sf.json-lib</groupId>
          <artifactId>json-lib</artifactId>
          <version>2.4</version>
          <classifier>jdk15</classifier>
      </dependency>
      
      <dependency>
          <groupId>org.codehaus.jackson</groupId>
          <artifactId>jackson-mapper-asl</artifactId>
          <version>1.9.10</version>
      </dependency>
      
      <!-- cache -->
      <dependency>
          <groupId>com.test.global.mobile.server</groupId>
          <artifactId>dataaccess-cache</artifactId>
          <version>1.0-SNAPSHOT</version>
      </dependency>
      
      <!-- test -->
      <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>${junit.version}</version>
          <scope>test</scope>
      </dependency>

      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-test</artifactId>
          <version>${spring.version}</version>
          <scope>test</scope>
      </dependency>

      <!-- Database -->
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>5.1.29</version>
      </dependency>

      <!-- Provided -->
      <dependency>
          <groupId>javax</groupId>
          <artifactId>javaee-api</artifactId>
          <version>6.0</version>
          <scope>provided</scope>
      </dependency>
      
      <dependency>
          <groupId>javax</groupId>
          <artifactId>javaee-web-api</artifactId>
          <version>6.0</version>
          <scope>provided</scope>
      </dependency>
   
  </dependencies> 
  <build>
      <finalName>pocketbonus</finalName>
      <plugins>
          <plugin>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>2.3.2</version>
              <configuration>
                  <source>1.8</source>
                  <target>1.8</target>
              </configuration>
 
                   </plugin>
          <plugin>
              <artifactId>maven-war-plugin</artifactId>
              <version>2.4</version>
              <configuration>
                  <version>3.0</version>
              </configuration>
          </plugin>
      </plugins>
  </build>
  
</project>
```

## 配置文件

### mybatis配置

在src/main/resources下创建mybatis目录，在目录下创建mybatis-config.xml文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
</configuration>
```

### 数据源配置

在src/main/resources下创建properties目录，在目录下创建test_jdbc.properties文件：

```
jdbc_driverClassName=com.mysql.jdbc.Driver
jdbc_url=jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=round
jdbc_username=root
jdbc_password=123
```

其中jdbc_url中的useUnicode=true&characterEncoding=utf8指定编码格式，zeroDateTimeBehavior=round会将0000-00-00 00:00:00转换为最近的日期0001-01-01，否则会报错Cannot convert value '0000-00-00 00:00:00' from column 1 to TIMESTAMP。

### 表映射文件配置

在src/main/resources下创建mapper目录，在目录下创建userMapper.xml文件：

```
    <?xml version="1.0" encoding="UTF-8"?>  
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.test.dao.UserDao">
        <!--设置domain类和数据库中表的字段一一对应，注意数据库字段和domain类中的字段名称不致，此处一定要！-->
        <resultMap id="BaseResultMap" type="com.test.domain.User">
            <id column="USER_ID" property="userId" jdbcType="INTEGER" />
            <result column="USER_NAME" property="userName" jdbcType="CHAR" />
            <result column="USER_PASSWORD" property="userPassword" jdbcType="CHAR" />
        </resultMap>
        <!-- 查询单条记录 -->
        <select id="selectUserById" parameterType="int" resultMap="BaseResultMap">
            SELECT * FROM t_user WHERE USER_ID = #{userId}
        </select>
    </mapper>
```

### Spring application文件配置

在src/main/resources下创建applicationContext.xml文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="  
           http://www.springframework.org/schema/beans  
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
           http://www.springframework.org/schema/aop  
           http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
           http://www.springframework.org/schema/context  
           http://www.springframework.org/schema/context/spring-context-3.0.xsd">

     <!-- 引入jdbc配置文件 -->  
     <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
               <value>classpath:properties/*.properties</value>
                <!--要是有多个配置文件，只需在这里继续添加即可 -->
            </list>
        </property>
     </bean>

     <!-- 配置数据源 -->
     <bean id="dataSource"
        class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!-- 不使用properties来配置 -->
        <!-- <property name="driverClassName" value="com.mysql.jdbc.Driver" /> 
             <property name="url" value="jdbc:mysql://localhost:3306/learning" /> 
             <property name="username" value="root" /> 
             <property name="password" value="christmas258@" /> -->
        <!-- 使用properties来配置 -->
        <property name="driverClassName">
            <value>${jdbc_driverClassName}</value>
        </property>
        <property name="url">
            <value>${jdbc_url}</value>
        </property>
        <property name="username">
            <value>${jdbc_username}</value>
        </property>
        <property name="password">
            <value>${jdbc_password}</value>
        </property>
     </bean>

     <!-- 自动扫描了所有的XxxxMapper.xml对应的mapper接口文件，这样就不用一个一个手动配置Mpper的映射了，只要Mapper接口类和Mapper映射文件对应起来就可以了。 -->
     <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage"
            value="com.luo.dao" />
     </bean>

     <!-- 配置Mybatis的文件 ，mapperLocations配置**Mapper.xml文件位置，configLocation配置mybatis-config文件位置-->
     <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>  
        <property name="configLocation" value="classpath:mybatis/mybatis-config.xml" />
        <!-- <property name="typeAliasesPackage" value="com.tiantian.ckeditor.model" /> -->
     </bean>

     <!-- 自动扫描注解的bean -->
     <context:component-scan base-package="com.test.service" />
</beans>
```


