---
title: Ubuntu server 16.04 下的环境配置
date: 2019-02-03 20:15:00
categories: 
 - linux
tags:
 - ubuntu
 - server
 - 配置
 - docker
---

&ensp;&ensp;&ensp;&ensp;由于毕设的需要、学习的需要等待原因需要配置一台云服务器。好久没搞了，这次就把整个过程记录下来下次好用。只是初步配置了以下，以后慢慢配，慢慢更新。

#### 系统更新
安装完系统之后首先要进行必要的系统更新。
```bash
$ sudo apt update
$ sudo apt upgrade
```

#### 修改root密码
密码还是要保管好。
```bash
sudo passwd root
```

#### 安装docker
docker是一个方便的虚拟化部署运行方案。
```bash
# 安装必要工具
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# 添加key
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 验证key
$ sudo apt-key fingerprint 0EBFCD88

# 添加源
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# 更新安装源
$ sudo apt-get update

# 安装docker
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

# 测试运行
$ sudo docker run hello-world

# 增加普通用户执行docker权限
$ sudo groupadd docker
$ sudo gpasswd -a ${USER} docker
$ sudo service docker restart或sudo systemctl restart docker
```
#### 安装pip
python的基本包安装工具，下面的docker-compose也依赖pip来安装
```bash
# 基本组件（py3的话就把python改成python3）
$ sudo apt-get install python-pip python-dev build-essential 
$ sudo pip install --upgrade pip 
$ sudo pip install --upgrade virtualenv

# 中途把pip升级搞坏了，这里提供几个解决办法

# ImportError: No module named _internal
# 强制重装
$ wget https://bootstrap.pypa.io/get-pip.py  --no-check-certificate
 
$ sudo python3 get-pip.py --force-reinstall

# 强制升级
$ sudo -H python -m pip install --upgrade pip
```

#### 安装docker-compose
Compose 通过一个配置文件来管理多个Docker容器
```bash
$ pip install docker-compose -i https://mirrors.aliyun.com/pypi/simple/
```

#### 安装swarmpit(废弃，改用rancher)
docker集群一个简单的图形化管理web ui，就用来看看系统运行状态
```bash
$ git clone https://github.com/swarmpit/swarmpit -b 1.5.1
$ docker stack deploy -c swarmpit/docker-compose.yml swarmpit
# localhost:888
# admin/admin
# 记得放通端口
```

#### 安装Rancher
同上，只不过上面的有点垃圾，这个比较好，还有中文。
```bash
$ sudo docker run -d --restart=unless-stopped -p 8080:8080 rancher/server:stable
# Tail the logs to show Rancher
$ sudo docker logs -f <CONTAINER_ID>

# 之后进入web可以设置中文，官方也有中文文档就不详说了，简单易用。记得使用github做用户授权。
```

#### docker部署mysql
使用docker来部署mysql，注意数据持久化
```bash
$ docker pull mysql
$ mkdir mysql_data
$ docker run -d --name mysql \  
-v /home/ubuntu/mysql_data:/var/lib/mysql \  
-p 3306:3306 \  
-e MYSQL_ROOT_PASSWORD=my-secret-pw \ mysql:latest
# -d 后台运行模式
# -v 目录映射
# -p 端口映射
# -e 环境变量
# 对所有的 A:B，A表示宿主机项，B表示容器项

# 设置可外网连接mysql（可选）
$ docker exec -it huas-mysql /bin/bash
$ mysql -u root -p
mysql>alter user 'root'@'%' identified with mysql_native_password by '123456';
# 或者
update user set host = ‘%’ where user = ‘root’;
# 前一条顺便改了密码，后一条不改
```

#### docker部署yapi

Yapi 由 YMFE 开源，旨在为开发、产品、测试人员提供更优雅的接口管理服务，可以帮助开发者轻松创建、发布、维护 API。

```bash
# 1、创建 MongoDB 数据卷
$ docker volume create mongo_data_yapi

# 2、启动 MongoDB
$ docker run -d --name mongo-yapi -v mongo_data_yapi:/data/db mongo

# 3、获取 Yapi 镜像，版本信息可在 阿里云镜像仓库 查看
$ docker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi

# 4、初始化 Yapi 数据库索引及管理员账号
$ docker run -it --rm \
  --link mongo-yapi:mongo \
  --entrypoint npm \
  --workdir /api/vendors \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  run install-server

# 自定义配置文件挂载到目录 /api/config.json

# 5、启动 Yapi 服务
$ docker run -d \
  --name yapi \
  --link mongo-yapi:mongo \
  --workdir /api/vendors \
  -p 3000:3000 \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  server/app.js

使用 Yapi
访问 http://localhost:3000   登录账号 admin@admin.com，密码 ymfe.org
```

#### docker部署redis
redis缓存数据库
```bash
$ docker pull redis
$ docker run -v /home/ubuntu/redis_data:/data -p 6379:6379 --name redis -d redis redis-server --appendonly yes --requirepass "mypassword"
```

#### 安装nodejs环境
Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。js利器。
```bash
# 安装低版本node
$ sudo apt-get install nodejs
$ sudo apt install nodejs-legacy
$ sudo apt install npm

# 更新淘宝镜像
$ sudo npm config set registry https://registry.npm.taobao.org

# 查看配置是否生效
$ sudo npm config list

# 安装版本更新工具N
$ sudo npm install n -g

# 更新nodejs到最新版本
$ sudo n stable
```

#### 启用/关闭虚拟内存
内存有点不够用，怕崩。
```bash
# 查看内存情况使用情况
$ sudo free -mh
# 查看当前系统是否存在swap
$ sudo swapon -s
# 新建4G虚拟内存
$ sudo fallocate -l 4G /swapfile
# 设置root读写权限
$ sudo chmod 600 /swapfile
# 通知系统挂载
$ sudo mkswap /swapfile
# 通知系统启用swap
$ sudo swapon /swapfile
# 开机自动加载
$ sudo vi /etc/fstab
# 打开文件后，最后面添加一行
/swapfile  none  swap  sw  0  0
# wq保存

# 关闭虚拟内存
$ sudo swapoff /swapfile
$ sudo rm /swapfile
```
