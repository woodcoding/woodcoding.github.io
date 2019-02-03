---
title: Ubuntu server 16.04 下的环境配置
date: 2019-02-03 21:55:00
categories: 
 - linux
tags:
 - ubuntu
 - server
 - 配置
 - docker
 - nginx
---

&ensp;&ensp;&ensp;&ensp;nginx作为一个极其重要的代理工具，我们就单独拎出来讨论讨论。
<!-- more -->

#### 安装nginx
```bash
$ sudo apt-get install nginx
```

#### 防火墙
个人电脑上需要搞这个，云服务器上有安全组可以弄
```bash
$ sudo ufw app list
# Nginx Full：此配置文件打开端口80（正常，未加密的Web流量）和端口443（TLS / SSL加密流量）
# Nginx HTTP：此配置文件仅打开端口80（正常，未加密的Web流量）
# Nginx HTTPS：此配置文件仅打开端口443（TLS / SSL加密流量）
$ sudo ufw allow 'Nginx Full'
$ sudo ufw status
```

#### 相关命令
基本操作的命令还是要了解一下的
````bash
# 启动服务器
$ sudo systemctl start nginx
# 停止服务器
$ sudo systemctl stop nginx  
# 重启服务器
$ sudo systemctl restart nginx
# 重载服务器配置
$ sudo systemctl reload nginx
# 查看服务器状态
$ sudo systemctl status nginx
# 开启开机启动
$ sudo systemctl enable nginx
# 关闭开机启动
$ sudo systemctl disable nginx
````

#### 相关目录文件
- /var/www/html：Web内容存放位置
- /etc/nginx：Nginx配置目录，相关文件基本都在这里。
- /etc/nginx/nginx.conf Nginx主配置文件。
- /etc/nginx/sites-available：存放每个站点的配置文件，需要链接到sites-enabled目录才会起作用。
- /etc/nginx/sites-enabled/：通过命令链接到sites-available目录可使其生效。
- /etc/nginx/snippets：Nginx配置片段，暂时不会用到。
- /var/log/nginx/access.log：默认配置的请求日志文件。
- /var/log/nginx/error.log：发生错误的请求记录在这里。

#### 软链接
使上述网站配置文件生效
```bash
sudo ln -s /etc/nginx/sites-available/my.conf /etc/nginx/sites-enabled/
```

#### 配置文件
在sites-available下建立conf文件，链接到sites-enabled下，然后重载配置即可使配置生效，命令都在上面了，nginx对文件语法比较严格，使用上面的status命令可以查看失败原因。
```bash
server {
    # 监听IPV4的80端口
	listen 80;
    # 监听IPV6的80端口
	listen [::]:80;
    # 域名
	server_name test.woodcoding.com;

	location / {
        # 几个请求头
		proxy_set_header  Host  $http_host;
		proxy_set_header  X-Real-IP  $remote_addr;
		proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
		# 代理地址
        proxy_pass http://127.0.0.1:3000;
        # 下面这行千万别加，加了静态文件就失效了
	    # try_files $uri $uri/ =404;
	}
}
# 更详细的用法请使用搜索引擎或者http://www.nginx.cn/doc/
```
暂时用到这些，以后再填坑。弄了nginx之后可以把云上不必要的端口都给封了，全让nginx代理即可。
