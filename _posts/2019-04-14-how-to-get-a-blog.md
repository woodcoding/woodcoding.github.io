---
title: 如何搭建一个自己的博客(github+jekyll)
date: 2019-04-14 21:14:00
categories: 
 - 环境搭建
tags:
 - blog
 - 博客
 - github
 - jekyll
 - 网站
 - 域名
 - https
---

从博客搭建到现在已经写了有12篇博客了，不(zhen)容(de)易(shao)啊。由于有朋友问起博客怎么搭建的，所以今天就来谈谈博客搭建与平时写博客的工具配置。

【彩蛋】支付宝口令：九九六ICU。哈哈，上周博客忘记更新了  
<!-- more -->

### 简介
搭建博客的方法有很多种，不过说来也是神奇，早期互联网是静态的页面，后来发展到动态，现在大家写博客又好像喜欢回归到静态了。比如像我的这个博客就是一个静态博客，当然功能是比早期的静态页面强大很多的，毕竟出现了这么多的新技术。  
先简单说说我的博客环境，这是一个使用jekyll静态博客框架搭建的博客（介绍以前讲过就不再叙述），代码全部保存在github仓库，并使用github提供的pages服务来实现网站的页面显示，主题采用了移植hexo的next，简洁大方，并且使用了阿里云的https证书服务来为自定义的域名加上了小绿锁。然后平时写博客用的是vscode（和开发一体，微软出品必是精品）。  
为什么选择静态博客？动态博客虽然可以拥有更强大的功能，但是你要维护一系列数据库啊，插件啊，服务器啊之类的麻烦事。至于服务器花钱之类的，钱不钱的不重要，重要的是我只是想写个博客啊，有基本的发布、归档和评论就好了。废话不多说，下面开始进入正题。

### jekyll模板选择
那么多静态博客框架，为什么选择jekyll？毕竟githbu官方扶持啊，具体前面博客介绍了，可以去看看。要想有自己的博客，当然要先选一套自己喜欢的主题（我的博客是基于next配置再微调了一下样式，也欢迎fork），如何找到自己喜欢的主题？这里有几种方法推荐：

1. github
由于静态博客很多都是搭建在github，所以在github上面找准没错了，基本上你网上能看到的模板，github都有开源的。像那些xxx.github.io的网站都是在github搭建的，当然你也可以自定义域名，或者使用国内的coding。直接关键字搜索`jekyll`、`jekyll template`、`github.io`都行，当然，使用github.io可能会搜索到一些hexo或者hugo的静态博客，不要紧，一般都会有移植版本的，耐心找找。

2. 网站
直接百度一下也有很多，或者有几个网站专门列出来了很多漂亮的，可以看看：  
- [http://jekyllthemes.org/](http://jekyllthemes.org/)
- [https://jekyllthemes.io/free](https://jekyllthemes.io/free)
- [http://themes.jekyllrc.org/](http://themes.jekyllrc.org/)

3. 知乎
逼户乎上面的人说话又好听，超喜欢在上面逛的。比如下面这个问题：
- [有哪些简洁明快的 Jekyll 模板？](https://www.zhihu.com/question/20223939/answer/50966881)

### github配置
选好模板之后我们就开始配置github了。

- Step1. 新建仓库  
在github上点击右上角的加号，再点`New repository`新建一个仓库，记得仓库名要这样填：你的用户名.github.io(如：woodcoding.github.io)，这样github会自动给你分配一个二级域名，而且还带小绿锁。
![](/resources/images/2019-04-14/QQ截图20190414225758.png)


- Step2. 上传博客模板
由于上一步我们建立了远程仓库，那就直接把远程仓库clone到本地，改好配置文件之后把博客模板放进去push上去就好了。把你下载的模板解压，找到根目录下应该是有`_config.yml`文件的，这个配置文件基本上是你要好好了解一番的了，网站所有信息都在这里配置，把这个文件夹下所有文件复制到仓库根目录，再找到官方文档把该配置的配置好就行。之后上传到github仓库（至于如何使用github这个请自行参考网上教程）。之后应该是这样的：
![](/resources/images/2019-04-14/QQ截图20190414231253.png)

好了，如果配置没错的话打开你对应的域名看看吧，漂亮的博客是不是就出现了。可以去environment里面查看自动部署情况：
![](/resources/images/2019-04-14/QQ截图20190414234152.png)


- Step3. 配置gitalk评论
由于静态博客本身是不支持评论的，因为要数据库啊。所以这里我们一般使用第三方的评论库。以前还有多说之类的好东西，现在没了，但是还以cy，lbl之类也可以用，不过为什么不试试gitalk呢？这个利用github的issues作为评论的插件库简直绝配，而且简洁大方，使用github账户即可登录评论。在Settings/Developer settings/OAuth Apps里面[https://github.com/settings/applications/new](https://github.com/settings/applications/new)新建一个app，信息如下填写：  
![](/resources/images/2019-04-14/QQ截图20190414233306.png)

新建之后会给你一个id和密钥，然后配置config.yml中相应的配置项：  
```yml
gitalk:
  enable: true
  clientID: ******
  clientSecret: ******
  repo: 用户名.github.io
  owner: 用户名
  admin: 用户名
```

### 自定义域名
- Step1. 解析域名
既然都自定义域名了，那你肯定是要买了域名，不然就那二级域名将就用着吧，直接跳过这一步，也挺好的。解析一条你自己的域名CNAME记录到github，如下图所示：  
![](/resources/images/2019-04-14/QQ截图20190414234606.png)

- Step2. 配置github域名
在github的仓库settings下配置刚解析的域名（Enforce HTTPS选项可以先不选）
![](/resources/images/2019-04-14/QQ截图20190414234827.png)
正常的话过个1分钟你就可以用你自己的域名访问了。

- Step3. 加上https小绿锁
本来一般的https证书都是要钱的，这里我们使用阿里云的免费证书。在产品>安全>SSL证书里面，找到免费的证书购买，然后按照阿里云的提示进行操作认证就行了：  
![](/resources/images/2019-04-14/QQ截图20190414235230.png)
过一会用https访问一下，是不是看见小绿锁了，不过这时候http还是能访问，所以我们去刚刚Step2中那里开启Enforce HTTPS选项，强制跳转https，这样一直都是绿的啦（使用了自定义域名的话上面那个gitalk要用新域名配，不然没办法评论，这是个坑点）。

### 写博客
网站是搭建好了，如何写博客呢？这里我们请出我们最强大的文字流工具markdown。对，我们的博客就用markdown写，写完传到目录下的_posts文件夹下即可。不过这里有个规定，文件名必须以`日期-slug`的格式命名，博客的开头要这样写(不然静态博客识别不出来)：  
```yml
---
title: 如何搭建一个自己的博客(github+jekyll)
date: 2019-04-14 21:14:00
categories: 
 - 环境搭建
tags:
 - blog
 - 博客
 - github
 - jekyll
 - 网站
 - 域名
 - https
---

文章内容

```

这样直接写的话默认所有内容都会显示在首页，加入`<!-- more -->`标签可以分割摘要。规定是有点麻烦，不过习惯了就好了，非常方便强大。  

写博客的话因为是markdown，所以你可以使用任何支撑markdown的工具如马克飞象，印象笔记，有道笔记，everynotes，等等。不过这里推荐使用typora或者vscode（我目前在用）。因为vscode同时还可以做开发，然后推荐一个vscode的markdown插件就是markdown preview enhanced。

### 总结
配置过程是有点繁琐，最好的办法就是fork一个仓库（比如我的），然后改仓库名，改config.yml文件，使用官方的二级域名。直接就可以访问了，懒人必备。
