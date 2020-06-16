---
title: CTF入门练习-HTTP基础
date: 2020-06-16 15:00:00
categories: 
 - CTF
tags:
 - CTF
 - 安全
 - 入门
 - 基础
 - HTTP
 - Web

---

### 简介
&ensp;&ensp;&ensp;&ensp;在CTF上面有个技能树，先把里面的基础练习做一遍。感觉CTF对知识面的广度要求非常高，这是个很好的学习机会。今天是HTTP协议相关的题目。<!-- more -->

### 题目列表

#### 1.请求方式

- 题目简介

HTTP 请求方法, HTTP/1.1协议中共定义了八种方法（也叫动作）来以不同方式操作指定的资源。

- 解题思路
  
打开题目给出的网址过后，显示如下内容的网页：

>HTTP Method is GET
Use CTF**B Method, I will give you flag.
Hint: If you got 「HTTP Method Not Allowed」 Error, you should request index.php.

显然，题目是想我们通过不同的HTTP方法请求来获取flag。但是刚开始理解题目可能有点难度，毕竟`CTF**B`他是想表达个啥嘛。我还以为是举个例子，然后用HTTP1.1的9种(题目写的8种，漏了patch)方法去请求，结果都没有看到flag。最后想想，有没有可能是除了这八大方法之外，我也可以使用自己定义的方法呢？于是搜索pyhton库里面使用自定义方法的教程，然后发起一个`CTFHUB`方式的请求，flag就出来了。


- 相关知识
  
  HTTP1.1请求定义的九种方法：

| 序号 | 方法  | 介绍                                                                                                                                   |
| ---- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | GET     | 请求指定的页面信息，并返回实体主体。                                                                                   |
| 2    | HEAD    | 类似于 GET 请求，只不过返回的响应中没有具体的内容，用于获取报头                                             |
| 3    | POST    | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立和/或已有资源的修改。 |
| 4    | PUT     | 从客户端向服务器传送的数据取代指定的文档的内容。                                                                 |
| 5    | DELETE  | 请求服务器删除指定的页面。                                                                                                  |
| 6    | CONNECT | HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。                                                        |
| 7    | OPTIONS | 允许客户端查看服务器的性能。                                                                                               |
| 8    | TRACE   | 回显服务器收到的请求，主要用于测试或诊断。                                                                          |
| 9    | PATCH   | 是对 PUT 方法的补充，用来对已知资源进行局部更新 。                                                                 |
    
**注意：** 特定的HTTP服务器支持自定义的HTTP请求方法。


- 题解代码
  
```python
from urllib import request

req = request.Request(method='CTFHUB', url='http://challenge-7c8a21f5ac0bee47.sandbox.ctfhub.com:10080/index.php')
res = request.urlopen(req)
print(res.read())

```


#### 2. 302跳转

- 题目简介
  
   HTTP临时重定向

- 解题思路
  
  打开题目给出的网址后，显示如下内容：
  
  ![](/resources/images/2020-06-16/2020-06-16-17-48-06.png)

由于是重定向，所以打开浏览器控制台我们可以发现，前往index.php的请求被临时重定向到了index.html，所以就想到，flag可能在index.php中，但是浏览器会自动跳转所以我们根本看不到内容，只能写代码阻止跳转，然后直接查看内容。

- 相关知识
  
  重定向的知识我之前有写一篇博客，这里就不多BB了。

- 题解代码

```python
import requests

url = 'http://challenge-df0e9edbdeb14ca4.sandbox.ctfhub.com:10080/index.php'
resp = requests.get(url, allow_redirects=False)
print(resp.text)
```


#### 3. Cookie

- 题目简介
  
   Cookie欺骗、认证、伪造

- 题解思路
  
  打开题目给的网址后，显示如下内容。
  
  > hello guest. only admin can get flag.

  既然是cookie，那肯定是cookie里面设置了权限。打开浏览器控制台如下图，果然看到admin有个值为0，改成1，ok，看到flag。

    ![](/resources/images/2020-06-16/2020-06-16-15-21-17.png)

- 相关知识
  
  cookie就是在浏览器本地存储一些用户相关的信息，用于动态网页的显示，早期互联网发展的时候还有在cookie存用户敏感信息的，但是这非常不安全，容易被窃取用于其他目的，因此在web开发中我们一定不能往cookie中存敏感信息，不然就容易像这个题目一样明明管理员才能看的就被我们看到了。


- 题解代码
  
  这题我直接改浏览器的，所以没写代码，但是要写也可以，传个cookie参数，把admin设为1就行。


#### 4. 基础认证

- 题目简介
  
   在HTTP中，基本认证（英语：Basic access authentication）是允许http用户代理（如：网页浏览器）在请求时，提供 用户名 和 密码 的一种方式。详情请查看 https://zh.wikipedia.org/wiki/HTTP基本认证

- 解题思路
  
  这种认证之前在路由器管理界面、nginx代理认证这些上面都有见过，对于单用户管理系统来说可以简化用户系统模块，是很方便的。打开题目给的网址，显示如下：

    ![](/resources/images/2020-06-16/2020-06-16-17-47-35.png)

    由于题目给出了100个密码，但是没有用户名，所以根据wiki给的知识，我们可以先往目标链接发一个请求，发现Authorization头里面有一句话：`Do you know admin?`(大概是这样的，忘记记录了)。所以admin肯定是用户名，但是问题又来了，这100个密码难道手动输入？不可能！直接上代码即可。

- 题解代码

    ```python
    def http_basic_auth():
        auth_url = 'http://challenge-49903ae2a973e552.sandbox.ctfhub.com:10080/flag.html'
        with open('10_million_password_list_top_100.txt', 'r', encoding='utf-8') as f:
            for pwd in f.readlines():
                resp = requests.get(auth_url, auth=('admin', pwd.strip()))
                if resp.status_code != 401:
                    print(resp.text)
    ```

#### 5. 响应包源代码

- 题目简介
  
  HTTP响应包源代码查看

- 解题思路
  
  这题太简单了，直接右键查看源代码，flag就在注释里面。

  ```HTML
  <!-- ctfhub{efff37a833235e5ff56b5e648276eb7d5b2d****} -->
  ```

- 相关知识
  
  HTML页面的源码是可以直接在浏览器右键查看的，但是这源码并不是我们后台系统的真正源码，是动态生成后的显示效果源码。By the way，现在国内B/S系统这么流行，不会还有人不知道怎么看页面源码吧？

- 题解代码
  这题也是不需要代码的，直接右键多方便，要写代码的话，直接打印目标链接的内容就行了（这也太麻烦了）。




### 参考资料

- https://www.runoob.com/http/http-methods.html
- https://blog.woodcoding.com/flask/2020/02/21/http-300-code-with-flask-form-anchor/
