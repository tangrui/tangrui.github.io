---
ID: 740
title: 在 CentOS 6 上部署 Cloudera Manager 集群
author: 唐睿
date: 2017-08-03 10:06:00 +0800
categories: [技术]
tags: [大数据, CDH, Cloudera, Hadoop]
excerpt: >
  本文主要介绍了如何在 CentOS 6 上部署 Cloudera Manager 及相关的大数据服务。
layout: post
published: true
---

# 一、准备工作

## 1.1、系统权限要求

最简单的情况是能够以 root 身份登录集群中的所有服务器，但是出于安全考虑，多数情况下的服务器配置是不允许这么操作的，因此需要系统能支持以无密码的方式使用 `sudo` 执行如下命令：

* yum
* sed
* service
* chkconfig
* id
* rm
* mv
* chown
* install
* hostname
* vim
* setenforce

配置方法是在 `/etc/sudoers` 文件中添加以下内容：

```
tangrui             ALL=(ALL)       NOPASSWD: ALL
```

该配置允许 tangrui 用户以无密码的 `sudo` 命令执行任意程序，如果想要配置白名单，可以将最后的 `ALL` 修改为对应的命令即可。

## 1.2、升级系统

将系统升级到最新版本：

```bash
$ sudo yum -y upgrade
```

## 1.3、安装 wget

```bash
$ sudo yum install -y wget
```

## 1.4、安装并配置 NTP 服务

```bash
$ sudo yum install -y ntp
$ sudo chkconfig ntpd on
$ sudo service ntpd start
$ sudo ntpdate -u pool.ntp.org
```

## 1.5、禁用 SELinux

使用 `getenforce` 命令检查 SELinux 是否启用，如果不是 Disabled 状态，可以使用如下命令关闭：

```bash
$ sudo setenforce 0
```

## 1.6、配置防火墙

配置防火墙，以允许集群间的节点主机可以相互通讯。

## 1.7、配置域名解析

修改每个节点上的 `/etc/hosts` 文件，以确保彼此之间能够通过域名解析到正确的 IP 地址，参考如下：

```
127.0.0.1 localhost.localdomain localhost
192.168.1.1 cluster-01.example.com cluster-01
192.168.1.2 cluster-02.example.com cluster-02
192.168.1.3 cluster-03.example.com cluster-03
```

*注：小规模集群可以采用修改 hosts 文件的方式实现域名解析，对于大规模集群而言，最好使用 DNS。具体 DNS 的配置方式请参考其他文档。*

## 1.8、安装 JDK

安装 JDK 1.7，要确保在系统全局能够访问到 `java` 命令，最好使用 rpm 包进行安装。

*注：如果系统自带安装有 OpenJDK，请将其卸载。*

```bash
$ rpm -aq | grep -i jdk
```

根据该命令输出的结果，使用 `sudo yum remove` 方法逐个删除已安装的 OpenJDK。

## 1.9、参考文档

CDH 5 和 Cloudera Manager 5 需求并支持的版本，请参考文档：[CDH 5 and Cloudera Manager 5 Requirements and Supported Versions](https://www.cloudera.com/documentation/enterprise/release-notes/topics/rn_consolidated_pcm.html)

# 二、系统规划

## 2.1、集群规划

集群规划请参考如下文档：

* [Cluster Hosts and Role Assignments](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_host_allocations.html)
* [Cloudera Manager and Managed Service Datastores](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_installing_configuring_dbs.html)

## 2.2、服务端口

各服务所需要使用的端口，请参考 Cloudera 的官方文档 [Ports](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_ports.html) ，需要据此配置防火墙，以便需要的外部端口能够被集群以外的机器访问。

# 三、安装 Cloudera Manager 基础组件

## 3.1、添加 Cloudera Manager 源

*注：需要在每个节点都添加此源。*

```bash
$ cd /etc/yum.repos.d
$ sudo wget http://archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo
```

## 3.2、安装 Cloudera Manager Server

*注：只需要在一个节点上安装此组件。*

```bash
$ sudo yum install -y cloudera-manager-server
```

## 3.3、安装 Cloudera Manager Daemons 和 Cloudera Manager Agent

*注：需要在每个节点上都安装此组件，此步骤可以省略，但是为了让后面运行 Cloudera Manager 管理界面的时候更快速，可以预先把这些比较大的组件安装好。*

```bash
$ sudo yum install -y cloudera-manager-daemons cloudera-manager-agent
```

# 四、数据库配置

## 4.1、安装 MySQL 服务器

*注：只需要在一个节点上安装此服务，也可以使用已有服务。*

```bash
$ sudo yum install -y mysql-server
$ sudo chkconfig mysqld on
$ sudo service mysqld start
# 在运行以下命令时，不要禁止 root 用户远程登录 MySQL 服务器
$ sudo mysql_secure_installation
```

倘若不小心在运行 `mysql_secure_installation` 命令的时候禁止了 root 用户远程登录 MySQL 服务器，以下命令可以重新开启：

```
$ mysql -u root -p
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '<root_password>' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```

## 4.2、准备 Cloudera Manager Server 数据库

*注：有关各数据库的容量规划，请参考文档：[Cloudera Manager and Managed Service Datastores](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_installing_configuring_dbs.html#cmig_topic_5_1)。*

在安装有 Cloudera Manager Server 的节点上运行如下命令：

```bash
$ sudo /usr/share/cmf/schema/scm_prepare_database.sh mysql -h <mysql_host> -u root -p --scm-host <scm_host> scm scm scm
```

需要注意以下事项：

1. 运行该命令，需要确保能够以 root 身份远程登录 MySQL 服务器
2. 该命令会修改 `/etc/cloudera-scm-server/db.properties` 文件中的相关配置
3. `mysql_host` 和 `scm_host` 最好使用域名
4. 要确保 `/usr/share/cmf/lib` 目录下包含 MySQL 数据库的驱动
5. 也可以手动创建数据库，并修改 `db.properties` 文件

## 4.3、准备 Hive 数据库

```
$ mysql -u root -p
mysql> CREATE DATABASE hive DEFAULT CHARACTER SET UTF8;
mysql> GRANT ALL PRIVILEGES ON hive.* TO 'hive'@'%' IDENTIFIED BY 'hive';
mysql> FLUSH PRIVILEGES;
```

## 4.4、准备 Oozie 数据库

```
$ mysql -u root -p
mysql> CREATE DATABASE oozie DEFAULT CHARACTER SET UTF8;
mysql> GRANT ALL PRIVILEGES ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';
mysql> FLUSH PRIVILEGES;
```

## 4.5、准备 Hue 数据库

```
$ mysql -u root -p
mysql> CREATE DATABASE hue DEFAULT CHARACTER SET UTF8;
mysql> GRANT ALL PRIVILEGES ON hue.* TO 'hue'@'%' IDENTIFIED BY 'hue';
mysql> FLUSH PRIVILEGES;
```

# 5、启动 Cloudera Manager 服务

```bash
$ sudo service cloudera-scm-server start
```

# 6、Cloudera Manager 安装过程

(1) 使用默认用户名和密码 `admin/admin` 登录
![使用默认用户名和密码 admin/admin 登录](/static/uploads/2017/cdh/1.png)

(2) 接受 License
![接受 License](/static/uploads/2017/cdh/2.png)

(3) 选择安装类别
![选择安装类别](/static/uploads/2017/cdh/3.png)

(4) 即将安装的组件列表
![即将安装的组件列表](/static/uploads/2017/cdh/4.png)

(5) 在输入框内，填写需要管理的主机地址
![在输入框内，填写需要管理的主机地址](/static/uploads/2017/cdh/5.png)

(6) 确认主机
![确认主机](/static/uploads/2017/cdh/6.png)

(7) 使用 Parcels 进行安装
![使用 Parcels 进行安装](/static/uploads/2017/cdh/7.png)
![使用 Parcels 进行安装](/static/uploads/2017/cdh/8.png)

(8) 配置 Parcels 的安装目录，如果提示目录不存在，需手动创建，并确保目录的 Owner 为 `cloudera-scm:cloudera-scm` 用户
![配置 Parcels 的安装目录](/static/uploads/2017/cdh/9.png)

(9) 选择是否需要安装 Oracle JDK。因为我们已经手动安装过了，因此这一步可以跳过，不用勾选
![选择是否需要安装 Oracle JDK](/static/uploads/2017/cdh/10.png)

(10) 是否使用单用户模式安装，不用勾选
![是否使用单用户模式安装，不用勾选](/static/uploads/2017/cdh/11.png)

(11) 指定登录主机的用户名和密码，也可以使用密钥登录
![指定登录主机的用户名和密码，也可以使用密钥登录](/static/uploads/2017/cdh/12.png)

(12) 准备服务器。因为我们此前已经手动安装好了 cloudera-manager-daemons 和 cloudera-manager-agent 这一步会比较快。如果前面没有手动安装，那么这里会自动安装，就要等待比较久的时间
![准备服务器](/static/uploads/2017/cdh/13.png)

(13) 服务器准备完成
![服务器准备完成](/static/uploads/2017/cdh/14.png)

(14) 下载、分发、解压和激活 Parcels
![下载、分发、解压和激活 Parcels](/static/uploads/2017/cdh/15.png)

(15) 完成
![完成](/static/uploads/2017/cdh/16.png)

(16) 可以不用继续下面的操作，直接返回就能够看到管理界面了，其他的服务可以后续手动安装
![接返回 Cloudera Manager](/static/uploads/2017/cdh/17.png)

# 7、安装其他组件

## 7.1、安装 Zookeeper

(1) 选择 Add Service 菜单
![选择 Add Service 菜单](/static/uploads/2017/cdh/18.png)

(2) 选择需要安装的服务
![选择需要安装的服务](/static/uploads/2017/cdh/19.png)

(3) 选择需要部署此服务的主机
![选择需要部署此服务的主机](/static/uploads/2017/cdh/20.png)
![对一些核心参数进行配置。](/static/uploads/2017/cdh/21.png)

(4) 对一些核心参数进行配置。因为所用的虚拟机默认 `/var` 目录容量比较小，因此需要做一些调整。如果参数设置不当，Cloudera Manager 会有告警，可以在安装完成以后根据提示进行修改
![等待服务启动](/static/uploads/2017/cdh/22.png)

(5) 等待服务启动
![服务启动完成](/static/uploads/2017/cdh/23.png)

(6) 服务启动完成
![服务启动完成](/static/uploads/2017/cdh/24.png)

(7) 安装完成
![安装完成](/static/uploads/2017/cdh/25.png)

## 7.2、安装 Cloudera Management Service 服务

(1) 点击右上角的 Add Cloudera Management Service 按钮
![点击右上角的 Add Cloudera Management Service 按钮](/static/uploads/2017/cdh/26.png)

(2) 配置参数
![配置参数](/static/uploads/2017/cdh/27.png)

(3) 安装完成，启动服务
![安装完成，启动服务](/static/uploads/2017/cdh/28.png)

(4) 等待服务启动完成
![等待服务启动完成](/static/uploads/2017/cdh/29.png)

## 7.3、安装其它服务

其它服务的安装方式大同小异，在这里就不再截图叙述，按照提示一步步完成即可。如果参数配置有误，系统会有告警，根据告警提示进行相应的修改即可。实在不行也可以直接删除，从头再来。

# 8、常见问题

## 8.1、安装 Hive 时找不到 JDBC Driver

请参考：[Cloudera Manager 添加 Hive 时报错找不到 JDBC Driver](http://fatkun.com/2015/05/cloudera-manage-jdbc-driver-cannot-be-found.html)。

将 MySQL Connector Java 复制到 `/usr/share/java` 目录下，但是文件名要命名为 `mysql-connector-java.jar`。

## 8.2、设置 swappiness

请参考：[Tweak Swap on CentOS 7](https://www.hostingstuff.net/tweak-swap-centos-7/)。

```bash
su -
sysctl vm.swappiness=10
echo "vm.swappiness = 10" >> /etc/sysctl.conf
```

## 8.3、修改内核参数

```bash
su -
echo never > /sys/kernel/mm/transparent_hugepage/defrag 
echo never > /sys/kernel/mm/transparent_hugepage/enabled 

# 并将如下两个语句写入 /etc/rc.local 文件中
echo "kudu soft nofile 5242880" >> /etc/security/limits.conf
echo "kudu hard nofile 5242880" >> /etc/security/limits.conf
```
