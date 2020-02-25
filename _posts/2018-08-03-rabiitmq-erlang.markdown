---
layout: post
title:  rabbitmq安装备忘
date:   2018-08-03
tags: [消息队列，rabbitmq，erlang]
---

>   不积跬步，无以至千里；不积小流，无以成江海。
>
>   ——荀子

# 安装erlang

系统版本是CentOS Linux release 7.4.1708 (Core)，根据erlang官方地址https://www.erlang-solutions.com/resources/download.html安装erlang，会报错。

```shell
[root@bogon rabbitmq]# wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
[root@bogon rabbitmq]# rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
error: Failed dependencies:
        epel-release is needed by erlang-solutions-1.0-1.noarch
```

最后从网上找到一篇文章http://blog.51cto.com/yanconggod/1933009，文章最后提供了解决方案：

```shell
#添加仓库
[root@bogon rabbitmq]# vim /etc/yum.repos.d/rabbitmq-erlang.repo
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq/rpm/erlang/19/el/7
gpgcheck=1
gpgkey=https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1

#然后执行yum安装erlang
[root@bogon rabbitmq]# yum install erlang -y
```

PS：这篇文章最后还有安装rabbitmq的部分，不过执行他的方法执行会报错，所以安装rabbitmq还是参考的官网。

``` shell
[root@bogon rabbitmq]# yum install rabbitmq-server -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.huaweicloud.com
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.huaweicloud.com
No package rabbitmq-server available.
Error: Nothing to do
```

# 安装RabbitMQ

参考官网的安装说明：https://www.rabbitmq.com/install-rpm.html。由于安装erlang的时候遇到了问题，通过上面的步骤解决后，直接进入安装RabbitMQ的步骤即可。

首先，根据系统下载对应的rpm包，我这里对应下载的是CentOS7.X的https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.7/rabbitmq-server-3.7.7-1.el7.noarch.rpm，下载成功后执行安装：

```shell
[root@bogon rabbitmq]# rpm --import https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
# this example assumes the CentOS 7 version of the package
[root@bogon rabbitmq]# yum install rabbitmq-server-3.7.7-1.el7.noarch.rpm
```

没有报错的话，就安装成功了。

# 启动RabbitMQ

安装后，将RabbitMQ加入开机自启动，并启动RabbitMQ服务。

```shell
[root@bogon rabbitmq]# chkconfig rabbitmq-server on
Note: Forwarding request to 'systemctl enable rabbitmq-server.service'.
Created symlink from /etc/systemd/system/multi-user.target.wants/rabbitmq-server.service to /usr/lib/systemd/system/rabbitmq-server.service.
[root@bogon rabbitmq]# systemctl enable rabbitmq-server.service
[root@bogon rabbitmq]# service rabbitmq-server start
```

# 其他

## 启动失败

启动`service rabbitmq-server start` 时间比较长，而且会报错`ERROR: epmd error for host bogon: timeout (timed out)` ，解决方案是修改`/etc/hosts`文件，增加一条：

```shell
127.0.0.1	bogon #这里是主机名hostname，请修改成自己服务器的
```

使用`rabbitmqctl status`查看状态，没有报错的话表示启动正常。

## 启动web管理页面

管理插件是自带安装的，只需要启动即可：

```shell
[root@bogon rabbitmq]# rabbitmq-plugins enable rabbitmq_management
```

根据官网说明，访问地址http://服务器地址:15672，使用默认用户guest（密码：guest）登录，localhost本机登录是正常的，远程登录还需要做一些设置，具体方法是：找到rabbit.app文件并修改loopback_users的设置。

```shell
[root@bogon rabbitmq]# vi /usr/lib/rabbitmq/lib/rabbitmq_server-3.7.7/ebin/rabbit.app
#修改前：
{loopback_users, [<<"guest">>]}
#修改后：
{loopback_users, []}
```

重启rabbitmq-server后，就可以使用guest账户登录。