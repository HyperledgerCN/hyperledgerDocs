
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/logging-control.html) | Yi Zeng |  |

## Overview概述

Logging in the peer application and in the shim interface to chaincodes is programmed using facilities provided by the github.com/op/go-logging package. This package supports

日志功能在节点的应用程序和链上代码的shim接口中使用，最终在github.com/op/go-logging包实现。这个包支持：

* Logging control based on the severity of the message
    基于消息的严重程度进行日志控制
* Logging control based on the software module generating the message
    基于软件模块产生的消息进行日志控制
* Different pretty-printing options based on the severity of the message
    基于消息的严重程度美观的打印到不同的格式的选项

All logs are currently directed to stderr, and the pretty-printing is currently fixed. However global and module-level control of logging by severity is provided for both users and developers. There are currently no formalized rules for the types of information provided at each severity level, however when submitting bug reports the developers may want to see full logs down to the DEBUG level.

所有日志目前都被定向到stderr，而pretty-printing目前是固定的。然而，为用户和开发人员提供了严格级别的全局和模块级别的日志记录控制。目前没有关于每个严重性级别提供的信息类型的正式规则，但是当提交错误报告时，开发人员可能希望看到完整的到DEBUG级别的日志记录。

In pretty-printed logs the logging level is indicated both by color and by a 4-character code, e.g, “ERRO” for ERROR, “DEBU” for DEBUG, etc. In the logging context a module is an arbitrary name (string) given by developers to groups of related messages. In the pretty-printed example below, the logging modules “peer”, “rest” and “main” are generating logs.

在pretty-printed的日志中，日志记录级别由颜色和4个字符的代码指示，例如ERROR的“ERRO”，DEBUG的“DEBU”等。在日志上下文中，模块是指由开发者指定的任意名称（字符串）的相关消息的组。在以下pretty-printed的例子中，日志模块“peer”，“rest”和“main”都产生了日志。

~~~
16:47:09.634 [peer] GetLocalAddress -> INFO 033 Auto detected peer address: 9.3.158.178:7051
16:47:09.635 [rest] StartOpenchainRESTServer -> INFO 035 Initializing the REST service...
16:47:09.635 [main] serve -> INFO 036 Starting peer with id=name:"vp1" , network id=dev, address=9.3.158.178:7051, discovery.rootnode=, validator=true
~~~

An arbitrary number of logging modules can be created at runtime, therefore there is no “master list” of modules, and logging control constructs can not check whether logging modules actually do or will exist. Also note that the logging module system does not understand hierarchy or wildcarding: You may see module names like “foo/bar” in the code, but the logging system only sees a flat string. It doesn’t understand that “foo/bar” is related to “foo” in any way, or that “foo/*” might indicate all “submodules” of foo.

可以在运行时创建任意数量的日志记录模块，因此没有模块的“主列表”一说，日志控制结构不能检查日志模块是否实际执行或将存在。另请注意，日志记录模块系统不明白层次结构或通配符：您可能会在代码中看到模块名称，如“foo/bar”，但日志记录系统只能看到一个扁平的字符串。它不明白“foo/bar”与“foo”有任何关系，或者“foo/*”可能表示foo的所有“子模块”。

## peer

The logging level of the peer command can be controlled from the command line for each invocation using the --logging-level flag, for example

peer命令的日志等级可以使用命令行控制，每次调用peer时使用--logging-level，例如：

~~~
peer node start --logging-level=debug
~~~

The default logging level for each individual peer subcommand can also be set in the core.yaml file. For example the key logging.node sets the default level for the node subcommmand. Comments in the file also explain how the logging level can be overridden in various ways by using environment varaibles.

每个单独的peer命令的默认日志记录级别也可以在core.yaml文件中设置。例如，键logging.node用于设置node子命令的默认级别。该文中的注释还解释了如何通过使用环境变量以各种方式覆盖日志级别。

Logging severity levels are specified using case-insensitive strings chosen from

使用以下选择的不区分大小写的字符串可以指定日志严重级别：

~~~
CRITICAL | ERROR | WARNING | NOTICE | INFO | DEBUG
~~~

The full logging level specification for the peer is of the form

peer的完整日志级别的规格如下格式：

~~~
[<module>[,<module>...]=]<level>[:[<module>[,<module>...]=]<level>...]
~~~

A logging level by itself is taken as the overall default. Otherwise, overrides for individual or groups of modules can be specified using the

本身的日志级别被视为总体默认值。另外，可以使用以下命令来指定单个或多个模块组的日志等级的覆盖:
~~~
<module>[,<module>...]=<level>
~~~
syntax. Examples of specifications (valid for all of --logging-level, environment variable and core.yaml settings):

语法。规范示例（适用于所有的--logging-level，环境变量和core.yaml设置）：
~~~
info                                       - Set default to INFO
warning:main,db=debug:chaincode=info       - Default WARNING; Override for main,db,chaincode
chaincode=info:main=debug:db=debug:warning - Same as above
~~~

## Go chaincodes

The standard mechanism to log within a chaincode application is to integrate with the logging transport exposed to each chaincode instance via the peer. The chaincode `shim` package provides APIs that allow a chaincode to create and manage logging objects whose logs will be formatted and interleaved consistently with the `shim` logs.

链上代码应用程序中日志的标准机制是通过peer与暴露于每个链码实例的日志传输进行集成。 链上代码的`shim`包提供了API，允许链码创建和管理日志记录对象，日志对象的日志将被格式化，并与`shim`日志交织在了一起。

As independently executed programs, user-provided chaincodes may technically also produce output on stdout/stderr. While naturally useful for “devmode”, these channels are normally disabled on a production network to mitigate abuse from broken or malicious code. However, it is possible to enable this output even for peer-managed containers (e.g. “netmode”) on a per-peer basis via the CORE_VM_DOCKER_ATTACHSTDOUT=true configuration option.

作为独立执行的程序，用户提供的链码在技术上也可以在stdout / stderr上产生输出。虽然对“开发模式”有用，但这种方式通常在生产环境上被禁用，以减轻破坏或恶意代码的滥用。然而，甚至可以通过CORE_VM_DOCKER_ATTACHSTDOUT = true配置选项在每个peer-peer的基础上为peer管理的容器（例如“netmode”）启用此输出。

Once enabled, each chaincode will receive its own logging channel keyed by its container-id. Any output written to either stdout or stderr will be integrated with the peer’s log on a per-line basis. It is not recommended to enable this for production.

一旦启用，每个链码将接收其自己的日志通道，其由container-id标识。写入stdout或stderr的任何输出将与peer的日志按照每行进行集成。不建议将其用于生产。

## API

`NewLogger(name string) *ChaincodeLogger` - Create a logging object for use by a chaincode

`(c *ChaincodeLogger) SetLevel(level LoggingLevel)` - Set the logging level of the logger

`(c *ChaincodeLogger) IsEnabledFor(level LoggingLevel) bool` - Return true if logs will be generated at the given level

`LogLevel(levelString string) (LoggingLevel, error)` - Convert a string to a LoggingLevel

A `LoggingLevel` is a member of the enumeration

    LogDebug, LogInfo, LogNotice, LogWarning, LogError, LogCritical

which can be used directly, or generated by passing a case-insensitive version of the strings

    DEBUG, INFO, NOTICE, WARNING, ERROR, CRITICAL

to the `LogLevel` API.

Formatted logging at various severity levels is provided by the functions

以下函数提供了各种严重级别的格式化日志记录
~~~
(c *ChaincodeLogger) Debug(args ...interface{})
(c *ChaincodeLogger) Info(args ...interface{})
(c *ChaincodeLogger) Notice(args ...interface{})
(c *ChaincodeLogger) Warning(args ...interface{})
(c *ChaincodeLogger) Error(args ...interface{})
(c *ChaincodeLogger) Critical(args ...interface{})

(c *ChaincodeLogger) Debugf(format string, args ...interface{})
(c *ChaincodeLogger) Infof(format string, args ...interface{})
(c *ChaincodeLogger) Noticef(format string, args ...interface{})
(c *ChaincodeLogger) Warningf(format string, args ...interface{})
(c *ChaincodeLogger) Errorf(format string, args ...interface{})
(c *ChaincodeLogger) Criticalf(format string, args ...interface{})
~~~

The `f` forms of the logging APIs provide for precise control over the formatting of the logs. The non-`f` forms of the APIs currently insert a space between the printed representations of the arguments, and arbitrarily choose the formats to use.

日志API的`f`形式可以精确控制日志格式。 API的非`f`形式当前在参数的打印表示之间插入一个空格，并任意选择要使用的格式。

In the current implementation, the logs produced by the `shim` and a `ChaincodeLogger` are timestamped, marked with the logger name and severity level, and written to `stderr`. Note that logging level control is currently based on the name provided when the `ChaincodeLogger` is created. To avoid ambiguities, all `ChaincodeLogger` should be given unique names other than “shim”. The logger name will appear in all log messages created by the logger. The `shim` logs as “shim”.

在当前实现中，由`shim`和`ChaincodeLogger`生成的日志是时间戳的，标有记录器名称和严重性级别，并写入`stderr`。请注意，日志级别控制当前基于创建`ChaincodeLogger`时提供的名称。为了避免歧义，所有`ChaincodeLogger`应该被赋予除“shim”之外的唯一名称。记录器名称将显示在由记录器创建的所有日志消息中。垫片记录为“shim”。

Go language chaincodes can also control the logging level of the chaincode shim interface through the `SetLoggingLevel` API.

Go语言链接代码还可以通过SetLoggingLevel API来控制链式代码垫片界面的日志记录级别。

`SetLoggingLevel(LoggingLevel level)` - Control the logging level of the shim  控制shim的日志记录级别

The default logging level for the shim is `LogDebug`.

shim的默认日志级别为LogDebug。

Below is a simple example of how a chaincode might create a private logging object logging at the `LogInfo` level, and also control the amount of logging provided by the `shim` based on an environment variable.

下面是一个简单的示例，说明链码如何创建`LogInfo`级别的专用日志对象日志记录，并且还可以基于环境变量来控制由`shim`提供的日志量。
~~~
var logger = shim.NewLogger("myChaincode")

func main() {

    logger.SetLevel(shim.LogInfo)

    logLevel, _ := shim.LogLevel(os.Getenv("SHIM_LOGGING_LEVEL"))
    shim.SetLoggingLevel(logLevel)
    ...
}
~~~
