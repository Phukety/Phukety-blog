---
title: Docker的下载与安装
date: 2020-09-24 20:48:46
cover: images/docker.jpg
photos: 
  -  images/docker.jpg
categories: 
  - 容器技术
tags: 
  - [docker]
---
## 本文主要介绍docker的下载与安装

<!--more-->

1. #### 若存在旧版本，则先卸载（没有则跳过这步）

```
sudo apt-get remove docker docker-engine docker-ce docker.io docker-ce-cli
```
2.	#### 更新apt包

```
sudo apt-get update
```
3.	#### 安装以下包使apt可以通过HTTPS使用存储库

```
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
```

4.	#### 添加Docker官方GPS密钥

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

5.	#### 设置stable存储库

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

6.	#### 再次更新apt索引

```
sudo apt-get update
```

7.	#### 安装最新的Docker CE

```
sudo apt-get install -y docker-ce
```

8.	#### 安装特定版本Docker CE

```
apt-cache madison docker-ce
```

##### 　列出可用的版本

![](p1.jpg)

- 第二列：版本字符串
- 第三列：存储库名称

##### 　选择指定版本

```
sudo apt-get install docker-ce=<VERSION>
```

9.	#### 验证是否安装成功

##### 　查看docker版本

```
docker -v
```
![](p2.jpg)
##### 　则docker安装成功

10.	#### 启动docker的hello world容器

```
docker run hello-world
```
![](p3.jpg)
##### 　提示以上错误，是因为docker安装后，**Docker的守护进程(Docker daemon)** 会监听 **Unix域套接字/var/run/docker.sock**，容器中的进程可以通过它与Docker daemon进行通信，但由于/var/run/docker.sock的用户组是docker，所以提示无权限，因此解决方法是将登录用户加入docker用户组或者直接使用sudo
- ###### 创建docker用户组（安装docker后会自动创建该用户组）
	```
sudo groupadd docker
	```
- ###### 添加用户至docker用户组
	```
sudo gpasswd -a yuanbw docker
	```

##### 　若出现，则启动成功

![](p4.jpg)

```

```