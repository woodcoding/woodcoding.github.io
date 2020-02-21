---
title: Flask锚点多表单实现与HTTP300重定向系列状态码
date: 2020-02-21 22:30:00
categories: 
 - Flask
tags:
 - Flask
 - HTTP
 - 状态码
 - 锚点
---

### 摘要

&ensp;&ensp;&ensp;&ensp;今天遇到了一个Flask重定向的问题，在此分享一下。问题情景：在一个前后端不分离的页面下，用Tab标签做了多个页面切换，每个页面有自己的表单数据，页面是统一一个地方加载的，假设为Profile视图，但是处理表单数据分了多个视图，如change-password，change-email。那么问题来了，如何实现正确的加载页面与数据处理操作呢？虽然这个问题很傻逼，可能一时还理不清这其中有啥问题，并且在前后端分离的情况下这根本不是一个问题。但是今天本着“作死”到底的精神，我们可以来详细分析一下。  

<!-- more -->

### 问题分析
如果上面的文字分析还不能理解清楚，那我们先来上一张图看看。  

![](/resources/images/2020-02-19/profile.png)

如上图所示，这是我们的一个多标签页的页面，这是用bootstrap生成的页面，多个标签之间的切换采用锚点来实现。我们在profile视图中渲染了这个页面，代码如下：  

```python
@user_bp.route('/profile/', methods=['GET', 'POST'])
@login_required
def profile():
    context = dict()
    ...
    return render_template('user/profile.html', **context)
```

用户可以在这个页面自由切换“个人信息”、“修改密码”等不同标签页。但是现在的问题是，我们不可能把这个标签页中所有的表单处理逻辑全放入profile函数之中，所以我们要抽离不同页面的表单处理逻辑，代码如下：  

```python
@user_bp.route('/profile/change-password/', methods=['GET', 'POST'])
@fresh_login_required
def change_password():
    pass

@user_bp.route('/profile/change-info/', methods=['GET', 'POST'])
@login_required
def change_password():
    pass
```

这里处理之后的逻辑就是，用户在profile视图加载所有的操作页面，对应不同的表单处理可以通过前端action传递到不同的视图，如change-password，然后由change-password处理完成之后再返回操作前的标签页面，如果表单的值验证不符则显示对应的提示。  

所以这里就有两个问题需要处理：
1、如何在处理完成后跳转到对应的标签视图？
2、重定向之后如何显示对应的标签页表单验证的错误提示？

### 问题解决

问题找到了，如何解决？

- 问题1：如何在处理完成后跳转到对应的标签视图？

首先，我们处理完成之后肯定是要进行跳转的，所以代码可以这样写：  

```python
redirect(url_for('.profile'))
```

直接跳转到profile视图，这里会存在一个问题，如果直接跳转，只会显示第一个标签页，想要显示对应的标签页还需要加上一个参数，如下，加入锚点值（锚点值其实应该是前端应该关注的内容，是前端用来定位元素的）：  

```python
redirect(url_for('.profile', _anchor='change-password'))
```

- 问题2：重定向之后如何显示对应的标签页表单验证的错误提示？
  
问题一很好解决，查文档之后就知道添加anchor参数，但是随即问题又来了。你会发现，如果表单验证不通过，他就是直接跳转到页面，不知道的还以为自己只是刷新了一下页面，用户体验极差。于是就debug了一下，发现重定向的时候数据完全没带过去，所以profile视图的表单也验证不了数据。经过一番面向搜索引擎编程之后，发现原来flask的redirect默认的http状态码是302，这里我们也可以从阅读flask源码中了解，在flask依赖的扩展werkzeug的utils模块中，定义了redirect函数如下：  

```python
def redirect(location, code=302, Response=None):
    """Returns a response object (a WSGI application) that, if called,
    redirects the client to the target location. Supported codes are
    301, 302, 303, 305, 307, and 308. 300 is not supported because
    it's not a real redirect and 304 because it's the answer for...
```

可以看到code确实默认是302的。不过问题又来了，重定向和这个302之间有啥关系呢？这里就牵涉到了一些HTTP协议相关的知识，有兴趣的可以去读一读图灵系列图书《图解HTTP》，对web有点了解的画个几天的零碎功夫就看完了。我们知道HTTP状态码有很多种，常见的有成功响应的200系列，重定向的300系列，请求出错的400系列，服务器内部错误的500系列，在浏览器的开发者控制台也能看到请求一个网页的状态码：  

![](./resouces/../../resources/images/2020-02-19/chrome_status_code.png)

我们返回的重定向函数并不是在服务器端的视图直接重定向，而是下发数据到浏览器，再由浏览器通过状态码来选择进行何种重定向操作。下面是我面向搜索引擎编程搞来的一些常见的重定向的解释：

> 300：Multiple Choice(多种选择)
该请求有多种可能的响应，用户代理或者用户必须选择它们其中的一个。服务器没有任何标准可以遵循去代替用户来进行选择。HTTP/1.0 以及以前的版本。

> 301：Moved Permanently(永久移动)
该状态码表示所请求的URI资源路径已经改变，新的URL会在响应的Location:头字段里找到。 HTTP/0.9 可用。

> 302：Found(临时移动)
该状态码表示所请求的URI资源路径临时改变，并且还可能继续改变。因此客户端在以后访问时还得继续使用该URI.新的URL会在响应的Location:头字段里找到。HTTP/0.9 可用。

> 303：See Other(查看其他位置)
服务器发送该响应用来引导客户端使用GET方法访问另外一个URL。 HTTP/0.9 和 1.1可用。

> 304：Not Modified(未修改)
告诉客户端，所请求的内容距离上次访问并没有变化。 客户端可以直接从浏览器缓存里获取该资源。HTTP/0.9 可用。

> 305：Use Proxy(使用代理)
所请求的资源必须统过代理才能访问到。由于安全原因，该状态码并未受到广泛支持。HTTP/1.1 可用。

> 306：unused(未使用)
这个状态码已经不再被使用，当初它被用在HTTP 1.1规范的旧版本中。HTTP/1.1 可用。

> 307：Temporary Redirect(临时重定向)
服务器发送该响应，用来引导客户端使用相同的方法访问另外一个URI来获取想要获取的资源。新的URL会在响应的Location:头字段里找到。与302状态码有相同的语义，且前后两次访问必须使用相同的方法(GET POST)。HTTP/1.1 可用。

> 308：Permanent Redirect(永久重定向)
所请求的资源将永久的位于另外一个URI上。新的URL会在响应的Location:头字段里找到。与301状态码有相同的语义，且前后两次访问必须使用相同的方法(GET POST)。HTTPbis(试验草案)

而在大部分浏览器用的302会将POST转为GET再重定向，这里我们使用307状态码就可以将post的数据直接传给重定向之后的视图了。至于你说为什么307能做到这样？大家规定就个状态码就是这样的操作，我也没办法。  