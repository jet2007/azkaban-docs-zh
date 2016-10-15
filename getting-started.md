# 入门

在3.0版本我们提供了三种模式：单个独立应用 “solo-server”模式，相对重量级点的双服务器模式（ two server mode）和分布式的多执行器模式（ multiple-executor mode ）。下面是不同模式的区别：

在 **solo server mode** 模式下，DB使用嵌入式H2数据库，并且 web 服务与 executor 服务运行在同一个进程中。当你只是想尝试体验下功能时，这种模式很有用。它也可以用在规模比较小、不需要什么扩展的场景上。

1. [下载并安装 Solo Server 安装包](#solo-setup)
2. [安装 Azkaban 插件](#plugin-setup)

**two server mode** 用于要求更高的生产环境。数据库存储可以基于主从模式的 MySQL 实例。web 服务和 executor 服务最好是运行在不同的主机上，这样升级和服务维护不会影响到用户。

1. [安装数据库](http://azkaban.github.io/azkaban/docs/latest/#plugin-setup)
2. [下载并安装 Web Server](http://azkaban.github.io/azkaban/docs/latest/#plugin-setup)
3. [下载并安装 Executor Server](http://azkaban.github.io/azkaban/docs/latest/#plugin-setup)
4. [安装 Azkaban 插件](http://azkaban.github.io/azkaban/docs/latest/#plugin-setup)

**multiple executor mode** 用于要求最高的生产环境。数据库存储要使用主从模式的 MySQL 实例。web 服务和 executor 服务要运行在不同的主机上，这样升级和服务维护不会影响到用户。这种多主机的搭建方式让 Azkaban 更加健壮和可扩展。



## 从源码构建

TODO



## 设置 Azkaban Solo Server

TODO

#### 安装 Solo Server

#### 获取 KeySotre 用于SSL（可选）

#### 设置用户管理

#### 运行 Web Server



## 设置数据库

目前 Azkaban2 仅支持 MySQL 作为数据存储，尽管我们正在评估其他可能的存储系统。

#### 1. 安装 MySQL

本说明手册不会讲解如何安装 MySQL 数据库，不过你可以访问 [MySQL 在线文档](http://dev.mysql.com/doc/index.html) 上的教程。

#### 2. 配置数据库

为 Azkaban [创建一个数据库](http://dev.mysql.com/doc/refman/5.7/en/create-database.html)，例如：

```mysql
# Example database creation command, although the db name doesn't need to be 'azkaban'
mysql> CREATE DATABASE azkaban;
```

为 Azkaban [创建一个数据库用户](http://dev.mysql.com/doc/refman/5.7/en/create-user.html)

```mysql
# Example database creation command. The user name doesn't need to be 'azkaban'
mysql> CREATE USER 'username'@'%' IDENTIFIED BY 'password';
```

为用户[设置在数据库上的操作权限](http://dev.mysql.com/doc/refman/5.7/en/grant.html)。如果还没有用户，则为 Azkaban 创建一个，并允许其对 Azkaban 数据库中的所有表进行 **INSERT**，**SELECT**，**UPDAT**，**DELETE** 操作。

```mysql
# Replace db, username with the ones created by the previous steps.
mysql> GRANT SELECT,INSERT,UPDATE,DELETE ON <database>.* to '<username>'@'%' WITH GRANT OPTION;
```

数据包大小可能也需要配置，MySQL 默认允许非常小的数据包。你可能需要把参数 **max_allowed_packet** 设置为一个更高的值，比如 1024M。

在 Linux 上配置该参数需要打开 **/etc/my.cnf** ，在 **mysqld**  下面添加如下内容：

```mysql
[mysqld]
...
max_allowed_packet=1024M
```

通过以下命令来重启 MySQL：

```shell
$ sudo /sbin/service mysqld restart
```

#### 3. 创建 Azkban 数据表

从[下载页面](http://azkaban.github.io/downloads.html) 下载 azkaban-sql-script 压缩包，里面包含了数据表的创建脚本。

在 MySQL 实例上分别执行各个创建表的脚本，或者简单的运行 **create-all-sql** 脚本。任何以 **update** 作为前缀的脚本将会被忽略不执行。

#### 4. 获取 JDBC 链接包（JDBC Connector Jar）

出于一些原因，Azkaban 中不提供 JDBC Connector Jar， 你可以从[该链接](http://www.mysql.com/downloads/connector/j/)下载。

该 jar 包 web server 和 executro sever 都需要使用，要放到这两个服务的 /extlib 目录下。



## 设置 Azkaban Web Server

Azkaban Web Server 负责工程管理，认证，任务调度和运行的触发。

#### 安装 Web Server

从[下载页面](http://azkaban.github.io/downloads.html)获取 azkaban-web-server 安装包。或者你可以 clone 这个 [GithHub 代码库](https://github.com/azkaban/azkaban2)，然后从 master 分支的最新版代码中构建。[参看这里](http://azkaban.github.io/azkaban/docs/latest/#building-from-source)的关于从源码构建的教程说明。

将安装包解压到一个目录，安装路径要跟 AzkabanExecutorServer 不一样。解压后，应该会有如下目录：

| 文件夹     | 说明                                       |
| ------- | ---------------------------------------- |
| bin     | 启动 Azkaban jetty 服务的脚本                   |
| conf    | Azkaban solo server 的配置文件                |
| lib     | Azkaban 依赖的 jar 包                        |
| extlib  | 此目录下的 jar 包会被添加到 Azkaban 的 classpath     |
| plugins | 插件安装目录                                   |
| web     | Azkaban web server 所需的 web 文件（ css，javascript，image） |

在 conf 目录下，会有三个文件：

- **azkaban.properteis** - Azkaban 的运行时参数

- **global.properties** - 全局静态参数，每个工作流和任务都共享其中的配置

- **azkaban-users.xml** - 添加用户和角色供认证使用。若 **XmlUserManager** 没有设置使用该文件，则其不会被用到。

  其中 **azkaban.properties** 是设置 Azkaban 所必需的主配置文件。

#### 为 SSL 获取 KeyStore

todo

#### 设置DB

为了让 Azkaban web client 找到 MySQL 实例，需要在 **azkaban.properties** 中添加链接参数：

```properties
database.type=mysql
mysql.port=3306
mysql.host=localhost
mysql.database=azkaban
mysql.user=azkaban
mysql.password=azkaban
mysql.numconnections=100
```

目前 MySQL 是 Azkaban 唯一支持的数据存储。所以 **database.type** 参数也应该为 **mysql**.

#### 设置用户管理

Azkaban 使用 **UserMananger** 提供认证和用户角色功能。

Azkaban 默认使用内置的 **XmlUserMananger** ，其会从 **azkaban-users.xml** 中获取用户名/密码和角色配置。

也可以从 **azkaban.properties** 文件中看出：

```properties
user.manager.class=azkaban.user.XmlUserManager
user.manager.xml.file=conf/azkaban-users.xml
```

#### 运行 Web Server

在 **azkaban.properties** 中的如下参数用于配置 jetty :

```properties
jetty.maxThreads=25
jetty.ssl.port=8443
```

执行 **bin/azkaban-web-start.sh** 启动 AzkabanWebServer，运行 **bin/azkaban-web-shutdown.sh** 关闭。

可以通过浏览器访问 web server 来测试。



## 设置 Azkaban Execuor Server

Azkaban Executor Server 承担工作流和任务的实际运行。

#### 安装 Executor Server
从[下载页面](http://azkaban.github.io/downloads.html)获取 azkaban-web-server 安装包。或者你可以 clone 这个 [GithHub 代码库](https://github.com/azkaban/azkaban2)，然后从 master 分支的最新版代码中构建。[参看这里](http://azkaban.github.io/azkaban/docs/latest/#building-from-source)的关于从源码构建的教程说明。

将安装包解压到一个目录，安装路径要跟 AzkabanExecutorServer 不一样。解压后，应该会有如下目录：

| 文件夹     | 说明                                   |
| ------- | ------------------------------------ |
| bin     | 启动 Azkaban jetty 服务的脚本               |
| conf    | Azkaban solo server 的配置文件            |
| lib     | Azkaban 依赖的 jar 包                    |
| extlib  | 此目录下的 jar 包会被添加到 Azkaban 的 classpath |
| plugins | 插件安装目录                               |

在 **conf** 目录下我们只需配置 **azkaban.properties** ，该文件是设置 Azkaban Executor 所必需的主要配置。

#### 设置DB

todo （该部分内容实际与 web server 中的一样）

#### 配置 AzabanWebServer 和 AzkabanExecutorServer 客户端

Executor server 需要配置一个端口，而 AzabanWebServer 需要知道该端口是多少。

下面的参数需要配置在 AzkabanExecutorServer 的 **azkaban.properties** 中。

```properties
# Azkaban Executor settings
executor.maxThreads=50
executor.port=12321
executor.flow.threads=30
```



##### 单执行器模式

**executor.port** 默认设置为 **12321**，AzkabanWebServer 需要指向该端口。

通过以下参数在 AzkabanWebServer 的 **azkaban.properteis** 中进行配置：

```properties
executor.port=12321
```



##### 多执行器模式

如果要运行在多执行器模式下，我们需要在 webserver 的配置中开启该模式。确认你的 azkaban.properties 配置文件中有如下参数，azkaban.use.multiple.executors 和 azkaban.executorselector.comparator.* 需要这些参数。请注意 **azkaban.use.multiple.executors ** 不是？？

```properties
azkaban.use.multiple.executors=true
azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus
azkaban.executorselector.comparator.NumberOfAssignedFlowComparator=1
azkaban.executorselector.comparator.Memory=1
azkaban.executorselector.comparator.LastDispatched=1
azkaban.executorselector.comparator.CpuUsage=1
```



#### 运行 Executor Server

- 启动：**bin/azkaban-exec-start.sh**
- 关闭：**bin/azkaban-exec-shutdown.sh**



## 为多执行器模式配置执行器

现阶段我们还没有一个执行器管理的界面，执行器需要在配置在数据库中。例如：- 

将所有执行器信息写入 MySQL DB 来完成设置，要确保正确的执行器在数据表中的记录处于激活状态。

```mysql
insert into executors(host,port) values("EXECUTOR_PORT",EXECUTOR_PORT);
```



## 设置 Azkaban 插件

Azkaban 的非核心功能设计为基于插件的方式来实现，则：

1. 插件可以在不同的环境中选择性地被安装、升级，而无须改动 Azkaban 的核心部分，并且
2. 插件式的设计使 Azkaban 非常易于扩展以适配其他系统

现在，Azkaban 有很多不同的插件。在 web 服务端，有：

- 浏览插件：
- 触发器插件：
- 用户管理插件：
- 告警插件：
- AzkabanExecutorServer 插件化的任务类型

### 用户管理插件



### 浏览插件

#### HDFS浏览插件



### 任务类型插件



### 属性重载



## 从2.1版本升级DB

## 从2.7.0版本升级DB