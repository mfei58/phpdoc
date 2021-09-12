# 前言

## 介绍

`JumpServer`是基于`Python / Django`开发的开源的堡垒机，遵循`web 2.0`规范，配备了领先的`Web Terminal`界面，保证了高效率、低成本、低风险的运维综合管理体系。

## 用途

使用`JumpServer`，可以针对运维专员进行安全的统一身份认证，对所有网络设备资源进行统一账号管理，对运维人员进行设备资源操作的管理授权，对用户行为及资源管理进行安全审计



# 快速入门

## 安装

### 自动部署

```
# curl -sSL https://github.com/jumpserver/jumpserver/releases/download/v2.11.2/quick_start.sh | bash
```

### 手动部署

```
# cd /opt
# wget https://github.com/jumpserver/installer/releases/download/v2.11.2/jumpserver-installer-v2.11.2.tar.gz
# tar -xf jumpserver-installer-v2.11.2.tar.gz
# cd jumpserver-installer-v2.11.2
# cat config-example.txt
# ./jmsctl.sh install
```



# 基础功能

## 系统设置

### 邮件设置

```
SMTP 主机: smtp.qq.com
SMTP 端口: 25
SMTP 账号: 你的邮箱账号
SMTP 密码: 邮箱授权码
安全设置: 使用 TLS (587)
发件人: 发送邮件账号
主题前缀: [JMS]
测试收件人: 接收邮件测试账号
```



## 用户管理

### 添加用户组

#### 用户组推荐：

- 团队拥有者：团队的创建者，拥有最高权限
- 团队管理员：由拥有者指定的管理员，一个团队可以有多位管理员
- 应用管理员：可以由团队管理员和团队拥有者指派，负责管理某个应用的资源
- 应用开发者：拥有某项资源的使用权

## 资产管理

### 管理用户

资产（被控服务器）上的 `root` 用户，或拥有 `ALL sudo` 权限的用户，`JumpServer `使用该用户来 推送系统用户、获取资产硬件信息 等。



### 系统用户

又名登入资产（被控服务器）用户，是`JumpServer` 跳转登入资产时使用的用户。



### 资产管理

```
主机名: 主机的昵称
IP(域名): 主机对应的IP地址
系统平台: 可选（Linux、Unix、MacOS、BSD、Windows）
公网IP: 主机对应的公网IP地址
协议: 一般是22
管理用户: 对应资产的root用户
节点: 资产分组的节点
```



## 应用管理

### 数据库管理

```
名称: 数据库昵称
类型: MySQL
主机: 数据库主机地址
端口: 数据库端口号
数据库: 数据库名
```





## 权限管理

### 资产授权

将资产分配给用户和用户组，资产和节点，系统用户



### 应用授权

将应用（数据库）分配给用户和用户组，系统用户



## 会话管理

### 命令记录

记录所有资产对应系统用户的操作命令及操作时间

### 会话管理

记录正在使用或历史使用资产和应用的系统用户的登入来源，登入设备，远端地址，协议，时长，日期。



## 日志审计

### 登入日志

记录用户的登入行为

### 操作日志

记录用户的操作行为

### 改密日志

记录用户账号的改密行为



## 常用命令

### 启动

```
./jmsctl.sh start
```



### 管理

```
Installation Commands: 
  install           安装 JumpServer
  upgrade [version] 升级 JumpServer
  check_update      检查 JumpServer
  reconfig          重新配置 JumpServer

Management Commands: 
  start             启动 JumpServer
  stop              停止 JumpServer
  close             关闭 JumpServer
  restart           重启 JumpServer
  status            检查 JumpServer
  down              下线 JumpServer
  uninstall         卸载 JumpServer

More Commands: 
  load_image        加载 Docker 镜像
  python            运行 python manage.py shell
  backup_db         备份数据库
  restore_db [file] 通过数据库备份文件恢复数据
  raw               执行原始 docker-compose 命令
  tail [service]    查看日志
```



### Web 访问

浏览器访问`http://IP:80`，默认用户: admin  默认密码: admin



### SSH/SFTP 访问

通过命令进行访问

```
ssh -p2222 admin@192.168.10.217
sftp -P2222 admin@192.168.10.217
```



# 参考文档

[JumpServer](https://www.fit2cloud.com/jumpserver/index.html)