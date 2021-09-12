## 前言

----

### PHP内置API

一、基本调试API

- `echo` 、`print`、`printf` 
- `print_r`、`var_dump`、`var_export`、`debug_zval_dump`
- `debug_print_backtrace` 、`debug_backtrace`

二、错误选项控制

- `error_reporting`、`display_errors`
- `log_errors`、`error_log`

三、错误抛出和处理

- `trigger_error`
- `set_error_handler`
- `set_exception_handler`
- `register_shutdown_function`



### Laravel 使用

一、错误细节显示（`.env`、`.env.environment`）

```
APP_DEBUG=true //除生产环境外，其他环境都可以设置为true来控制浏览器对错误的显示细节
```

二、日志存储

`Laravel `提供可立即使用的 `single`、`daily`、`syslog `和 `errorlog` 日志模式。例如，如果你想要每天保存一个日志文件，而不是单个文件，则可以在 `config/app.php` 配置文件内设置 `log `变量。

```
'log' => 'daily'
```

三、自定义`Monolog`设置

```php
$monolog = Log::getMonolog();
$monolog->pushHandler(...);
```

四、记录异常

`report` 方法用于记录异常或将异常寄给外部服务如 [Bugsnag](https://bugsnag.com/) 或 [Sentry](https://github.com/getsentry/sentry-laravel) 。你可以可以自定义异常类型报告给不同的方法。

```php
namespace App\Exceptions;

use Exception;
use Illuminate\Auth\AuthenticationException;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
        	//
    	}
        parent::report($exception);
    }
}
```

五、输出异常

`render` 方法负责将异常转换成 `HTTP` 响应发送给浏览器。你也可以根据不同的异常类型渲染不同的输出效果。

```php
namespace App\Exceptions;

use Exception;
use Illuminate\Auth\AuthenticationException;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
	/**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if($exception instanceof ApiException){
            return ...
        }
        return parent::render($request, $exception);
    }
}
```



## 异常加载源码分析

----

`error_reporting`设置错误级别

- -1 ：报告所有 `PHP `错误
- 0 ：关闭所有`PHP`错误报告
- `E_ALL ^ E_NOTICE `：除了 `E_NOTICE`，报告其他所有错误

`set_error_handler`当发生错误的时候，说明用什么函数来处理错误，可以看到`Laravel`会抛出一个异常`ErrorException`

`set_exception_handler` 该函数设置默认的异常处理程序，用于没有用 `try/catch` 块来捕获的异常，`Laravel`会交给`App\Exceptions\Handler`来记录和输出异常

```php
<?php

namespace Illuminate\Foundation\Bootstrap;

use Exception;
use ErrorException;
use Illuminate\Contracts\Foundation\Application;
use Symfony\Component\Console\Output\ConsoleOutput;
use Symfony\Component\Debug\Exception\FatalErrorException;
use Symfony\Component\Debug\Exception\FatalThrowableError;

class HandleExceptions
{
    /**
     * The application instance.
     *
     * @var \Illuminate\Contracts\Foundation\Application
     */
    protected $app;

    /**
     * Bootstrap the given application.
     *
     * @param  \Illuminate\Contracts\Foundation\Application  $app
     * @return void
     */
    public function bootstrap(Application $app)
    {
        $this->app = $app;

        error_reporting(-1);

        set_error_handler([$this, 'handleError']);

        set_exception_handler([$this, 'handleException']);

        register_shutdown_function([$this, 'handleShutdown']);

        if (! $app->environment('testing')) {
            ini_set('display_errors', 'Off');
        }
    }

    /**
     * Convert a PHP error to an ErrorException.
     *
     * @param  int  $level
     * @param  string  $message
     * @param  string  $file
     * @param  int  $line
     * @param  array  $context
     * @return void
     *
     * @throws \ErrorException
     */
    public function handleError($level, $message, $file = '', $line = 0, $context = [])
    {
        if (error_reporting() & $level) {
            throw new ErrorException($message, 0, $level, $file, $line);
        }
    }

    /**
     * Handle an uncaught exception from the application.
     *
     * Note: Most exceptions can be handled via the try / catch block in
     * the HTTP and Console kernels. But, fatal error exceptions must
     * be handled differently since they are not normal exceptions.
     *
     * @param  \Throwable  $e
     * @return void
     */
    public function handleException($e)
    {
        if (! $e instanceof Exception) {
            $e = new FatalThrowableError($e);
        }

        $this->getExceptionHandler()->report($e);

        if ($this->app->runningInConsole()) {
            $this->renderForConsole($e);
        } else {
            $this->renderHttpResponse($e);
        }
    }

    /**
     * Render an exception to the console.
     *
     * @param  \Exception  $e
     * @return void
     */
    protected function renderForConsole(Exception $e)
    {
        $this->getExceptionHandler()->renderForConsole(new ConsoleOutput, $e);
    }

    /**
     * Render an exception as an HTTP response and send it.
     *
     * @param  \Exception  $e
     * @return void
     */
    protected function renderHttpResponse(Exception $e)
    {
        $this->getExceptionHandler()->render($this->app['request'], $e)->send();
    }

    /**
     * Handle the PHP shutdown event.
     *
     * @return void
     */
    public function handleShutdown()
    {
        if (! is_null($error = error_get_last()) && $this->isFatal($error['type'])) {
            $this->handleException($this->fatalExceptionFromError($error, 0));
        }
    }

    /**
     * Create a new fatal exception instance from an error array.
     *
     * @param  array  $error
     * @param  int|null  $traceOffset
     * @return \Symfony\Component\Debug\Exception\FatalErrorException
     */
    protected function fatalExceptionFromError(array $error, $traceOffset = null)
    {
        return new FatalErrorException(
            $error['message'], $error['type'], 0, $error['file'], $error['line'], $traceOffset
        );
    }

    /**
     * Determine if the error type is fatal.
     *
     * @param  int  $type
     * @return bool
     */
    protected function isFatal($type)
    {
        return in_array($type, [E_ERROR, E_CORE_ERROR, E_COMPILE_ERROR, E_PARSE]);
    }

    /**
     * Get an instance of the exception handler.
     *
     * @return \Illuminate\Contracts\Debug\ExceptionHandler
     */
    protected function getExceptionHandler()
    {
        return $this->app->make('Illuminate\Contracts\Debug\ExceptionHandler');
    }
}

```

为了查看加载流程，这里我们可以分析一下`Illuminate\Foundation\Bootstrap\HandleExceptions::bootstrap`的调用栈：

```php
#0 Illuminate\Foundation\Bootstrap\HandleExceptions->bootstrap() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Application.php:203]
#1 Illuminate\Foundation\Application->bootstrapWith() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:254]
#2 Illuminate\Foundation\Http\Kernel->bootstrap() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:145]
#3 Illuminate\Foundation\Http\Kernel->sendRequestThroughRouter() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:117]
#4 Illuminate\Foundation\Http\Kernel->handle() called at [/www/laravel53/public/index.php:53]
```



介绍了系统异常，那么用户自定义的异常是如何处理的呢，回到`Kernel`

```php
namespace Illuminate\Foundation\Http;

use Illuminate\Contracts\Http\Kernel as KernelContract;
class Kernel implements KernelContract
{
	protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);

        Facade::clearResolvedInstance('request');

        $this->bootstrap();

        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
}
```

`Kernel`中通过`Illuminate\Routing\Pipeline`来处理用户请求，该请求中`getSlice`和`getInitialSlice`均通过`try/catch`来捕获异常，捕获到异常后同样交给`App\Exceptions\Handler`来处理。

```php
<?php

namespace Illuminate\Routing;

use Closure;
use Exception;
use Throwable;
use Illuminate\Http\Request;
use Illuminate\Contracts\Debug\ExceptionHandler;
use Illuminate\Pipeline\Pipeline as BasePipeline;
use Symfony\Component\Debug\Exception\FatalThrowableError;

/**
 * This extended pipeline catches any exceptions that occur during each slice.
 *
 * The exceptions are converted to HTTP responses for proper middleware handling.
 */
class Pipeline extends BasePipeline
{
    /**
     * Get a Closure that represents a slice of the application onion.
     *
     * @return \Closure
     */
    protected function getSlice()
    {
        return function ($stack, $pipe) {
            return function ($passable) use ($stack, $pipe) {
                try {
                    $slice = parent::getSlice();
                    $callable = $slice($stack, $pipe);

                    return $callable($passable);
                } catch (Exception $e) {
                    return $this->handleException($passable, $e);
                } catch (Throwable $e) {
                    return $this->handleException($passable, new FatalThrowableError($e));
                }
            };
        };
    }

    /**
     * Get the initial slice to begin the stack call.
     *
     * @param  \Closure  $destination
     * @return \Closure
     */
    protected function getInitialSlice(Closure $destination)
    {
        return function ($passable) use ($destination) {
            try {
                return $destination($passable);
            } catch (Exception $e) {
                return $this->handleException($passable, $e);
            } catch (Throwable $e) {
                return $this->handleException($passable, new FatalThrowableError($e));
            }
        };
    }

    /**
     * Handle the given exception.
     *
     * @param  mixed  $passable
     * @param  \Exception  $e
     * @return mixed
     *
     * @throws \Exception
     */
    protected function handleException($passable, Exception $e)
    {
        if (! $this->container->bound(ExceptionHandler::class) || ! $passable instanceof Request) {
            throw $e;
        }

        $handler = $this->container->make(ExceptionHandler::class);

        $handler->report($e);

        $response = $handler->render($passable, $e);

        if (method_exists($response, 'withException')) {
            $response->withException($e);
        }

        return $response;
    }
}
```



关于错误细节显示`APP_DEBUG`是如何控制的呢，这段代码给出了答案

```php
namespace Illuminate\Foundation\Exceptions;
use Illuminate\Contracts\Debug\ExceptionHandler as ExceptionHandlerContract;

class Handler implements ExceptionHandlerContract
{
 	/**
     * Create a Symfony response for the given exception.
     *
     * @param  \Exception  $e
     * @return \Symfony\Component\HttpFoundation\Response
     */
    protected function convertExceptionToResponse(Exception $e)
    {
        $e = FlattenException::create($e);

        $handler = new SymfonyExceptionHandler(config('app.debug'));

        return SymfonyResponse::create($handler->getHtml($e), $e->getStatusCode(), $e->getHeaders());
    }
}
```

我们查看一下它的调用栈：

```php
#0 Illuminate\Foundation\Exceptions\Handler->convertExceptionToResponse() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Exceptions/Handler.php:155]
#1 Illuminate\Foundation\Exceptions\Handler->prepareResponse() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Exceptions/Handler.php:140]
#2 Illuminate\Foundation\Exceptions\Handler->render() called at [/www/laravel53/app/Exceptions/Handler.php:47]
#3 App\Exceptions\Handler->render() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:81]
#4 Illuminate\Routing\Pipeline->handleException() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:55]
#5 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Middleware/SubstituteBindings.php:41]
#6 Illuminate\Routing\Middleware\SubstituteBindings->handle() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:137]
#7 Illuminate\Pipeline\Pipeline->Illuminate\Pipeline\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:33]
#8 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Middleware/ThrottleRequests.php:49]
#9 Illuminate\Routing\Middleware\ThrottleRequests->handle() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:137]
#10 Illuminate\Pipeline\Pipeline->Illuminate\Pipeline\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:33]
#11 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:104]
#12 Illuminate\Pipeline\Pipeline->then() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Router.php:655]
#13 Illuminate\Routing\Router->runRouteWithinStack() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Router.php:629]
#14 Illuminate\Routing\Router->dispatchToRoute() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Router.php:607]
#15 Illuminate\Routing\Router->dispatch() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:268]
#16 Illuminate\Foundation\Http\Kernel->Illuminate\Foundation\Http\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:53]
#17 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/CheckForMaintenanceMode.php:46]
#18 Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode->handle() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:137]
#19 Illuminate\Pipeline\Pipeline->Illuminate\Pipeline\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:33]
#20 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:104]
#21 Illuminate\Pipeline\Pipeline->then() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:150]
#22 Illuminate\Foundation\Http\Kernel->sendRequestThroughRouter() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:117]
#23 Illuminate\Foundation\Http\Kernel->handle() called at [/www/laravel53/public/index.php:53]
```



## 实战演练

----

添加异常`ApiException`

```
<?php
namespace App\Exceptions;

use Exception;

class ApiException extends Exception
{

}
```

添加控制器，并抛出一个`ApiException` 

```
<?php
namespace Plugins\Test\api\controllers;
use App\Exceptions\ApiException;
use Illuminate\Routing\Controller as BaseController;

class ApiController extends BaseController
{
    public function test()
    {
        throw new ApiException('.............');
    }
}
```

`debug_print_backtrace`调试

```
	public function report(Exception $exception)
    {
        if($exception instanceof ApiException){
            debug_print_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS);exit;
        }
        parent::report($exception);
    }
```

查看调用栈：

```php
#0 App\Exceptions\Handler->report() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:79]
#1 Illuminate\Routing\Pipeline->handleException() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:55]
#2 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Middleware/SubstituteBindings.php:41]
#3 Illuminate\Routing\Middleware\SubstituteBindings->handle() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:137]
#4 Illuminate\Pipeline\Pipeline->Illuminate\Pipeline\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:33]
#5 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Middleware/ThrottleRequests.php:49]
#6 Illuminate\Routing\Middleware\ThrottleRequests->handle() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:137]
#7 Illuminate\Pipeline\Pipeline->Illuminate\Pipeline\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:33]
#8 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:104]
#9 Illuminate\Pipeline\Pipeline->then() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Router.php:655]
#10 Illuminate\Routing\Router->runRouteWithinStack() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Router.php:629]
#11 Illuminate\Routing\Router->dispatchToRoute() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Router.php:607]
#12 Illuminate\Routing\Router->dispatch() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:268]
#13 Illuminate\Foundation\Http\Kernel->Illuminate\Foundation\Http\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:53]
#14 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Middleware/CheckForMaintenanceMode.php:46]
#15 Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode->handle() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:137]
#16 Illuminate\Pipeline\Pipeline->Illuminate\Pipeline\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Routing/Pipeline.php:33]
#17 Illuminate\Routing\Pipeline->Illuminate\Routing\{closure}() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Pipeline/Pipeline.php:104]
#18 Illuminate\Pipeline\Pipeline->then() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:150]
#19 Illuminate\Foundation\Http\Kernel->sendRequestThroughRouter() called at
[/www/laravel53/vendor/laravel/framework/src/Illuminate/Foundation/Http/Kernel.php:117]
#20 Illuminate\Foundation\Http\Kernel->handle() called at [/www/laravel53/public/index.php:53]
```