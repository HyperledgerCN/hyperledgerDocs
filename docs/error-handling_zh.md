
| 原文 | 作者 | 审核修正 |
| --- | --- | --- |
| [原文](http://hyperledger-fabric.readthedocs.io/en/latest/error-handling.html) | Zhangjiong Xuan |  |


## 1.1. 总体概览

The Hyperledger Fabric error handling framework can be found in the source repository under common/errors. It defines a new type of error, CallStackError, to use in place of the standard error type provided by Go.

Fabric的错误处理框架可以在Fabric代码仓库的common/errors目录下找到。它定义了一种新的错误类型，`CallStackError`,用于取代`Go`标准库中实现的错误类型。

A CallStackError consists of the following:

* Component code - a name for the general area of the code that is generating the error. Component codes should consist of three uppercase letters. Numerics and special characters are not allowed. A set of component codes is defined in common/errors/codes.go
* Reason code - a short code to help identify the reason the error occurred. Reason codes should consist of three numeric values. Letters and special characters are not allowed. A set of reason codes is defined in common/error/codes.go
* Error code - the component code and reason code separated by a colon, e.g. MSP:404
* Error message - the text that describes the error. This is the same as the input provided to fmt.Errorf() and Errors.New(). If an error has been wrapped into the current error, its message will be appended.
* Callstack - the callstack at the time the error is created. If an error has been wrapped into the current error, its error message and callstack will be appended to retain the context of the wrapped error.

一个`CallStackError`包含以下内容：

* `Component code` - 一个生成错误信息的错误码的通用区域的组件名称。Component codes应该由3个大写字母组成。不允许出现数字和特殊字符。一系列component codes被定义在`common/errors/codes.go`文件中。
* `Reason code` - 一个用于在错误出现时定位错误原因的较短的错误码。Reason codes应该由3位数字组成。不允许出现字母和特殊字符。一系列reason codes被定义在`common/errors/codes.go`文件中。
* `Error code` - 由冒号分隔的component code和reason code组成的错误码，例如`MSP：404`。
* `Error message` - 描述错误信息的文本。这与提供的`fmt.Errorf()`和`Errors.New()`类似。如果一个错误被包含到当前的错误中，那么它的错误消息将被附加。
* `Callstack` - 错误出现时的调用堆栈。如果一个错误被包含在当前的错误中，那么它的错误消息和调用堆栈信息会被附加到被包含的错误的上下文中。

The CallStackError interface exposes the following functions:

* Error() - returns the error message with callstack appended
* Message() - returns the error message (without callstack appended)
* GetComponentCode() - returns the 3-character component code
* GetReasonCode() - returns the 3-digit reason code
* GetErrorCode() - returns the error code, which is “component:reason”
* GetStack() - returns just the callstack
* WrapError(error) - wraps the provided error into the CallStackError

`Callstack`接口暴露了以下方法：

* `Error()` - 返回一个带有调用堆栈的错误消息。
* `Message()` - 返回一个错误消息。（不包含调用堆栈信息）。
* `GetComponentCode()` - 返回由3个字母组成的组件代码。
* `GetReasonCode()` - 返回由3个数字组成的错误原因代码。
* `GetErrorCode()` - 返回错误代码，由"component:reason"组成。
* `GetStack()` - 仅返回调用堆栈。
* `WrapError(error)` - 将提供的错误包装进`CallStackError`。

## 1.2. 使用说明

The new error handling framework should be used in place of all calls to fmt.Errorf() or Errors.new(). Using this framework will provide error codes to check against as well as the option to generate a callstack that will be appended to the error message.

应该使用新的错误处理框架来替换所有调用`fmt.Errorf()`或者`Errors.new()`的地方。用新的错误处理框架将提供错误代码以及将附加到错误消息的调用堆栈信息的选项。

Using the framework is simple and will only require an easy tweak to your code.

使用这个错误框架简单易用，只需要简单地调整你的代码。

First, you’ll need to import github.com/hyperledger/fabric/common/errors into any file that uses this framework.

首先，您需要将`github.com/hyperleger/fabric/common/errors`导入到使用此框架的任何文件中。

Let’s take the following as an example from core/chaincode/chaincode_support.go:

以`core/chaincode/chaincode_support.go`为例：
```go
err = fmt.Errorf("Error starting container: %s", err)
```
For this error, we will simply call the constructor for Error and pass a component code, reason code, followed by the error message. At the end, we then call the WrapError() function, passing along the error itself.

对于这个错误，我们将简单地调用Error的构造函数，并传递一个组件代码，原因代码，然后是错误消息。最后，我们调用`WrapError()`函数，传递错误本身。
```go
fmt.Errorf("Error starting container: %s", err)
```
变成
```go
errors.ErrorWithCallstack("CHA", "505", "Error starting container").WrapError(err)
```
You could also just leave the message as is without any problems:

您也可以仅编写错误信息，也不会有任何问题：
```go
errors.ErrorWithCallstack("CHA", "505", "Error starting container: %s", err)
```

With this usage you will be able to format the error message from the previous error into the new error, but will lose the ability to print the callstack (if the wrapped error is a CallStackError).

如果使用这种方法，您将能够将上一个错误消息格式化成一个新的错误，但是将失去打印调用堆栈的能力（如果包装的错误是CallStack）。

A second example to highlight a scenario that involves formatting directives for parameters other than errors, while still wrapping an error, is as follows:

另一个凸显的例子涉及了格式化错误以外的参数指令，同时仍然包含了错误，如下所示：
```go
fmt.Errorf("failed to get deployment payload %s - %s", canName, err)
```
变成
```go
errors.ErrorWithCallstack("CHA", "506", "Failed to get deployment payload %s", canName).WrapError(err)
```

## 1.3. 显示错误消息

Once the error has been created using the framework, displaying the error message is as simple as:

一旦使用框架创建啦错误，显示错误消息将十分简单：
```go
logger.Errorf(err)
```
或者
```go
fmt.Println(err)
```
或者
```go
fmt.Printf("%s\n",err)
```

来自`peer/common/common.go`的一个例子：
```go
errors.ErrorWithCallstack("PER", "404", "Error trying to connect to local peer").WrapError(err)
```
将显示错误消息：
```
PER:404 - Error trying to connect to local peer
Caused by: grpc: timed out when dialing
```
Note
The callstacks have not been displayed for this example for the sake of brevity.

>注意
>>为了简洁起见，本示例尚未展示调用堆栈信息。

## 1.4. Hyperledger Fabric中错误处理的一般准则

* If it is some sort of best effort thing you are doing, you should log the error and ignore it.
* If you are servicing a user request, you should log the error and return it.
* If the error comes from elsewhere, you have the choice to wrap the error or not. Typically, it’s best to not wrap the error and simply return it as is. However, for certain cases where a utility function is called, wrapping the error with a new component and reason code can help an end user understand where the error is really occurring without inspecting the callstack.
* A panic should be handled within the same layer by throwing an internal error code/start a recovery process and should not be allowed to propagate to other packages.

* 如果这是你正在努力做的某种事情，你应该记录错误并忽略它。
* 如果你正在为用户请求提供服务，则应该记录错误并返回。
* 如果错误来自其它地方，你可以选择包装错误。通常，最好不要包装错误，让它原样返回。然而，对于工具函数调用的某些情况，使用component code和reason code来包装可以帮助用户在不检查调用堆栈的情况下了解正真发生错误的位置。
* 一个panic 应该在同一层通过抛出内部错误代码/启动一个恢复进程来处理，而且不允许传播到其他软件包。