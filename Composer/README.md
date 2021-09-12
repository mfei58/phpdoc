[TOC]
# 介绍
## 本节学习目标
- 了解什么是组件化开发
- 学习Composer安装使用
- 了解包依赖、包版本约束、平台包、扩展包
- 了解自动加载
- 了解版本库、资源库
- 学会使用和开发组件

## 组件化开发
在大型软件项目中，组件化开发思想逐渐兴起并快速流行起来，开源文化的流行，让“不要重复发明轮子”影响着越来越多的开发者，正是由于这样，PHP有了统一的PSR编码规范，新的框架也迅速崛起，组件化开发衍生出许多优秀的设计者、艺术家，现如今，这种新的开发模式已经成为绝大多数项目不可或缺的一部分。
## 安装
### 通过官网安装
```
curl -sS https://getcomposer.org/installer | php \
&& mv composer.phar /usr/local/bin/composer \
&&　chmod +x /usr/local/bin/composer
```
### 通过镜像安装
```
curl -o /usr/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x /usr/bin/composer
```
# 快速使用
## 组件安装
添加composer.json，执行composer install
```
{
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```

## 自动加载
对于库的自动加载信息，Composer 生成了一个 vendor/autoload.php 文件。你可以简单的引入这个文件，你会得到一个免费的自动加载支持。
```
require 'vendor/autoload.php';
```
### 注册autoloader
Composer 将注册一个 PSR-4 autoloader 到 Acme 命名空间。Acme映射当前路径下的src目录
```
{
    "autoload": {
        "psr-4": {"Acme\\": "src/"}
    }
}
```
## 包版本约束

| 名称         | 实例                                    | 描述                                                         |
| :----------- | :-------------------------------------- | :----------------------------------------------------------- |
| 确切的版本号 | `1.0.2`                                 | 你可以指定包的确切版本。                                     |
| 范围         | `>=1.0` `>=1.0,<2.0` `>=1.0,<1.1|>=1.2` | 通过使用比较操作符可以指定有效的版本范围。 有效的运算符：`>`、`>=`、`<`、`<=`、`!=`。 你可以定义多个范围，用逗号隔开，这将被视为一个**逻辑AND**处理。一个管道符号`|`将作为**逻辑OR**处理。 AND 的优先级高于 OR。 |
| 通配符       | `1.0.*`                                 | 你可以使用通配符`*`来指定一种模式。`1.0.*`与`>=1.0,<1.1`是等效的。 |
| 赋值运算符   | `~1.2`                                  | 这对于遵循语义化版本号的项目非常有用。`~1.2`相当于`>=1.2,<2.0`。想要了解更多，请阅读下一小节。 |
| 折音号       | ^1.2.3\|^0.3                            | `^`锁定不允许变的第一位，例如，`^1.2.3`相当于`>=1.2.3, <2.0.0`，对于1.0之前的版本，这种约束方式也考虑到了安全问题，例如`^0.3`会被当作`>=0.3.0 <0.4.0` |

## 设置镜像加速
### 全局设置
```
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```
### 当前项目
```
composer config repo.packagist composer https://mirrors.aliyun.com/composer/
```
### 解除镜像
```
composer config -g --unset repos.packagist
```
## 平台软件包
Composer 将那些已经安装在系统上，但并不是由 Composer 安装的包视为一个虚拟的平台软件包。这包括PHP本身，PHP扩展和一些系统库。
### 查看可用的平台软件包的列表。
```
composer show --platform
```
- php 表示用户的 PHP 版本要求，你可以对其做出限制。例如 >=5.4.0。如果需要64位版本的 PHP，你可以使用 php-64bit 进行限制。
- ext-<name> 可以帮你指定需要的 PHP 扩展（包括核心扩展）。通常 PHP 拓展的版本可以是不一致的，将它们的版本约束为 * 是一个不错的主意。一个 PHP 扩展包的例子：包名可以写成 ext-gd。
- lib-<name> 允许对 PHP 库的版本进行限制。
以下是可供使用的名称：curl、iconv、icu、libxml、openssl、pcre、uuid、xsl。



## 常用命令

### 用法
```
command [options] [arguments]
```
### 选项说明
| 名称        | 说明                       |
| :---------- | :------------------------- |
| -h          | 帮助                       |
| -V          | 版本                       |
| -v\|vv\|vvv | v 正常 vv 更多  vvv  debug |

### 参数说明
| 名称           | 说明           |
| :------------- | :------------- |
| init           | 初始化         |
| install        | 安装           |
| update         | 更新           |
| require        | 声明依赖       |
| search         | 搜索           |
| show           | 展示           |
| depends        | 依赖性检测     |
| validate       | 有效性检测     |
| status         | 依赖包状态检测 |
| self-update    | 自我更新       |
| config         | 更改配置       |
| global         | 全局执行       |
| create-project | 创建项目       |
| dump-autoload  | 更新自动加载   |
| diagnose       | 诊断           |
| archive        | 归档           |
| run-script     | 执行脚本       |
| help           | 帮助           |

# 手动构建Laravel框架
## 项目初始化
运行如下命令，添加依赖包：illuminate/routing:~5.3，illuminate/events:~5.3
```
composer init
```
## 添加路由组件
添加入口文件
```php
// public/index.php
<?php
require __DIR__.'/../vendor/autoload.php';
$app = new \Illuminate\Container\Container();
with(new \Illuminate\Events\EventServiceProvider($app))->register();
with(new \Illuminate\Routing\RoutingServiceProvider($app))->register();
require_once __DIR__.'/../app/Http/routes.php';
$request = \Illuminate\Http\Request::createFromGlobals();
$response = $app['router']->dispatch($request);
$response->send();
```
添加路由配置
```php
// app/Http/routes.php
$app['router']->get('/',function (){
   return '<h1>router success</h1>';
});
```

访问Web服务器

## 添加控制器模块
添加自动加载
```php
	"autoload": {
        "psr-4": {
            "App\\": "app"
        }
    }
```
更新自动加载
```
composer dump-autoload
```
添加控制器
```
// app/Http/Controllers/WelcomeController.php
<?php
namespace App\Http\Controllers;
use App\Models\Product;
use Illuminate\Container\Container;

class WelcomeController
{
    public function index()
    {
        return 'Controller success';
    }
}
```
添加路由配置
```php
// app/Http/routes.php
$app['router']->get('/welcome','App\Http\Controllers\WelcomeController@index');
```
## 添加模型组件
> 假设已安装数据库服务，已创建数据库laravel_test，已创建数据表product

添加illuminate/database依赖包
```
composer require illuminate/database:~5.3
```
更新入口文件
```php
// public/index.php
<?php
require __DIR__.'/../vendor/autoload.php';
$app = new \Illuminate\Container\Container();
with(new \Illuminate\Events\EventServiceProvider($app))->register();
with(new \Illuminate\Routing\RoutingServiceProvider($app))->register();
// 实例化数据库管理类，启动Eloquent ORM
$manage = new \Illuminate\Database\Capsule\Manager();
$manage->addConnection(require  __DIR__.'/../config/database.php');
$manage->bootEloquent();
require_once __DIR__.'/../app/Http/routes.php';
$request = \Illuminate\Http\Request::createFromGlobals();
$response = $app['router']->dispatch($request);
$response->send();
```

添加数据库配置文件
```php
// config/database.php
<?php
return [
  'driver'=>'mysql',
  'host'=>'127.0.0.1',
  'database'=>'laravel_test',
  'username'=>'root',
  'password'=>'root',
  'charset'=>'utf8',
  'collection'=>'utf8_general_ci',
  'prefix'=>'',
];
```

添加模型文件
```php
// app/Models/Product.php
<?php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
class Product extends Model
{
    protected $table = 'product';
    public $timestamps=false;
}
```

访问数据
```
// app/Http/Controllers/WelcomeController.php
<?php
namespace App\Http\Controllers;
use App\Models\Product;
use Illuminate\Container\Container;

class WelcomeController
{
    public function index()
    {
        $data = Product::first()->getAttributes();
        return data;
    }
}
```
## 添加视图组件
添加illuminate/view依赖包
```
composer require illuminate/view:~5.3
```
更新入口文件
```php
// public/index.php
<?php
require __DIR__.'/../vendor/autoload.php';

$app = new \Illuminate\Container\Container();
\Illuminate\Container\Container::setInstance($app);
with(new \Illuminate\Events\EventServiceProvider($app))->register();
with(new \Illuminate\Routing\RoutingServiceProvider($app))->register();
// 实例化数据库管理类，启动Eloquent ORM
$manage = new \Illuminate\Database\Capsule\Manager();
$manage->addConnection(require  __DIR__.'/../config/database.php');
$manage->bootEloquent();
// 添加视图模板文件和编译文件的存储路径，对视图进行相关配置和服务注册
$app->instance('config',new \Illuminate\Support\Fluent());
$app['config']['view.compiled'] = "/www/lara/storage/framework/";
$app['config']['view.paths'] = ["/www/lara/resources/views/"];
with(new \Illuminate\View\ViewServiceProvider($app))->register();
with(new \Illuminate\Filesystem\FilesystemServiceProvider($app))->register();
require_once __DIR__.'/../app/Http/routes.php';
$request = \Illuminate\Http\Request::createFromGlobals();
$response = $app['router']->dispatch($request);
$response->send();
```
创建编译文件目录
```
mkdir -p storage/framework
```
创建视图模板
```php
// resources/views/welcome.blade.php
id:{{$data['id']}}</br>
name:{{$data['name']}}</br>
price:{{$data['price']}}</br>
```
使用视图文件
```php
// app/Http/Controllers/WelcomeController.php
<?php
namespace App\Http\Controllers;
use App\Models\Product;
use Illuminate\Container\Container;

class WelcomeController
{
    public function index()
    {
        $data = Product::first()->getAttributes();
        $app = Container::getInstance();
        $factory = $app->make('view');
        return $factory->make('welcome')->with('data',$data);
    }
}
```

# 发布自己的包
## 创建Github项目
上传项目
```
git add .
git commit -m "Initial commit"
git push
```
发布版本1.0.0
```
git tag -a 1.0.0 -m "1.0版本"
git push origin 1.0.0
```

## 发布Packagist包
```
// https://packagist.org/packages/submit
Repository URL :Git/Svn/Hg
```

## 通过Hook自动发布

配置 webhook
```
// https://github.com/username/project/settings/hooks
Payload URL:https://packagist.org/api/github?username=username
Content type:application/json
Secret:packagist API token
```