title: Linux环境下Maven服务器搭建
date: 2016-01-25 16:48:25
tags: maven
---
#### Maven基本准备
### Maven服务器下载
官网地址：[http://download.sonatype.com/nexus/oss/nexus-2.11.1-01-bundle.zip](http://download.sonatype.com/nexus/oss/nexus-2.11.1-01-bundle.zip)，通过此地址下载maven.
``` Linux
$ unzip nexus-2.11.1-01-bundle.zip
```
然后解压文件会得到nexus-2.6.0-05和sonatype-work。
<!--more-->
### Maven安装和启动
进入Maven解压nexus目录，然后执行：
``` Linux
$ nexus-2.11.1-01/bin/nexus start
```
在安装时可能会遇到错误，比如环境变量未配置时请配置一下：
``` text
RUN_AS_USER=root   
NEXUS_HOME=$HOME/nexus-2.0.3/
```
启动成功之后，通过[http://ip:8081/nexus/index.html](http://ip:8081/nexus/index.html)访问maven仓库，默认管理员账户和密码：admin/admin123

### 修改Maven工作目录sonatype-work位置
默认Maven安装之后会生成sonatype-work目录，此目录将作为默认的jar文件放置处。为了防止后期不停的累加导致Maven仓库变大同时也不利于后期对Maven的管理。
--待定--

### 修改Maven仓库访问的端口
首先查看Maven是否正在运行
``` Linux
$ ps -ef |grep nexus
```
然后通过kill杀掉进程，然后修改maven配置文件
```
$ cd $HOME/nexus-2.11.1-01/conf
$ vi nexus.properties
```
把application-port=8081 修改为application-port=8981<br>
然后访问：http://IP:8981/nexus/index.html#welcome
### 仓库添加

* [个人发布库流程](http://www.open-open.com/lib/view/open1435109824278.html)
