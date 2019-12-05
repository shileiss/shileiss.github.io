---
title: "Dubbo 使用指南"
date: 2019-12-05 10:00:00 +0800
category: Dubbo
tags: [Dubbo, 使用指南]
excerpt: 本文主要主要说明Dubbo的概念和基本使用方法。
---

# Dubbo 使用指南

> 随着互联网的发展，之前的架构已经无法满足正常的需要，所以出现了SOA的架构。为满足这一架构，Alibaba公司出品了Dubbo的框架，该框架在国内的电商平台上使用的很广泛。这样一个优秀的架构也成为了Apache的孵化项目，可以预见Dubbo的框架以后会越来越成功。

## Dubbo 划分

Dubbo框架中最为核心的便是服务提供者，服务消费者的概念。

服务的提供者在注册中心注册所暴露的服务。

服务的消费者在注册中心订阅所需要的服务。

注册中心将通知消费者服务提供者的地址列表。

服务的消费者根据该地址列表基于负载均衡的算法调用服务提供者其中的一个，如果失败，调用其他。

服务的消费者与服务的提供者定时向统计数据到监控中心。

## 如何创建一个服务的提供者

因为我们的应用采用了CRM的五层架构，所以我们将服务从服务分类、数据对象、业务对象等多角度、多维度进行了查分。这每一个原子化的服务都是服务的提供者。那么我们如何编写一个服务的提供者，并让他注册到注册中心供服务的消费者调用呢？

### 建立一个SpringBoot的模块

我们的系统是一个多模块的系统，这在电商平台中非常的常见。我们将这些SpringBoot项目生成一个Docker镜像，然后通过对镜像的实例化，从而实现服务的增减。

我们可以使用[Dubbo Spring Initializr](http://start.dubbo.io/)创建一基础的项目。这个项目包括了非常常用的几个依赖，并创建一个基于Dubbo的项目，我们只需要通过一些简单的配置，就能使它可用。

![Screenshot_2019-03-11 Dubbo Spring Initializr](./Screenshot_2019-03-11 Dubbo Spring Initializr.png)

在以上的项目中我们创建了一个名字叫mall-server-cms的项目（我们在这个项目中将它作为一个模块）

#### 依赖与插件

首先，我们添加以下依赖：

```xml
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
	<java.version>1.8</java.version>
	<skipTests>true</skipTests>
	<dubbo.version>2.7.0</dubbo.version>
</properties>

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo-dependencies-bom</artifactId>
			<version>${dubbo.version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

<dependencies>
	<!-- 项目内依赖 start -->
	<dependency>
		<groupId>dev.litong.mall</groupId>
		<artifactId>mall-api</artifactId>
		<version>1.0-SNAPSHOT</version>
		<scope>compile</scope>
	</dependency>
	<!-- 项目内依赖 end -->
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>fastjson</artifactId>
		<version>1.2.47</version>
	</dependency>
	<!-- Dubbo 相关 start -->
	<dependency>
		<groupId>com.alibaba.boot</groupId>
		<artifactId>dubbo-spring-boot-starter</artifactId>
		<version>0.1.0</version>
	</dependency>
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
        <version>${dubbo.version}</version>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
    </dependency>
	<dependency>
		<groupId>com.101tec</groupId>
		<artifactId>zkclient</artifactId>
	</dependency>
	<!-- Dubbo 相关 end -->
	<dependency>
		<groupId>org.apache.commons</groupId>
		<artifactId>commons-lang3</artifactId>
		<version>3.7</version>
	</dependency>

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>
```

再在build节点添加以下的插件，该插件为可将SpringBoot生成Docker镜像。

```xml
<plugin>
	<groupId>com.spotify</groupId>
	<artifactId>docker-maven-plugin</artifactId>
	<version>1.1.0</version>
	<executions>
		<execution>
			<id>build-image</id>
			<phase>package</phase>
			<goals>
				<goal>build</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<imageName>mall/${project.artifactId}:${project.version}</imageName>
		<baseImage>java:8</baseImage>
		<entryPoint>["java", "-jar", "-Dspring.profiles.active=prod","/${project.build.finalName}.jar"]
		</entryPoint>
		<resources>
			<resource>
				<targetPath>/</targetPath>
				<directory>${project.build.directory}</directory>
				<include>${project.build.finalName}.jar</include>
			</resource>
		</resources>
	</configuration>
</plugin>
```

#### 接口

我们选择在根项目中添加一个模块，命名为mall-api，这个模块包含了项目中的所有服务接口，可供服务的提供者和服务的消费者使用。下面是一个接口的例子：

```java
package dev.litong.mall.service;

import com.macro.mall.model.CmsPrefrenceArea;

import java.util.List;

/**
 * 優選專區Service
 *
 * @author litong
 */
public interface CmsPreferenceAreaService {
    /**
     * 列出所有優選專區
     *
     * @return 所有優選專區
     */
    List<CmsPrefrenceArea> listAll();
}

```

#### 服务实现

接下来，我们在新建的项目中改写接口的实现类：

```java
package dev.litong.mall.service.impl;

import com.macro.mall.model.CmsPrefrenceArea;
import dev.litong.mall.service.CmsPreferenceAreaService;
import org.apache.dubbo.config.annotation.Service;
import org.springframework.stereotype.Component;

import java.util.List;

@Service(version = "1.0.0")
@Component
public class CmsPreferenceAreaServiceImpl implements CmsPreferenceAreaService {

    @Override
    public List<CmsPrefrenceArea> listAll() {
        return null;
    }
}
```

> 注：该Service是apache dubbo的Service并非Spring的。

#### 配置SpringBoot启动

首先，需要在SpringBoot的启动APP上添加注解`@EnableDubbo`：

```java
@SpringBootApplication
@EnableDubbo
public class MallServerCmsApplication {

    public static void main(String[] args) {

        SpringApplication.run(MallServerCmsApplication.class, args);
    }
}
```

再者，我们改写resurces文件中的application.properties文件：

```properties
dubbo.application.name = mall-server-cms

# Base packages to scan Dubbo Component: @com.alibaba.dubbo.config.annotation.Service
dubbo.scan.basePackages  = dev.litong.mall.service.impl

## RegistryConfig Bean
dubbo.registry.address = zookeeper://localhost:2181
dubbo.config-center.address=zookeeper://127.0.0.1:2181

dubbo.protocol.name=dubbo
dubbo.protocol.port=20880

dubbo.application.qosEnable=true
```