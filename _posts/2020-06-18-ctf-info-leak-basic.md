---
title: CTF入门练习-信息泄露
date: 2020-06-18 15:00:00
categories: 
 - CTF
tags:
 - CTF
 - 安全
 - 入门
 - 基础
 - 信息泄露

---

### 简介
&ensp;&ensp;&ensp;&ensp;今天技能树加点的是信息泄露方面的内容，主要有目录遍历、PHPINFO、备份文件下载、Git泄露、SVN泄露、HG泄露等几个方面的内容。<!-- more -->


### 目录遍历

- 题目简介
  
  题目给出一个网址，打开之后可以浏览服务器提供的目录

- 解题思路
  
  这题简单，就是去目录下找到flag文件即可。我看还有通过nginx配置漏洞去找系统级目录的，那个应该难一些。

- 相关知识
  
  主要是nginx、apache配置目录浏览问题。有时候公司内部可能为了方便提供文件下载使用这种方式，但是配置不当可能会被利用。

- 题解代码
  
  无

### PHPINFO

- 题目简介
  
  题目给出一个网址，打开之后可以查看服务器php版本的信息

- 解题思路
  
  这题也简单，直接ctrl+f找页面中的flag就成。

- 相关知识
  
  php探针

- 题解代码
  
  无

### 备份文件下载

#### bak文件

- 题目简介
  
  当开发人员在线上环境中对源代码进行了备份操作，并且将备份文件放在了 web 目录下，就会引起网站源码泄露。常见的就是bak结尾的文件。

- 解题思路
  
  枚举一些常见的备份文件名即可。

- 相关知识
  
  类似php、asp这种网站可以通过虚拟主机控制面板直接线上备份源码，备份在网站目录下的可能会被访问到，因此备份尽量放在其他目录。

- 题解代码
  
  ```python
  import os
  import requests

  suffix_list = ['tar', 'tar.gz', 'zip', 'rar']

  filename_list = ['web', 'website', 'backup', 'back', 'www', 'wwwroot', 'temp']

  url = 'http://challenge-b2bb6e0f371b7f16.sandbox.ctfhub.com:10080/'

  for filename in filename_list:
      for suffix in suffix_list:
          full_filename = '{name}.{suffix}'.format(name=filename, suffix=suffix)
          file_url = os.path.join(url, full_filename)
          resp = requests.get(file_url)
          if resp.status_code != 404:
              print(file_url)
  ```

#### vim缓存

- 题目简介
  
  当开发人员在线上环境中使用 vim 编辑器，在使用过程中会留下 vim 编辑器缓存，当vim异常退出时，缓存会一直留在服务器上，引起网站源码泄露。

- 解题思路
  
  vim在正常退出时会在当前目录保留一份.filename.swp的文件(如果在恢复文件中再次非正常退出又会生成.swo文件...)。获取到swp文件后直接用`vim -r .filename.swp`就能看到文件详情了。

- 相关知识
  
  vim缓存文件相关知识，最好不要在生产服务器使用vim进行修改。

- 题解代码
  
  无

#### .DS_store

- 题目简介
  
  .DS_Store 是 Mac OS 保存文件夹的自定义属性的隐藏文件。通过.DS_Store可以知道这个目录里面所有文件的清单。

- 解题思路
  
  获得.DS_Store文件后可以使用工具可以从文件中读取到文件夹目录下的文件，在网站上访问flag文件即可获得。

- 相关知识
  
  .DS_Store是Mac中保存文件夹信息的文件。拷贝文件时千万不要拷贝这个文件。

- 题解代码
  
  ```shell
  # https://github.com/gehaxelt/Python-dsstore
  python main.py samples/.DS_Store.ctf
  ```

### Git泄露

- 题目简介
  
  当前大量开发人员使用git进行版本控制，对站点自动部署。如果配置不当,可能会将.git文件夹直接部署到线上环境。这就引起了git泄露漏洞。

- 解题思路
  
  可以通过工具下载网站上的.git目录，本地再进行信息探索即可。不过要注意不要过于依赖某个工具，因为我之前通过[GitHacker](https://github.com/WangYihang/GitHacker)查到了通过log泄露的flag，但是后面的stash和index却提示错误，似乎并没有正确重建，所以拿不到数据。后来换了另外一个[Git_Extract](https://github.com/gakki429/Git_Extract)直接提取文件恢复拿到了stash和index泄露的flag。

- 相关知识
  
  .git文件夹是Git版本控制所用到的。所以即使我们删除了文件如果不删除.git文件夹，我们依然可以恢复到之前某个版本，获得相应的文件。因此，切记不可把.git文件夹放到网站服务器上。

- 题解代码
  
  ```shell
  # https://github.com/WangYihang/GitHacker
  # https://github.com/gakki429/Git_Extract
  # 使用对应工具进行操作。
  ```


### SVN泄露

- 题目简介
  
  当开发人员使用 SVN 进行版本控制，对站点自动部署。如果配置不当,可能会将.svn文件夹直接部署到线上环境。这就引起了 SVN 泄露漏洞。

- 解题思路
  
  使用`dvcs-ripper`工具箱下面的rip-svn.pl(注意在自己系统上使用的话可能要安装svn软件等，可能会存在一些版本依赖问题，很难装上，可以使用封装的docker进行操作会比较方便)。svn中wc.db记录一些版本相关信息，在wc.db中可以看见flag文件，但是访问却显示不存在，继而转`.svn/pristine/`目录直接搜索对象文件即可找到flag。

- 相关知识
  
  和Git差不多，SVN版本控制使用了.svn文件夹。dvcs-ripper是一款perl的版本控制软件信息泄露利用工具，支持SVN, GIT, Mercurial/hg等。

- 题解代码
  
  ```shell
  # https://github.com/kost/dvcs-ripper
  # 使用对应工具进行操作。
  ```

### HG泄露

- 题目简介
  
  当开发人员使用 Mercurial 进行版本控制，对站点自动部署。如果配置不当,可能会将.hg 文件夹直接部署到线上环境。这就引起了 hg 泄露漏洞。

- 解题思路
  
  使用`dvcs-ripper`工具箱下面的rip-hg.pl。clone的仓库可能并不完整，所以可以去`.hg/store/fncache`目录找文件列表，找到可以直接访问获取。如果文件被删除，则从历史记录`.hg/store/data/`中寻找。

- 相关知识
  
  Mercurial 是一种轻量级分布式版本控制系统，采用 Python 语言实现，易于学习和使用，扩展性强。需要了解其目录结构。

- 题解代码
  
  ```shell
  # https://github.com/kost/dvcs-ripper
  # 使用对应工具进行操作。
  ```