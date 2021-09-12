# 前言

# 快速入门

## 安装

### 第一步：后台申请

在[官网](https://www.swoole-cloud.com/dashboard/catdemo)申请使用后，会看到如下信息

```
demo网址：http://tracker.demo.swoole-cloud.com

账号: demo   密码: admin

申请时间：2021-06-28 21:20:45

服务端地址：http://www.swoole-cloud.com:29666

默认用户名：****** 默认密码：****** 注意：请勿泄露用户名密码

安装脚本：下载

如何安装客户端包，详情点击查看使用文档
```



### 第二步：安装Agent进程

点击客户端包后的**下载**，会得到一个名为`swoole-tracker-install.sh`的脚本，上传到机器后进行如下操作：

```
chmod +x swoole-tracker-install.sh

./swoole-tracker-install.sh
```

#### Docker容器中启动Agent进程

```sh
/opt/swoole/script/php/swoole_php /opt/swoole/node-agent/src/node.php 
```

#### 查看 Agent 进程是否启动

```
ps -ef | grep agent
```



### 第三步：安装扩展

根据你的机器PHP版本安装对应的扩展，复制对应的扩展到PHP环境扩展安装目录

```sh
// 查看扩展所在目录
php -i | grep -i extension_dir
// 复制扩展文件
cp swoole_tracker74.so extension_dir/swoole_tracker.so
// 查看ini文件
php --ini 
// 编辑 php.ini，添加如下配置
extension=swoole_tracker.so
;打开总开关
apm.enable=1
;采样率 例如：100%
apm.sampling_rate=100

;开启内存泄漏检测时添加 默认0 关闭状态
apm.enable_memcheck=1
```

查看扩展信息

```sh
php --ri swoole_tracker
```



#### 卸载不兼容扩展

1. xdebug
2. ioncube loader
3. zend guard loader
4. xhprof
5. swoole_loader （加密后的代码不能进行分析）



### 第四步：重启PHP服务

### 第五步：查看系统信息上报状态

登入服务端地址，点击 Agent 列表，查看系统信息上报状态是否处于开启状态



# 基础功能
