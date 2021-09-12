# Xhgui

## 安装

### 安装PHP扩展

- xhprof
- mongodb



### 安装`xhgui`

```
git clone git@github.com:perftools/xhgui.git
```



## 使用

### 配置`mongodb`

```
# mongo -uroot -p123456
> use xhprof
> db.results.ensureIndex( { 'meta.SERVER.REQUEST_TIME' : -1 } )
> db.results.ensureIndex( { 'profile.main().wt' : -1 } )
> db.results.ensureIndex( { 'profile.main().mu' : -1 } )
> db.results.ensureIndex( { 'profile.main().cpu' : -1 } )
> db.results.ensureIndex( { 'meta.url' : 1 } )
```

### 配置`xhgui`

```php
// config/config.php
    // mongodb host
    'db.host' => 'mongodb://xhgui-mongodb:27017',
    // mongodb database
    'db.db' => 'xhprof',
    // 'username', 'password' and 'db' (where the user is added)
    'db.options' => array(),
```

### 安装`composer`依赖

```
composer install --prefer-dist
```



### 配置`nginx`

```nginx
server {
    listen       80 ;
    server_name  localhost.xhgui.com;
    root   /www/xhgui/webroot;
    index  index.php index.html index.htm;
    #charset koi8-r;
    
    access_log /dev/null;
    #access_log  /var/log/nginx/nginx.localhost.xhgui.com.access.log  main;
    error_log  /var/log/nginx/nginx.localhost.xhgui.com.error.log  warn;
    
    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    #location = /50x.html {
    #    root   /usr/share/nginx/html;
    #}
    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        fastcgi_pass   sample-php:9000;
        include        fastcgi-php.conf;
        include        fastcgi_params;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

### 配置用户组

```
chown -R www-data:www-data xhgui
```



### 添加域名映射

```
localhost.xhgui.com 127.0.0.1
```



### 植入监控程序

```php
<?php
namespace app\common\services;
class Xhgui
{
    // xhgui web root
    const XHGUI_ROOT = '/www/xhgui-branch';
    public function __construct()
    {
    }
	/**
     * 开始监控
     */
    public function enable()
    {
        $this->_xguiEnabled();
    }
    /**
     * 结束监控，打印报告
     */
    public function disable()
    {
        $this->_xhguidisabled();
    }
    private function _xguiEnabled()
    {
        if(!is_dir(self::XHGUI_ROOT)){
            return;
        }
        if (!extension_loaded('xhprof') && !extension_loaded('uprofiler') && !extension_loaded('tideways') && !extension_loaded('tideways_xhprof')) {
            error_log('xhgui - either extension xhprof, uprofiler or tideways must be loaded');
            return;
        }

        require_once self::XHGUI_ROOT.'/src/Xhgui/Config.php';
        require_once self::XHGUI_ROOT.'/src/bootstrap.php';
        \Xhgui_Config::load(self::XHGUI_ROOT.'/config/config.php');
        unset($dir);
        if(\Xhgui_Config::read('debug'))
        {
            ini_set('display_errors',1);
        }

        $filterPath = \Xhgui_Config::read('profiler.filter_path');

        if(is_array($filterPath)&&in_array($_SERVER['DOCUMENT_ROOT'],$filterPath)){
            return;
        }

        if ((!extension_loaded('mongo') && !extension_loaded('mongodb')) && \Xhgui_Config::read('save.handler') === 'mongodb') {
            error_log('xhgui - extension mongo not loaded');
            return;
        }

        if (!\Xhgui_Config::shouldRun()) {
            return;
        }

        if (!isset($_SERVER['REQUEST_TIME_FLOAT'])) {
            $_SERVER['REQUEST_TIME_FLOAT'] = microtime(true);
        }
        xhprof_enable(XHPROF_FLAGS_CPU | XHPROF_FLAGS_MEMORY);
    }
    private function _xhguidisabled()
    {
	   if(!is_dir(self::XHGUI_ROOT)){
            return;
        }
        $data['profile'] = xhprof_disable();

        // ignore_user_abort(true) allows your PHP script to continue executing, even if the user has terminated their request.
        // Further Reading: http://blog.preinheimer.com/index.php?/archives/248-When-does-a-user-abort.html
        // flush() asks PHP to send any data remaining in the output buffers. This is normally done when the script completes, but
        // since we're delaying that a bit by dealing with the xhprof stuff, we'll do it now to avoid making the user wait.
        ignore_user_abort(true);
        flush();

        $uri = array_key_exists('REQUEST_URI', $_SERVER)
            ? $_SERVER['REQUEST_URI']
            : null;
        if (empty($uri) && isset($_SERVER['argv'])) {
            $cmd = basename($_SERVER['argv'][0]);
            $uri = $cmd . ' ' . implode(' ', array_slice($_SERVER['argv'], 1));
        }

        $time = array_key_exists('REQUEST_TIME', $_SERVER)
            ? $_SERVER['REQUEST_TIME']
            : time();
        $requestTimeFloat = explode('.', $_SERVER['REQUEST_TIME_FLOAT']);
        if (!isset($requestTimeFloat[1])) {
            $requestTimeFloat[1] = 0;
        }

        if (\Xhgui_Config::read('save.handler') === 'file') {
            $requestTs = array('sec' => $time, 'usec' => 0);
            $requestTsMicro = array('sec' => $requestTimeFloat[0], 'usec' => $requestTimeFloat[1]);
        } else {
            $requestTs = new \MongoDate($time);
            $requestTsMicro = new \MongoDate($requestTimeFloat[0], $requestTimeFloat[1]);
        }

        $data['meta'] = array(
            'url' => $uri,
            'SERVER' => $_SERVER,
            'get' => $_GET,
            'env' => $_ENV,
            'simple_url' => \Xhgui_Util::simpleUrl($uri),
            'request_ts' => $requestTs,
            'request_ts_micro' => $requestTsMicro,
            'request_date' => date('Y-m-d', $time),
        );

        try {
            $config = \Xhgui_Config::all();
            $config += array('db.options' => array());
            $saver = \Xhgui_Saver::factory($config);
            $saver->save($data);
        } catch (\Exception $e) {
            error_log('xhgui - ' . $e->getMessage());
        }
    }
}
```



### 运行监控项目

查看`xhgui`监控平台，访问`http://localhost.xhgui.com`


