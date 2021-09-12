# PHP调试技术

> PHP中将代码自身异常（一般是环境或者语法非法所致）称为错误，将运行中出现的逻辑错误称为异常[`Exception`](https://www.php.net/manual/zh/class.exception.php)错误是没法通过代码处理的，而异常则可以通过`try/catch`处理.

"错误"和"异常"的概念十分相似，很容易混淆，"错误"和"异常"都表明了项目出了问题，都会提供相关的信息，并且都有错误类型。然而，"异常机制"是在"错误机制"后才出现的，"异常"是避免"错误"的不足。比较重要的一点就是因为"错误"的信息不丰富，我们见过最多的函数说明就是: 成功时候返回`TRUE`, 错误的时候返回`FALSE`, 然而一个函数出错的原因可能有多种, 出错的种类更有多种. 一个简单的`FALSE`, 并不能把具体的错误信息告诉调用者.



## 异常

----



> 异常是Exception类的对象，在遇到无法修复的状况时抛出，出现问题时，异常用于主动出击，委托职责，异常还可用于防守，预测潜在的问题，减轻其影响。

**异常的属性**

- `message`  错误信息
- `code` 错误编码
- `file` 文件名称
- `line` 行号

**抛出异常**

当一个异常被抛出后代码会立即停止执行，其后的代码将不会继续执行，`PHP` 会尝试查找匹配的 "`catch`" 代码块。如果一个异常没有被捕获，而且又没用使用[`set_exception_handler`](http://www.php.net/manual/zh/function.set-exception-handler.php)作相应的处理的话，那么 `PHP` 将会产生一个严重的错误，并且输出未能捕获异常(**`Uncaught Exception ...`**)的提示信息。

**捕获异常**

我们应该捕获抛出的异常并且使用优雅的方式处理。拦截并处理异常的方式是，把可能抛出异常的代码放到`try/catch`块中。并且如果使用多个`catch`拦截多个异常的时候，只会运行其中一个，如果`PHP`没有找到合适的`catch`块，异常会向上冒泡，直到`PHP`脚本由于致命错误而终止运行。

**异常处理程序**

那么我们应该如何捕获每个可能抛出的异常呢？`PHP`允许我们注册一个全局异常处理程序，捕获所有未被捕获的异常。异常处理程序使用[`set_exception_handler`](http://www.php.net/manual/zh/function.set-exception-handler.php)函数注册。



## 错误

----



> 除了异常之外，`PHP`还提供了用于报告错误的函数。PHP能触发不同类型的错误，例如致命错误、运行时错误、编译时错误、启动错误以及用户触发的错误。可以在`php.ini`中设置错误报告方式。

**常见的错误报告级别**：

|     值      |  常量   | 说明                                                         |
| :---------: | :-----: | ------------------------------------------------------------ |
|    `-1`     |  `-1`   | 报告所有 `PHP` 错误                                          |
|     `0`     |   `0`   | 关闭所有`PHP`错误报告                                        |
|  `E_ERROR`  |   `1`   | 致命的运行时错误。这类错误一般是不可恢复的情况，例如内存分配导致的问题。后果是导致脚本终止不再继续运行。 |
| `E_WARNING` |   `2`   | 运行时警告 (非致命错误)。仅给出提示信息，但是脚本不会终止运行。 |
|  `E_PARSE`  |   `4`   | 编译时语法解析错误。解析错误仅仅由分析器产生。               |
| `E_NOTICE`  |   `8`   | 运行时通知。表示脚本遇到可能会表现为错误的情况，但是在可以正常运行的脚本里面也可能会有类似的通知。 |
|   `E_ALL`   | `32767` | `PHP` `5.4.0` 之前为 **`E_STRICT`** 除外的所有错误和警告信息。 |

**开发规范**：

- 一定要让PHP报告错误
- 在开发环境中要显示错误
- 在生产环境中不能显示错误
- 在开发环境和生产环境中都要记录错误

**错误处理程序**

与异常处理程序一样，我们也可以使用[set_error_handler](https://www.php.net/manual/zh/function.set-error-handler.php)注册全局错误处理程序，使用自己的逻辑方式拦截并处理PHP错误。我们要在错误处理程序中调用`die`或`exit`函数。如果不调用，`PHP`脚本会从出错的地方继续向下执行

**错误转换为异常**

我们可以把PHP错误转换为异常（并不是所有的错误都可以转换,只能转换`php.ini`文件中[`error_reporting`](https://www.php.net/manual/zh/function.error-reporting)指令设置的错误），使用处理异常的现有流程处理错误。这里我们使用[set_error_handler](https://www.php.net/manual/zh/function.set-error-handler.php)函数将错误信息托管至`ErrorException`（它是`Exception`的子类），进而交给现有的异常处系统处理。

## PHP7的错误异常处理

----

> `PHP 7` 改变了大多数错误的报告方式。不同于传统（`PHP 5`）的错误报告机制，现在大多数错误被作为 `Error` 异常抛出。
>
> 这种 `Error` 异常可以像 [`Exception`](http://php.net/manual/zh/class.exception.php) 异常一样被第一个匹配的 `try / catch` 块所捕获。如果没有匹配的 [`catch`](http://php.net/manual/zh/language.exceptions.php#language.exceptions.catch) 块，则调用异常处理函数（事先通过 [`set_exception_handler`()](http://php.net/manual/zh/function.set-exception-handler.php) 注册）进行处理。 如果尚未注册异常处理函数，则按照传统方式处理：被报告为一个致命错误

**Throwable** 

`PHP 7` 里，**`Throwable`** 是能被 [`throw`](https://www.php.net/manual/zh/language.exceptions.php) 语句抛出的最基本的接口（interface），包含了 [`Error`](https://www.php.net/manual/zh/class.error.php) 和 [`Exception`](https://www.php.net/manual/zh/class.exception.php)。

> **注意**:
>
> PHP 类无法直接实现 （`implement`） **`Throwable`** 接口，而应当去继承 [`Exception`](https://www.php.net/manual/zh/class.exception.php)。





## 设计思路

**异常处理器**

----

一、异常记录

全局处理所有错误和异常，并定制化异常报告

二、异常回显

除生产环境外，异常可以以浏览器的形式输出异常，便于开发调试

**系统异常**

----

一、全局异常处理程序

二、全局错误处理程序

三、设置错误处理级别和错误报告

**用户异常**

----

一 、全局异常捕获

二、自定义异常分类



## 项目实战

- [Laravel异常详解]()