---
title: Django中配置redis缓存与celery异步队列
date: 2019-04-07 11:04:00
categories: 
 - 开发工具
tags:
 - django
 - redis
 - celery
 - 缓存
 - cache
 - 异步
 - flower
---
由于毕设项目的需要，引进了redis缓存与celery异步队列的支持，但是配置过程似乎不是那么一帆风顺，所以在此记录下配置过程。
<!-- more -->
### 简介
#### Redis
Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。它支持多种类型的数据结构，有序集合与范围查询，地理空间，索引半径查询。 Redis 内置了复制，LUA脚本， LRU驱动事件，事务和不同级别的磁盘持久化，并通过Redis哨兵和自动分区提供高可用性。  
使用场景有：
- 会话缓存
- 消息队列
- 活动排行榜或计数
- 发布/订阅消息

Redis一共支持五种数据结构：
- string（字符串）
- hash（哈希）
- list（列表）
- set（集合）
- zset（sorted set有序集合）
#### Celery
Celery 是一个简单、灵活且可靠的，处理大量消息的分布式系统，并且提供维护这样一个系统的必需工具。它是一个专注于实时处理的任务队列，同时也支持任务调度。其实异步工具很多，celery也算不上是行业标准，但是在python web里面用作异步队列也算是有一席之地了。

### 配置
#### Redis
##### 下载
奈何开发平台是Win10，Redis官方本来都没有打算支持win平台的，但是我们的微软作为最佳开发者给他搞了一个分支，地址在：https://github.com/MicrosoftArchive/redis。 问题又来了，微软大刀部可能觉得这个项目会太受欢迎就把它给砍了，目前最后的版本还停留在3.2.100，但是redis官方linux版本都已经飙到5.x了，虽然远程服务器上也有部署redis，可是不想连网络啊，所以开发机测试将就用一下3.2.100，反正都是基本功能，目前没什么差的，去release页面下载最新的zip到本地解压就可以直接跑了。
```bash
D:\Program Files\Redis-x64-3.2.100\redis-server.exe
```

##### 配置django缓存
这里的话使用redis主要是三个用途，一个是django自带的缓存中间件，一个是celery的中间人，还有就是以后业务逻辑里面的分析队列和爬虫任务要用。先看看配置在setttings.py里面配置django缓存：
```python
# Redis Client
REDIS_HOST = 'localhost'
REDIS_PORT = 6379
REDIS_DB = 0
REDIS_URI = "redis://%s:%s/%s" % (REDIS_HOST, REDIS_PORT, REDIS_DB)

CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": REDIS_URI,
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
```
由于django本身并不支持redis作为缓存中间件，所以我们使用了一个`django-redis`的第三方包来协助做中间件，记得pip install一下哦。

#### RedisClient
对于我这种渣渣来说，命令行虽然实用，但是想掌握全局状况的话，有个可视化平台当然是最好的，选来选取看到一个小前端打包的客户端，优点漂亮，还支持中文，就先用上了。地址在：https://github.com/qishibo/AnotherRedisDesktopManager， 开箱即用，来欣赏一下。
![](/resources/images/2019-04-07/QQ截图20190414205326.png)

#### Celery
##### 下载
目前celery的最新版本是4.2，所以我们直接使用这个版本，如果要看中文文档以及使用django-celery的话，3.x版本才是适合你的。由于celery是一个python包，所以直接pip install就可以安装最新的celery版本了。

##### 配置
由于不想维护太多工具，这里我们同样使用了redis做celery的支架工具。先在settings里面配置一下：
```python
CELERY_BROKER_URL = 'redis://localhost:6379/1'  # 由于缓存用了0数据库，这里就别挤在一块了，用1数据库算了，以后爬虫分析任务就用2数据库。
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_BACKEND = 'django-db'
CELERY_TIMEZONE = TIME_ZONE
CELERYD_CONCURRENCY = 4
```

然后在`settings.py` 文件同级目录下新建一个`celery.py`，配置如下：

```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'AcmAnalyse.settings.dev')

app = Celery('AcmAnalyse')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

##### 运行
启动celery：
```bash
celery -A AcmAnalyse worker --pool=solo -l info
```

#### Flower
flower是celery推荐的任务监控工具，可以监控celery任务进程和历史，显示任务的详细信息（arguments, start time, runtime等），图形化和统计。这里我们也是同样适用pip安装即可。虽然比较丑，但是将就也能用一下，执行以下命令，flower会自动发现并监控celery：
```bash
celery -A AcmAnalyse flower --pool=solo -l info
```
运行后会提示打开 http://localhost:5555 查看监控，截图如下：

![](/resources/images/2019-04-07/QQ截图20190414211157.png)

### 总结
这些缓存和异步队列工具还是强大啊，可惜还不是很懂其中的原理，以后有时间要多看看，了解一下，要是自己也能写出这么牛逼的工具就(zuo)好(meng)了(ba)。
