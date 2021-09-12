# Xhprof

## 安装

### 安装PHP扩展

- xhprof



### 安装`xhprof`

```
$ git clone https://github.com/longxinH/xhprof
```

## 使用

### 配置`nginx`

```nginx
server {
    listen       80 ;
    server_name  localhost.xhprof.com;
    root   /www/xhprof/xhprof_html;
    index  index.php index.html index.htm;
    #charset koi8-r;
    
    access_log /dev/null;
    #access_log  /var/log/nginx/nginx.localhost.laravel53.com.access.log  main;
    error_log  /var/log/nginx/nginx.localhost.xhprof.com.error.log  warn;
    
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
localhost.xhprof.com 127.0.0.1
```



### 植入监控代码

```php
<?php
namespace app\common\services;
class Xhprof
{
    // xhprof web root
    const XHPROF_ROOT = '/www/xhprof';
    protected $xhprof_source;
    public function __construct()
    {
      $this->xhprof_source = 'xhprof_source';
    }
    /**
     * 开始监控
     */
    public function enable()
    {
        xhprof_enable();
        return;
    }
    /**
     * 结束监控，打印报告
     */
    public function disable()
    {
        $xhprof_data = xhprof_disable();
        require_once self::XHPROF_ROOT.'/xhprof_lib/utils/xhprof_lib.php';
        require_once self::XHPROF_ROOT.'/xhprof_lib/utils/xhprof_runs.php';
        $xhprof_runs = new \XHProfRuns_Default();
        $run_id = $xhprof_runs->save_run($xhprof_data, $this->xhprof_source);
        return;
    }
}
```

### 运行监控项目

查看`xhprof`监控平台，访问`http://localhost.xhprof.com`