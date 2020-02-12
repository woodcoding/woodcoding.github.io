---
title: 获取百度搜索结果时解决人机滑动验证码的问题
date: 2020-02-12 23:19:00
categories: 
 - 爬虫
tags:
 - 百度
 - selenium
 - 爬虫
 - 验证码
---

### 摘要

前阵子把毕设重构了一遍，爬虫部分涉及到了一个需求：获取一个oj题目在csdn或者博客园的题解链接。由于博客园和csdn都没有找到好的api可以调用，百度api也是收费的，所以就想能不能利用爬虫直接爬取百度搜索结果的前列，然后做百度url转真实解析。具体的一些设计思路后面再说，这里我们先来讨论爬虫中的一个问题，就是解决验证码的问题。首先由于requests尝试好几次都不行，所以换了selenium，虽然慢一点，但是至少有个解决方案了。但是百度的技术也不差，就检测到了我的爬虫行为（之后加了各种办法来避免这个问题，到写这篇文章的时候，本来想把验证码图给大家截一下，但是我不管怎么暴力爬百度还是不给验证码页面，这个以后遇到再补吧），就出现了滑动验证的人机检测。不过由于这个滑动验证码实在是太鸡肋了，就是直接滑倒最右边，所以相比于其他一些图形验证码来说还是比较简单的，废话说了一大堆，下面就来说说解决办法。 

<!-- more -->

### 前言

首先我们来看一下这个滑动验证码长什么样子（百度的因为日志图太多被我删了，网上随便找了个差不多的，将就看一下吧） 
![](/resources/images/2020-02-12/verify_code.png)

解决思路很简单，就是模拟人把这个按钮移到最右边。但是要考虑人移动滑块是有加速和减速的，不然还是会被检测为人机，所以我们待会用到初中物理的知识去解决他。 

### Step1:获取到验证码界面的信息

- 获取滑动按钮的对象： 

```python
btn_id = driver.find_element_by_class_name('vcode-slide-button').get_attribute('id')
btn = driver.find_element_by_id(btn_id)
```

- 获取滑动条的长度：

```python
distance = driver.find_element_by_class_name('vcode-slide-bottom').size['width'] - btn.size['width'] + random.randint(0, 4)
```

这里我获取了滑动条长度，然后减去了按钮的宽度加了个随机长度。减去按钮宽度肯定是要的，然后这个随机长度是因为我发现直接滑这么长还是会被检测到，或者是出现有时候滑不到的情况，毕竟人哪有这么精准。

### Step2: 设计滑动路径

初中物理的路程、速度与加速度公式大家应该都熟记于心了。 

$ V = V_0 + at $
$ S = V_0t +\frac{1}{2}at^2 $

然后我们先来看代码： 

```python
def get_track(distance):
    """
    利用加速度公式构建滑动路径
    :param distance: 滑动距离
    :return:
    """
    tracks = []
    current = 0
    mid = distance * 3 / 4
    t = random.randint(20, 40) * 0.01
    v = 0
    while current < distance:
        a = 10 if current < mid else -20
        v0 = v
        v = v0 + a * t
        move = v0 * t + 1 / 2 * a * a * t
        current += move
        tracks.append(round(move))
    return tracks
```

首先我们传入了一个distance作为滑动距离，然后获得一个加速和减速之间的一个过渡点，这里取$\frac{3}{4}$长度作为过度点。然后随机一个时间作为我们滑动所需要的时间，这里取了0.2s到0.4s之间一个随机值。然后我们通过基本公式计算出每次移动到的位置并加入到tracks列表里面，之后把tracks，即移动路径返回。这里修改不同的变量都可能导致移动轨迹的大幅变化，随自己需要调整即可。 



### Step3：Selenium模拟滑动

```python
def move_to_right(button, driver, distance):
    """
    移动滑块到最右
    :param button: 滑动按钮
    :param driver: selenium驱动
    :param distance: 滑动距离
    :return:
    """
    tracks = get_track(distance)
    log.debug("验证码移动轨迹：{msg}".format(msg=tracks.__repr__() + 'moving...'))
    action = ActionChains(driver)
    action.click_and_hold(button).perform()     # 模拟按下
    begin = button.location['x']

    for x in tracks:
        try:
            action.move_by_offset(xoffset=x, yoffset=0).perform()       # 模拟移动
            now = button.location['x'] - begin
            process = round((now / distance) * 100, 2)
            log.debug('当前验证码进度：{process}%'.format(process=process))
        except selenium_error.StaleElementReferenceException:
            break
    process = 100.00
    log.debug('当前验证码进度：{process}%'.format(process=process))
    time.sleep(1)
```

这里我们拿到滑动的按钮和selenium对象，以及上面第二部计算出的路径值，然后用selenium自带的ActionChains函数执行动作。先模拟按压，然后不断移动。中间可能出现验证码完成验证但是还会移动的情况（随机出现），所以当我们获取不到滑动元素的时候说明验证码已经完成验证了。然后就可以愉快的获取你想要的东西啦。 
