# 创建流（Flow）

一个任务就是一个你想要在 Azkaban 中运行的一个处理过程。任务可以设置为被其他任务所依赖。一组任务以及它所依赖的组件一起，组成一个流（Flow）。

### 创建任务

创建一个任务很简单。我们创建一个扩展名为 **.job** 的属性文件，这个任务文件中定义了要执行的任务类型、任务所依赖的组件，以及正确设置你的任务所需要的一切参数。

```properties
# foo.job
type=command
command=echo "Hello World"
```

上述例子中，任务的类型是 **command**，另一个 **command** 参数是 “**command**” 这种任务类型所需的。此例中，任务会执行命令并打印出 “**Hello world**"。任务运行时的输出和错误信息会写入日志中，可以在 Azkaban Web 界面中查看。

要了解更多关于任务的信息，参看[任务配置](http://azkaban.github.io/azkaban/docs/latest/#job-configuration)页面

### 创建流

一个流是一组彼此依赖的任务。当前任务所依赖的任务总是先执行。通过 **dependencies** 属性来添加依赖到一个任务中，如下所示：

```properties
# foo.job
type=command
command=echo foo
```

```properties
# bar.job
type=command
dependencies=foo
command=echo bar
```

**dependencies** 参数使用一个由任务名组成的“,“分隔的列表。需要确保任务名字是真实存在的并且没有环状的依赖关系。

没有被其他任务所依赖的任务，则系统会自动创建一个与该任务同名的流。比如，在上面的例子中，**bar** 任务依赖于 **foo** 任务，但是没有其他任务依赖于 **bar** ，所以，一个名为 **bar** 的流会被自动创建。

### 嵌入式流

![嵌入的流](http://azkaban.github.io/azkaban/docs/latest/images/embedded-flow.png)

一个流也可以作为一个节点被包含在别的流中，称之为 **嵌入式流**。创建一个定义了 **type=flow** 和 **flow.name** 的**.job** 文件，即可创建一个**嵌入式流**，如下：

```properties
# baz.job
type=flow
flow.name=bar
```

通过在每个嵌入流的 **.job** 文件中添加不同的参数，可以实现多次嵌入同一个流但每次都使用不同的设置。

### 上传流

要上传流，需要将你所有的 **.job** 文件和任何需要运行的二进制文件打包到一个 **.zip** 文件中。然后通过 Azkaban 的用户界面，你可以部署你的工作流。系统会校验该流是否缺少依赖或存在环状依赖。详见[上传工程](http://azkaban.github.io/azkaban/docs/latest/#project-uploads)。



## 任务配置

### 通用参数

除了 **type** 和 **dependencies** 参数，还有许多 Azkaban 预留的参数。下面所有的参数都是可选的。

| 参数             | 说明                              |
| -------------- | ------------------------------- |
| retries        | 对失败任务的自动重试次数                    |
| retry.backoff  | 重试的时间间隔毫秒数                      |
| working.dir    | 覆盖任务执行的工作目录配置，默认为包含了要运行的任务所在的目录 |
| env.property   | 通过参数名字来设置环境变量                   |
| failure.emails | 逗号分隔的邮件地址列表，用于通知执行失败            |
| success.emails | 逗号分隔的邮件地址列表，用于通知执行成功            |
| notify.emails  | 逗号分隔的邮件地址列表，用于通知执行成功和失败         |

> ### Email 属性
>
> 注意，对于邮件相关的属性，？？
>
> todo

### 运行时属性

这些属性会在运行时自动加入 Azkaban 的属性集合供任务来使用。

todo

| 参数   | 说明   |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |
|      |      |



### 继承的参数



### 参数替换



### 参数传输



### 参数输出



## 内置任务类型

###  Command

### Java Process

### Noop

