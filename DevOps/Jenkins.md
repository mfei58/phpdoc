# 前言

## 介绍

## 用途

# 快速入门

## 安装

### Docker 安装

#### Windows

```
docker run ^
-u root ^
-p 9203:8080 ^
-v jenkins-data:/var/jenkins_home ^
-v /var/run/docker.sock:/var/run/docker.sock ^
-v "%HOMEPATH%":/home ^
-d
jenkinsci/blueocean:1.24.3
```



#### Linux

```
docker run \
-u root \
-p 9203:8080 \
-v $(pwd)/jenkins/jenkins-data:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
--name jenkins \
-d \
jenkinsci/blueocean:1.24.3
```



# 基础功能

## 基础配置

### 系统管理-系统配置-系统配置

#### 设置主页URL

`Jenkins Location` 设置`Jenkins URL`

### 系统管理-安全-全局安全配置

#### 开启代理

`Agent → Controller Security`  开启 `Agent`

### 系统管理-工具和动作-脚本命令行

#### 设置时区

```groovy
System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')
```

## Pipline

### `ssh`登入授权

```
## 进入Jenkins Docker容器
docker exec -it jenkins /bin/bash
ssh-keygen
cat ~/.ssh/id_rsa.pub
## 进入Jenkins加入的主机A，将Jenkins用户公钥添加到A主机用户的authorized_keys
echo COPY_JENKINS_ROOT_ISA_PUB>>~/.ssh/authorized_keys
```

### `Pipline` 登入测试

```
sh '''
ssh huasheng@192.168.10.218
'''
```



## 集成Gitlab

### 安装Gitlab插件

###### 进入>系统管理-系统配置-插件管理

下载`Gitlab`插件

### 系统配置Gitlab属性

###### 进入>系统管理-系统配置-系统配置-Gitlab

```
Gitlab-Enable authentication for '/project' end-point
Connection name：连接名
Gitlab host URL：Gitlab地址
```

### Pipline配置Gitlab触发器

###### 进入>Pipline-配置-构建触发器

选择`Build when a change is pushed to GitLab`

选择`Allow all branches to trigger this job`

点击`Generate`生成 `Secret token`

### 配置 Gitlab Webhooks

###### 进入>Gitlab项目-设置-Webhooks

添加`URL`和`Secret Token`，`Push Event` 分支



## 集成钉钉

### 通过Curl触发通知

```groovy
sh '''
     curl 'https://oapi.dingtalk.com/robot/send?access_token=30656449ca040d08ee1f3bc93c0ec0aac95dd694543308ec23441a08150e50dd' \
-H 'Content-Type: application/json' \
-d '{"msgtype": "markdown","markdown": { "title": "项目名称","text":"<font color=#3399ff size=4>项目名称</font> \n\n--- \n - 任务：任务名称 \n - 状态：任务状态"},"at":{"isAtAll": true}}'
        '''
```







