---
layout: post
title: 使用 TeamCity 实现持续集成（CI）
date: 2016-12-25 00:22
author: admin
comments: true
categories: [CI,TeamCity]
tags: [CI,TeamCity]
---

持续集成（Continuous Integration），也就是我们经常说的 CI，是现代软件开发技术的基础。本文论述了如何使用 TeamCity 持续集成工具来实现项目的持续集成。
   
<!-- more -->

## 为我们什么需要 CI 

目前，CI 已在当前业界被许多软件开发团队所采用，是一种在整个软件开发生命周期内保证代码质量的常见做法。它是一种开发实践，旨在帮助开发团队应对软件开发过程中的如下挑战：

* 自动化检查 ：当软件开发团队在周期性的新增或修改后的代码后，CI 服务器能持续地获取新增或修改后签入的源代码，并可以对这些变更的代码进行质量或者编码规范的检查。常用的工具有 PMD、SonarQube、CheckStyle、FindBugs等。
* 自动化构建 ：CI 系统会依照预先制定的配置计划，或某一特定事件，自动检出代码，并对目标软件进行构建。这里的计划，可以是周期性的时间点，比如10分钟或者1小时构建一次，也可以根据特定事件来触发构建，比如用户主动发出构建命令，或者根据代码的变更来触发构建。构建工具可以选择 Maven 或者 Ant。
* 自动化测试 ：构建检查完成后，可以执行预先制定的一套测试规则，也可以在执行构建的过程中进行测试用例的测试，前提是项目团队采用了测试驱动开发（Test-Driven Development，TDD）。常用的测试工具，有 JUnit、JWebUnit、Selenium 等。
* 自动化部署 :当自动化检查和测试成功完成，将打包软件、构件部署到一个运行环境或者软件仓库。这样，构件才能更迅速地提供给用户使用。
* 及时提醒：当上面定义的任何一个阶段进行过程中发现出错或者预设事件得到触发，都能够及时通知给相应的项目干系人来进行处理。比如，在构建过程发生了错误，CI 服务器可以邮件通知开发人员来进行修复；自动化部署完成了，CI 服务器通知会测试人员来进行测试，等待。除了邮件，提醒的方式可以是短信、桌面通知器，也可以是音响大喇叭。

简言之，CI 是在敏捷开发过程中，实现速度、效率、质量的软件开发实践，可以持续为用户提供交付可用的软件产品。更多详情可以参考[《为什么我们迫切需要持续集成（Continuous Integration）》](https://waylau.com/why-we-need-continuous-integration/)。 

## TeamCity 简介

正如 TeamCity 官网的自我介绍的那样，TeamCity 是一款强大的开箱机用 CI 工具（“Powerful Continuous Integration out of the box”）。其特性包括：

* Technology Awareness
* Key Integrations
* Cloud Integrations
* Continuous Integration
* Configuration
* Build History
* Build Infrastructure
* Code Quality Tracking
* VCS Interoperability
* Extensibility and Customization
* System Maintenance
* User Management
* Pre-Tested Commit

## 使用 TeamCity 实现 CI

下面介绍下 TeamCity 的常见用法。

### 创建项目，关联源码

在创建一个项目（Project）后，可以将项目与相应的源码进行关联。源码管理工具支持 Git、CVS、Subversion 等。本例使用的项目是 necc_country，使用的源码管理工具为 Subversion。

在 VCS Roots 下，添加一个源码关联的地址：  `svn://10.30.22.18:32881/unengli/biz/gov/necc/branches/country`。

![](/images/post/20170107-ci-teamcity-001.jpg)

### 创建构建配置

构建配置（Build Configurations），是指项目构建过程中，一些列的计划动作。比如，可以是代码质量检查、Maven 构建、发布等等动作。

我们选择点击“Edit”按钮，在“Build Steps”中来设置一些构建计划。

![](/images/post/20170107-ci-teamcity-002.jpg)

![](/images/post/20170107-ci-teamcity-003.jpg)

#### 1. 代码质量检查

使用 SonarQube 来进行代码质量检查。

![](/images/post/20170107-ci-teamcity-004.jpg)
 

#### 2. Maven 构建项目

使用 Maven 来项目的构建。可以自定义  Maven 的  Goals，比如：

```
clean install -Dmaven.test.skip=true
```

或者

```
clean package
```

等。如果是采用 TDD 的开发方式，建议不要使用`-Dmaven.test.skip=true`来过滤掉测试步骤。


![](/images/post/20170107-ci-teamcity-005.jpg)


#### 3. 部署项目

可以使用 FTP Upload 或者 SSH Upload 等方式将发布包发布到部署环境中。在本例，由于 CI 和部署的环境是在同一台主机上，使用  FTP Upload 即可。

![](/images/post/20170107-ci-teamcity-006.jpg)

其中，

* Deployment Credentials：部署主机的用户名和密码。
* Target host：是目标部署环境的位置，这里的位置是指 用户的相对路径位置，比如设置位置为`10.30.22.18:/necc_simulation/gov-tomcat-necc/webapps/gov`，使用的用户为`dev`，那么，最终部署到主机的绝对路径为`/home/dev/necc_simulation/gov-tomcat-necc/webapps/gov` 。
* Paths to sources：待部署发布包的位置，这里 `%teamcity.build.workingDir%/web/gov/target/gov`中的 `%teamcity.build.workingDir%`是 TeamCity 构建的工作区间。

#### 4. 重启 Web 容器

在本例中，我们将项目部署到了 Tomcat 容器中，部署完之后，需要重启 Tomcat。这里，我们使用 SSH Exec 来执行一段重启服 Tomcat 的脚本。


![](/images/post/20170107-ci-teamcity-007.jpg)


其中，

* Deployment Target:Tomcat 容器所在的主机。
* Deployment Credentials：部署主机的用户名和密码。
* SSH Commands：在主机上执行的脚本。

```
# 切换到 Tomcat 安装目录的 bin 目录下
cd /home/dev/necc_simulation/gov-tomcat-necc/bin
# 是打印当前的工作目录
pwd
# 杀掉使用特定端口的 Tomcat 进程，即关闭当前程序
/sbin/fuser -k -n tcp 6060
# 给脚本赋予可以执行的权限
chmod 775 startup.sh
# 删除旧的日志
rm -rf ../logs/*
# 查看 Java 版本
java -version
# 启动
./startup.sh
```


## 参考资料

* https://www.jetbrains.com/teamcity/
* https://waylau.com/why-we-need-continuous-integration/