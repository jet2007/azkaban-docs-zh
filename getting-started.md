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

Azkaban 使用 SSL 套接字连接，要求必须有一个可用的密钥库。你可以按照[此链接](http://docs.codehaus.org/display/JETTY/How+to+configure+SSL中的步骤来创建一个。

密钥库建好之后，Azkaban 中必须配置密钥库的路径和密码。在 **azkaban.properties** 文件中覆盖一下属性：

```properties
jetty.keystore=keystore
jetty.password=password
jetty.keypassword=password
jetty.truststore=keystore
jetty.trustpassword=password
```

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

现在，Azkaban 有很多不同的插件。在 web 服务器端，

- 浏览插件：可以自定义页面来为azkaban增加新特性。已知的实现包括” HDFS 文件系统查看“和”报告“
- 触发器插件：可以自定义触发方法
- 用户管理插件：可以自定义用户认证方式；比如，在 LinkedIn 我们有基于 LDAP 的用户认证
- 告警插件：除了基于邮件的告警之外，还可以使用其他不同的告警方式
- AzkabanExecutorServer 插件化的任务类型

在执行服务器端，

- 可插拔的不同任务类型的执行器，比如为 hadoop 生态系统中各组件提供的任务类型（job types）

为了更好地使用 Azkaban，我们建议安装这些插件。已经有一些可用的通用型的插件，可以在[下载页面](http://azkaban.github.io/downloads.html)中获取。除此之外，你还可以从 [Github 仓库](https://github.com/azkaban/azkaban-plugins) 拷贝源码，然后在不同的插件目录下运行 **ant** 命令来创建压缩包。

下面介绍下如何在 Azkaban 中安装以及使用这些插件。

### 用户管理插件

Azkaban 默认使用使用 XMLUserManager 类，该类基于一个 xml 文件（位于 **conf/azkaban-users.xml**）来实现用户认证。

这种方式不安全且无法支持很多用户。在实际的生产环境中，你需要依靠自己提供的用户管理 class 来满足需求，比如一个基于 LDAP 的实现方式。 **XMLUserManager** 仍可以用于一些特殊的用户账号和权限管理。你能在默认的 **azkaban-users.xml** 文件中找到两个这种场景的例子。

要使用你自己的用户管理类，可在 **Azkaban2-web-server-install-dir/conf/azkaban.properties** 中定义：

```properties
user.manager.class=MyUserManagerClass
```

并将对于的 jar 包放到 **plugins** 目录下。

### 浏览插件

#### HDFS 浏览插件

HDFS 浏览插件需要安装在 AzkabanWebServer 的插件目录，并在 AzkabanWebServer 的配置文件中定义

如，在 **Azkaban2-web-server-install-dir/conf/azkaban.properties** 中：

```properties
viewer.plugins=hdfs
```

会告诉 Azkaban 从  **Azkaban2-web-server-install-dir/plugins/viewer/hdfs** 目录下加载 hdfs 浏览插件。

将 **azkaban-hdfs-viewer** 解压到 AzkabanWebServer 的 **./plugins/viewer** 下，重命名目录为上面所定义的 **hdfs**。

根据 hadoop 的安装方式：

1. 若 Hadoop 安装时未开启安全保护，使用默认配置即可，重启 **AzkabanWebServer** 后可使用 HDFS 查看器。

2. 若 Hadoop 安装时启用了安全保护，以下配置参数不能使用默认值，在插件的配置文件中：

   | 参数                            | 说明                                       |
   | ----------------------------- | ---------------------------------------- |
   | azkaban.should.proxy          | Azkaban 是否代理为另一个用户来查看 hdfs 文件系统，<br/> 而不是 Azkaban 用户，默认为 **true** |
   | hadoop.security.manager.class | 使用的安装管理类，用来与安全认证的 hadoop 集群进行交互，<br/>默认为**azkaban.security.HadoopSecurityManager_H\_1\_0 **（用于 hadoop 1.x） |
   | proxy.user                    | Azkaban 与 kerberos[^20] 和 hadoop 的用户配置，类似于 oozie[^21] 配置，<br/>用于启用了安全设置的 hadoop |
   | proxy.keytab.location         | keytab[^22] 文件的位置，Azkaban 可以为特定的 **proxy.user** 使用 Kerberos 作认证 |

   要了解更多关于 Hadoop 安全性相关的信息，参考[HadoopSecurityManager](http://azkaban.github.io/azkaban/docs/latest/#hadoopsecuritymanager)

### 任务类型插件

Azkaban 有数量有限的内置的任务类型，来运行本地的 unix 命令和简单的 java 程序。大部分情况下，你会想要安装其他额外的任务类型插件，比如：hadoopJava，Pig，Hive，VoldemortBuildAndPush等等。一些常见的类型已经包含在 azkaban-jobtyp 压缩包种了，以下是如何安装：

任务类型插件需要安装在 AzkabanExecutorServer 的插件目录，并在 AzkabanExecutorServer 的配置文件中进行定义，如，在 **Azkaban2-web-server-install-dir/conf/azkaban.properties** 中配置：

```properties
azkaban.jobtype.plugin.dir=plugins/jobtypes
```

以上配置告诉 Azkaban 从  **Azkaban2-exec-server-install-dir/plugins/jobtypes** 目录加载所有的任务类型。

解压插件包到 AzkabanWebServer 的 **./plugins/** 目录下，重命名目录为上面所定义的 **jobtypes**。

以下是运行 Hadoop 任务的常用配置：

| 参数                       | 说明                                       |
| ------------------------ | ---------------------------------------- |
| hadoop.home              | **$HADOOP_HOME** 环境变量设置                  |
| jobtype.global.classpaht | 集群定义的 hadoop 资源文件，诸如 hadoop-core.jar 和 hadoop 配置文件等（ **${hadoop.home}/hadoop-core-1.0.4.jar，${hadoop.home}/conf** ） |

根据 hadoop 的安装方式：

1. 若 Hadoop 的安装未开启安全认证，使用默认配置即可

2. 若 Hadoop 的安装启用了 kerboeros 认证，需要进行如下设置：

   | 参数                            | 说明                                       |
   | ----------------------------- | ---------------------------------------- |
   | hadoop.security.manager.class | 使用的安装管理类，用来与安全认证的 hadoop 集群进行交互，<br/>默认为**azkaban.security.HadoopSecurityManager_H\_1\_0 **（用于 hadoop 1.x） |
   | proxy.user                    | Azkaban 与 kerberos[^20] 和 hadoop 的用户配置，类似于 oozie[^21] 配置，<br/>用于启用了安全设置的 hadoop |
   | proxy.keytab.location         | keytab[^22] 文件的位置，Azkaban 可以为特定的 **proxy.user** 使用 Kerberos 作认证 |

   要了解更多关于 Hadoop 安全的相关信息，参考 HadoopSecurityManager

   最后，启动执行器，观察是否有报错信息，并检查执行器服务器的日志。对于任务类型的插件，执行器会执行一个小测试以便让你知道其是否安装正确。

### 属性（Property）重载

Azkaban 的任务由一系列我们称之为属性（property）的键值对的集合来定义。最终的任务执行中包含哪些属性，是由多个来源所决定的。下表类列出了所有的属性来源以及他们的优先级。需要注意，如果一个属性在多个来源中的都有定义，则从优先级高的来源中取值。

以下属性是用户可见的。一些重复的属性合并组成  **AbstractProcessJob.java** 中的 **jobProps** 

| 属性来源                                     | 描述                                       | 优先级   |
| ---------------------------------------- | ---------------------------------------- | ----- |
| conf 目录下的 global.properties              | Azkaban 安装时设置的管理属性。所有任务类型的全局属性。          | 最低（0） |
| jobtype 目录下的 common.properties           | Azkaban 安装时设置的管理属性。所有任务类型的全局属性。          | 1     |
| jobtype/{jobtype-name} 目录下的 plugin.properties | Azkaban 安装时设置的管理属性。对某个特定的任务类型有效          | 2     |
| 工程 zip 包中的 common.properties             | 用户定义的属性，对位于同级或子级目录中的所有任务有效               | 3     |
| 触发流执行时定义的流属性                             | 用户定义的属性，可以在用户界面或 Ajax 接口调用时定义，但不会保存到项目 zip 包中 | 4     |
| 任务描述文件 {job-name}.job                    | 在实际的任务文件中，用户定义的属性                        | 最高（5） |

以下属性用户不可见，根据不同的任务类型实现，这些属性用于限制用户的任务和属性。一些重复的属性合并组成  **AbstractProcessJob.java** 中的 **sysProps**

| 属性来源                                     | 描述                                       | 优先级   |
| ---------------------------------------- | ---------------------------------------- | ----- |
| jobtype 目录下的 commonprivate.properties    | Azkaban 安装时设置的管理属性。所有任务类型的全局属性。所有任务类型使用的全局属性 | 最低（0） |
| jobtype/{jobtype-name} 目录下的 private.properties | Azkaban 安装时设置的管理属性。所有任务类型的全局属性。对某个特定的任务类型有效 | 最高（1） |

**azkaban.properties** 中定义的是另外一类仅用于控制 Azkaban web服务器和执行服务器配置的属性。需要注意的是， **jobPros**，**sysProps** 和 **azkaban.properties** 是三种不同类型的属性，通常不会被合并（根据不同的任务类型的实现）。

## 从2.1版本升级DB

todo

## 从2.7.0版本升级DB

todo

## 附录

[^20]: Kerberos：一种计算机网络授权协议，其设计目标是通过密钥系统，为C/S 模式的应用程序提供高强度的认证服务
[^21]: oozie：一款 Hadoop 任务调度的开源工作流引擎，由 Cloudera 公司贡献给 Apache 
[^22]: keytab：Kerberos 使用的包含了加密 key 的