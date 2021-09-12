# 前言

## 介绍

`Gitlab`是基于Ruby开发的，以`Git`作为代码管理工具，完全开源的自建式版本控制系统。

## 用途

`Gitlab`是一种跨职能的工作模式（`DevOps`），它为研发、测试和运维提供了理论性指导。

`Gitlab`的优势体现在：

- 组织、计划、协调和跟踪项目工作，以确保团队在正确的时间从事正确的工作，并维护从想法到生产整个交付生命周期中端到端的可见性和问题的可追溯性。
- 分布式设计、开发和安全地管理团队代码和项目数据，以实现快速迭代和业务价值的交付。
- 集成`CI`功能，支持自动化测试、静态分析安全性测试、动态分析安全性测试和代码质量分析，为开发人员和测试人员提供关于代码质量的快速反馈。通过支持并发测试和并行执行的管道，团队可以快速了解每一次提交，从而更快地交付更高质量的代码。
- 集成`CD`解决方案，通过将零接触连续交付(`CD`)内置到管道中，可以将部署自动化到登台和生产等多个环境中，缩短交付生命周期，简化手工流程，并加速团队速度。
- 配置和管理他们的应用程序环境。
- 获取反馈信息和工具，帮助您降低事件的严重程度和频率



# 快速入门

## 安装

### Docker 下安装

```
docker run \
-d \
-p 443:443 \
-p 9201:80 \
-p 9202:22 \
--name gitlab \
--restart always \
-v $(pwd)/gitlab/config:/etc/gitlab \
-v $(pwd)/gitlab/logs:/var/log/gitlab \
-v $(pwd)/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:13.6.1-ce.0
```



# 基础功能

### 基础配置

##### 修改如下：

```ruby
### config/gitlab.rb

## 设置外部访问URL
external_url "http://gitlab.example.com:9201"
## 设置ssh主机地址
gitlab_rails['gitlab_ssh_host'] = 'gitlab.example.com'
## 设置时区
gitlab_rails['time_zone'] = 'Asia/Shanghai'
## 设置GIT数据存放路径
git_data_dirs({
   "default" => {
       "path" => "/var/opt/gitlab/git-data"
   }
})
## 设置ssh端口
gitlab_rails['gitlab_shell_ssh_port'] = 9202
## 设置用户信息
user['username'] = "git"
user['group'] = "git"
user['home'] = "/var/opt/gitlab"
user['git_user_email'] = "mfei58@qq.com"
## 设置ssh授权文件路径
gitlab_shell['auth_file'] = "/var/opt/gitlab/.ssh/authorized_keys"
## 设置邮箱信息
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "mfei58@qq.com"
gitlab_rails['smtp_password'] = "*****"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'none'
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'mfei58@qq.com'
gitlab_rails['gitlab_email_display_name'] = 'Gitlab'
gitlab_rails['gitlab_email_reply_to'] = 'mfei58@qq.com'
gitlab_rails['gitlab_email_subject_suffix'] = '[Gitlab]'
## 设置nginx监听端口
nginx['listen_port'] = 80
```

##### 邮箱测试

```
gitlab-rails console
>Notify.test_email("mfei58@qq.com","test","test").deliver_now
```



##### 重载配置

```
sudo gitlab-ctl reconfigure
```





### 端口检查

#### 查看端口

若`Gitlab`安装正常，则可看到如下服务已启动

```
netstat -tlnp |grep -E '9202|9201'
tcp        0      0 0.0.0.0:9201            0.0.0.0:*               LISTEN      83866/docker-proxy  
tcp        0      0 0.0.0.0:9202            0.0.0.0:*               LISTEN      83906/docker-proxy  
tcp6       0      0 :::9201                 :::*                    LISTEN      83874/docker-proxy  
tcp6       0      0 :::9202                 :::*                    LISTEN      83913/docker-proxy 
```



#### `web`测试

浏览器访问`external_url`，查看网页是否正常



#### `ssh`测试

如果出现`Welcome to GitLab, @root!`，说明`ssh`可以正常登入

```
ssh -p gitlab_shell_ssh_port git@gitlab_ssh_host
Welcome to GitLab, @root!
Connection to gitlab.hsyes.net closed.
```





### 端口转发

#### 目的

通过[Frp](https://gofrp.org/docs/)实现内网穿透，访问内网环境下的`Gitlab`服务

#### 准备

1. 准备一个已备案的域名，如`gitlab.example.com`

2. 准备一个可供外网访问的服务器，假设`IP`地址`111.111.11.111`

3. 将域名解析至该`IP`地址

#### 配置说明

- 将外网域名的`80`端口转发到内网的`9201`
- 将外网`ssh`的`9202`端口转发到内网的`9202`

#### 服务端配置

`vhost_http_port`设置域名访问的端口

```ini
[common]
bind_port = 7000
vhost_http_port=80
```

#### 客服端配置

```ini
[common]
server_addr = 111.111.11.111
server_port = 7000
[ssh218_1]
type = tcp
local_ip = 127.0.0.1
local_port = 9202
remote_port = 9202
[web218_1]
type = http
local_port = 9201
custom_domains = gitlab.example.com
```

#### `Gitlab`配置

将内网配置改写成外网配置

```ruby
## 设置外部访问URL
external_url "http://gitlab.example.com"
## 设置ssh主机地址
gitlab_rails['gitlab_ssh_host'] = 'gitlab.example.com'
## 设置ssh端口
gitlab_rails['gitlab_shell_ssh_port'] = 9202
```



#### `ssh`测试

```
ssh -p 9202 git@gitlab.example.com
Welcome to GitLab, @root!
Connection to gitlab.hsyes.net closed.
```





### 常用命令

#### 服务相关

```
gitlab-ctl start #启动全部服务
gitlab-ctl restart#重启全部服务
gitlab-ctl stop #停止全部服务
gitlab-ctl restart nginx #重启单个服务，如重启nginx
gitlab-ctl status #查看服务状态
```

#### 配置相关

```
gitlab-ctl reconfigure #使配置文件生效
gitlab-ctl show-config #验证配置文件
```

#### 日志相关

```
gitlab-ctl tail <service name> #查看服务的日志
gitlab-ctl tail nginx  #如查看gitlab下nginx日志
```

#### 其他

``` 
gitlab-ctl uninstall #删除gitlab（保留数据）
gitlab-ctl cleanse #删除所有数据，从新开始
gitlab-rails console  #进入控制台
```







# 参考文档

[Gitlab官网](https://about.gitlab.com/stages-devops-lifecycle/)

[GoFrp](https://gofrp.org/docs/)