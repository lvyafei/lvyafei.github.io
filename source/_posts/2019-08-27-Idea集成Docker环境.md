---
layout: lay_post
title: "Idea集成Docker环境"
date: 2019-08-27
categories: Docker指令
tags: 备忘录
author: lvyafei
---

## 0.概述

Idea中集成Docker环境，自动部署

<!-- more -->

## 1.liunx下docker开启TCP管理端口

1.创建目录/etc/systemd/system/docker.service.d

mkdir /etc/systemd/system/docker.service.d

2.在这个目录下创建tcp.conf文件,增加以下内容 

vim  tcp.conf

```conf
[Service] 
ExecStart= 
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 
EOF
```

3.Daemon重新reload ，并重启docker

```shell
systemctl daemon-reload
systemctl restart docker
```
4.查看端口是否打开

lsof -i:2375

## 2.idea插件配置

安装插件：Docker integration

IDEA Docker插件配置 

File–>Settings–>Build,Execution,Deployment–>Docker–>进行如下配置：

添加tcp socket:<ip>:2375

添加Docker的菜单窗口 

IDEA顶部工具栏的View–>Tool Windows–>Docker 完成点击左下角的小窗口图标放大即可看到Docker的菜单工具栏

## 3.windows环境变量配置

需增加系统变量,host地址为liunx机器的地址

```
DOCKER_HOST=tcp://<host>:2375
```

## 4.项目配置

项目工程中须检查如下配置：

```
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.2.0</version>
    <executions>
	    <execution>
	        <phase>deploy</phase>
	        <goals>
	            <goal>build</goal>
	        </goals>
	    </execution>
	    <!--依次生成远程docker私服的镜像tag：[project.version、latest]-->
	    <execution>
	        <id>tag-image</id>
	        <phase>deploy</phase>
	        <goals>
	            <goal>tag</goal>
	        </goals>
	        <configuration>
	            <forceTags>true</forceTags>
	            <image>${project.imagename}:${project.version}</image>
	            <newName>
	                ${docker.repository}/${docker.registry.name}/${project.imagename}:${project.version}
	            </newName>
	        </configuration>
	    </execution>
	    <execution>
	        <id>tag-image-latest</id>
	        <phase>deploy</phase>
	        <goals>
	            <goal>tag</goal>
	        </goals>
	        <configuration>
	            <forceTags>true</forceTags>
	            <image>${project.imagename}:latest</image>
	            <newName>${docker.repository}/${docker.registry.name}/${project.imagename}:latest
	            </newName>
	        </configuration>
	    </execution>
	    <!--推送[project.version、latest]到远程私服镜像仓库-->
	    <execution>
	        <id>push-image</id>
	        <phase>deploy</phase>
	        <goals>
	            <goal>push</goal>
	        </goals>
	        <configuration>
	            <imageName>
	                ${docker.repository}/${docker.registry.name}/${project.imagename}:${project.version}
	            </imageName>
	            <imageName>${docker.repository}/${docker.registry.name}/${project.imagename}:latest
	            </imageName>
	        </configuration>
	    </execution>
	</executions>
	<configuration>
	    <!--Dockerfile文件所在目录-->
	    <dockerDirectory>${project.basedir}/target/classes/docker</dockerDirectory>
	    <imageName>${project.imagename}</imageName>
	    <imageTags>
	        <!--docker的tag为项目版本号、latest-->
	        <imageTag>${project.version}</imageTag>
	        <!--实际测试过程中发现，latest tag会默认生成-->
	        <!--<imageTag>latest</imageTag>-->
	    </imageTags>
	    <!-- optionally overwrite tags every time image is built with docker:build -->
	    <forceTags>true</forceTags>
	    <!-- 私有仓库配置，需要settings.xml文件配合serverId对应的服务地址 -->
	    <serverId>zts-docker-hub</serverId>
	    <registryUrl>${docker.repository}</registryUrl>
	    <resources>
	        <!-- 将打包文件放入dockerDirectory指定的位置 -->
	        <resource>
	            <targetPath>/</targetPath>
	            <directory>${project.build.directory}</directory>
	            <include>*.zip</include>
	        </resource>
	    </resources>
	</configuration>
</plugin>
```

其中需要注意的是serverId的配置，该配置为私有仓库的登录信息，需要settings.xml文件配合serverId对应的服务地址：

```
<serverId>zts-docker-hub</serverId>
```

在maven的setting.xml中配置如下：

```
<server>
  <id>zts-docker-hub</id>
  <username>xxx</username>
  <password>xxx</password>
</server>
```

## 5.编译项目

```
mvn clean deploy -T 1C -DskipTests=true -Dmaven.compile.fork=true
```