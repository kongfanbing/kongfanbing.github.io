---
layout: post
title:  gitlab-runner使用记录
date:   2020-03-03
tags: [GitLab,CI/CD,DevOps]
---

>   不积跬步，无以至千里；不积小流，无以成江海。
>
>   ——荀子

# 前言

很多年以前，就做过CD（持续部署）的脚本，那时SVN还是主流，采用SVN+rsync的方式从线下往线上推送代码，但是效果不是很理想。这两年一直在想着DevOps，不过限于各个项目的背景不同，最近才算做了一个开头，特意做个记录。`本次实践是使用自建GitLab（12.2.5-ee）系统+GitLab-Runner完成的Laravel项目自动部署。`

# GitLab-Runner

## GitLab-Runner是什么？

GitLab-Runner是配合GitLab-CI进行使用的。在GitLab里，每次代码的提交变动都会汇聚到GitLab-CI，再由GitLab-CI分配给关联的Runner，Runner可以理解为是要执行的脚本集合，但是每个Runner需要注册到GitLab-CI。

## GitLab-Runner安装

由于访问国外资源太慢，这里使用了[清华的镜像网站](https://mirrors.tuna.tsinghua.edu.cn/help/gitlab-runner/)，新建 `/etc/yum.repos.d/gitlab-runner.repo`

```shell
[gitlab-runner]
name=gitlab-runner
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-runner/yum/el6 #centos6
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```

再执行

```shell
sudo yum makecache
sudo yum install gitlab-runner
```

执行成功后，会自动创建gitlab-runner用户，如果采用编译安装的话，需要自行创建gitlab-runner用户（主要用于运行gitlab-runner，后面会讲到）。

## GitLab-Runner注册

运行如下命令：

```bash
sudo gitlab-runner register
```

输入GitLab的地址：

```bash
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.xxx.com/
```

输入你的注册令牌，需要进入GitLab对应项目的设置->CI/CD页面，点开Runner，查看specific Runner中的注册令牌信息：

```bash
Please enter the gitlab-ci token for this runner  
xxx
```

输入Runner的描述信息，我这里采用部署后的域名（测试环境和正式环境不同，可以作为区分）：

```bash
Please enter the gitlab-ci description for this runner  
[hostame] https://dev.xxx.com
```

输入Runner的tag，稍后可以在GitLab上修改它：

```bash
Please enter the gitlab-ci tags for this runner (comma separated):
dev
```

输入Runner的executor，我这里是使用的shell就结束了（如果使用docker还会有下一步配置）：

```bash
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
shell
```

注册完毕。

## GitLab-Runner运行

直接运行Runner即可：

```bash
sudo gitlab-runner run
```

# .gitlab-ci.yml

## 介绍

.gitlab-ci.yml这是管理本项目的gitlab-ci配置文件，采用yaml语法，**需要放在项目的根目录下，并提交到gitlab**。

## 示例

```yaml
stages: #定义构建阶段，可以被job调用的阶段，最多有三个build，test，deploy顺序执行。
  - deploy #这里只列出了deploy，也就是说下面的job只能在deploy阶段才能触发。
job_dev: #测试环境部署的job
    stage: deploy #在deploy后运行（在本例中，设为test就不会被触发,而且还会报语法错误）
    when: always #无论前面stages中jobs执行状态如何都执行这个job。
    script: #运行的命令
      - deploy #这里是在服务器创建了一个deploy的命令，里面详细写了需要执行的脚本，后面会贴出来。
    only: #确定哪些分支可以触发该job
      - master #只有master分支的提交才触发这个job。
    tags: #指定包含哪些tag的Runner才会执行
      - dev #tag包含dev的Runner，如果多个的话，Runner要同时包含指定的tags
job_prod: #正式环境部署的job
    stage: deploy
    when: always
    script:
      - deploy
    only:
      - deploy #只有deploy分支可触发这个job。
    tags:
      - prod #这里的tag跟job_dev的不同，可以指定Runner运行，以此来区分测试环境和正式环境的自动部署。

```

这里只使用了部分配置，如果想了解全部配置信息，可参考[通过 .gitlab-ci.yml配置任务](https://fennay.github.io/gitlab-ci-cn/gitlab-ci-yaml.html)。

# 服务器部署

## 准备工作

在.gitlab-ci.yml示例中，两个job中script都调用了deploy命令，这个命令是我自己在服务器创建的，因为项目部署存在一定的复杂度和对.gitlab-ci.yml配置不熟的原因，所以就写了个shell脚本放到服务器上执行。

为了便于管理，创建了一个文件/home/gitlab-runner/.local/deploy（这里就用到了gitlab-runner用户），加上执行权限：

``` shell
sudo chmod +x deploy
```

新增环境变量:

``` bash
sudo vim /etc/profile
#新增一行
export PATH=$PATH:/home/gitlab-runner/.local
#保存后退出，使用source立即生效
sudo source /etc/profile
```

**gitlab-runner进程是由gitlab-runner用户执行的，所以在deploy中所执行的命令都需要有足够的权限，包括项目部署的目录等。**

## deploy脚本

``` bash
#!/bin/bash
#进入项目部署目录
cd /data/www/xxx
#拉取代码
git pull https://gitlab.xxx.com/xxx #这里git拉取的配置已经做好，不再赘述。
#安装依赖
compsoer install
#执行数据迁移【Laravel项目】
php artisan migrate
#部署完成。
```

# 结语

这次实践只是简单的代码拉取和持续部署，对于DevOps的目标来说，还有很多需要学习实践的，比如docker和Kubernetes。一步一步来吧，如果项目有需要的话，还是要试一试的。