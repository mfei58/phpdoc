# 程序启动准备

- 文件自动加载
- 服务容器实例化
- 注册基础绑定
- 注册基础服务提供者
- 注册别名
- 配置路径
- 注册请求核心处理类
- 实例化请求核心处理类

**入口文件**

```php
// index.php
require __DIR__.'/../bootstrap/autoload.php';

$app = require_once __DIR__.'/../bootstrap/app.php';

$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

```

**引导文件**

```php
// bootstrap/app.php
$app = new Illuminate\Foundation\Application(
    realpath(__DIR__.'/../')
);

$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);
```

**应用容器类**

```php
namespace Illuminate\Foundation;
class Application extends Container implements ApplicationContract, HttpKernelInterface
{
 	public function __construct($basePath = null)
    {
        $this->registerBaseBindings();

        $this->registerBaseServiceProviders();

        $this->registerCoreContainerAliases();

        if ($basePath) {
            $this->setBasePath($basePath);
        }
    }
}
```

## 注册基础服务提供者

- 注册事件服务提供者
- 注册路由服务提供者

**应用容器类**

```php
namespace Illuminate\Foundation;
class Application extends Container implements ApplicationContract, HttpKernelInterface
{
 	protected function registerBaseServiceProviders()
    {
        $this->register(new EventServiceProvider($this));

        $this->register(new RoutingServiceProvider($this));
    }
}
```



# 程序加载阶段

- 实例化请求类
- 启动引导程序
- 分发请求

**入口文件**

```php
// index.php
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
```

**Http请求处理类**

```php
namespace Illuminate\Foundation\Http;
class Kernel implements KernelContract
{
	public function handle($request)
    {
        try {
            $request->enableHttpMethodParameterOverride();

            $response = $this->sendRequestThroughRouter($request);
        } catch (Exception $e) {
            $this->reportException($e);

            $response = $this->renderException($request, $e);
        } catch (Throwable $e) {
            $this->reportException($e = new FatalThrowableError($e));

            $response = $this->renderException($request, $e);
        }
        $this->app['events']->fire('kernel.handled', [$request, $response]);

        return $response;
    }
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



## 启动引导程序

- 探测环境
- 加载配置
- 启动日志处理程序
- 启动异常处理程序
- 注册门面
- 注册应用服务提供者
- 启动应用服务提供者

**Http请求处理类**

```php
namespace Illuminate\Foundation\Http;
class Kernel implements KernelContract	
	protected $bootstrappers = [
        'Illuminate\Foundation\Bootstrap\DetectEnvironment',
        'Illuminate\Foundation\Bootstrap\LoadConfiguration',
        'Illuminate\Foundation\Bootstrap\ConfigureLogging',
        'Illuminate\Foundation\Bootstrap\HandleExceptions',
        'Illuminate\Foundation\Bootstrap\RegisterFacades',
        'Illuminate\Foundation\Bootstrap\RegisterProviders',
        'Illuminate\Foundation\Bootstrap\BootProviders',
    ];
    public function bootstrap()
    {
        if (! $this->app->hasBeenBootstrapped()) {
            $this->app->bootstrapWith($this->bootstrappers());
        }
    }
```

**应用容器类**

```php
namespace Illuminate\Foundation;
class Application extends Container implements ApplicationContract, HttpKernelInterface
{
   public function bootstrapWith(array $bootstrappers)
    {
        $this->hasBeenBootstrapped = true;
        foreach ($bootstrappers as $bootstrapper) {
            $this['events']->fire('bootstrapping: '.$bootstrapper, [$this]);

            $this->make($bootstrapper)->bootstrap($this);

            $this['events']->fire('bootstrapped: '.$bootstrapper, [$this]);
        }
    }
}
```



# 程序响应阶段

```php
$response->send();
```



# 程序结束阶段

```php
$kernel->terminate($request, $response);
```