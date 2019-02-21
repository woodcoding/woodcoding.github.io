---
title: Python调试之RPDB工具
date: 2019-02-21 17:00:00
categories: 
 - Python
tags:
 - Python
 - 工具
 - 调试
 - PDB
---

## Python调试之RPDB工具
开发调试工具有很多种，最原始古老的方式应该都知道，就是直接print，但是那样效率比较低下，有可能要多次重启服务才能调试完成。因此，选择一种高效的调试工具对开发来说是非常必要的。像以前用C/C++的时候有接触过gdb，但是那时候写个传统算法其实用print足够了，再不行codeblocks上面还有调试工具呢，但是如果是在linux系统下非图形界面还是没辙啊。对python来说，`pdb`，`rpdb`，`pycharm-debug.egg`，`pydevg`等工具都可以用来调试。同理，pycharm的调试虽然好，但是对于非图形界面的话就还是用pdb（不过现在pycharm也支持远程调试了，不过设置方面要麻烦点）。下面简要介绍下pdb的使用。

<!-- more -->
### 1. PDB(The Python Debugger)
看名字就知道，pdb就是python的debug工具，而且这个工具python安装后都是自带的。使用也挺简单，直接import再set_trace就好了。这里先给出一些常用命令，然后我们再一一解释用法。

| 命令            | 简写        | 作用                         |
| ----------------- | ------------- | ----------------------- |
| args              | a             | 查看传入参数             |
| break             | b             | 显示所有断点              |
| break lineno      | b lineno      | 在指定行设置断点          |
| break file:lineno | b file:lineno | 在指定文件的行设置断点     |
| bt                |               | 查看函数调用栈帧          |
| clear num         |               | 删除指定断点              |
| clear     | cl        | 清除断点                       |
| condition | 无       | 设置条件断点                 |
| continue  | c或者cont | 继续运行，知道遇到断点或者脚本结束 |
| disable   | 无       | 禁用断点                       |
| enable    | 无       | 启用断点                       |
| help      | h         | 查看pdb帮助                    |
| ignore    | 无       | 忽略断点                       |
| jump      | j         | 跳转到指定行数运行        |
| list      | l         | 列出脚本清单                 |
| next      | n         | 执行下条语句，遇到函数不进入其内部 |
| print     | p         | 后面跟变量名，打印变量值 |
| quit      | q         | 退出 pdb                         |
| return    | r         | 一直运行到函数返回或者下一次断点 |
| step      | s         | 执行下一条语句，遇到函数进入其内部 |
| tbreak    | 无       | 设置临时断点，断点只中断一次 |
| where     | w         | 打印堆栈信息，查看所在的位置 |
| !         | 无       | 在pdb中执行语句              |
|                   | 回车        | 重复上一条命令              |

#### 进入调试
  - 执行时调试
    程序直接停在入口开始等待调试
    ```bash
    python -m pdb your_module_file.py
    ```
  - 交互调试
    程序运行后打开python解释器进行调试  
    将需要调试的函数名通过pdb来运行
    ```python
    import pdb
    pdb.run('main()')
    ```
  - 埋点调试
    在需要调试的地方埋下断点，运行到时会进入调试
    ```python
    import pdb 
    pdb.set_trace()
    ```

#### 调试过程
  - 命令的使用
    进入调试后会显示一个`(Pdb) `的前缀，然后就可以使用上面的命令列表进行操作了
    ```bash
    # 以下输出我做了对齐处理，控制台里面直接显示的有点乱，
    # 刚开始看网上教程的时候看到一堆乱符号有点懵逼
    (Pdb) l
        [EOF]
    (Pdb) w
        c:\python27\lib\bdb.py(400)run()
        -> exec cmd in globals, locals
        > <string>(1)<module>()
    (Pdb) b
    ```
    具体命令干啥用的自己用上面的命令试试就知道了，注意一下某些语句的区别。比如next是直接执行下一语句，函数也是当做语句执行，要进入函数要使用step单步调试，continue是一直执行到下一断点中间不停顿(不报错的话)......
  - 执行代码（修改参数）
    这个还是很强大的，加`!`前缀可以直接执行代码动态调试，完全不用停止服务。因此也可以用来修改一些参数的值等
    ```bash
    (Pdb) !100+200
        300
    (Pdb) !a=666
    (Pdb) !a+1000
        1666
    (Pdb) 
    ```

#### 总结
pdb主要的一些特性是可以对程序进行设置断点、单步调试、查看代码、查看栈片段、动态改变变量的值等。但是pdb调试有个明显的缺陷就是对于多线程，远程调试等支持得不够好，同时没有较为直观的界面显示，不太适合大型的 python 项目。而在较大的 python 项目中，这些调试需求比较常见，因此需要使用更为高级的调试工具。

### 2. RPDB(remote debugger based on pdb)
RPDB从名字上可以看出，他是一个支持远程调试的工具。其实rpdb就是在pdb的基础上做了一层封装，让pdb支持了远程调试。因此rpdb的源码也很简短，只有172行，可以在github上面看到：
> [`https://github.com/tamentis/rpdb/blob/master/rpdb/__init__.py`](https://github.com/tamentis/rpdb/blob/master/rpdb/__init__.py)

RPDB默认监听了本地IP127.0.0.1，端口是4444，这个也是可以自己调的。

#### 进入调试
调试的命令就不贴了，因为是对PDB的封装，所以和上面的PDB命令是一样的。这里因为RPDB支持远程调试，调试方式上可能有点细微的差别，要设置一下调试的IP和PORT，然后telnet连接一下，而且上面的三种方式中只支持埋点调试。

- 默认埋点调试
  直接在地方埋点就好了，和pdb是一样的
  ```python
  import rpdb
  rpdb.set_trace()
  ```

- 自定义IP端口调试
  毕竟rpdb的优点就在于远程调试，要是只能本地机器上跑一跑运行时环境还是不太够，因此我们可以自定义一下使得可以在远程主机上进行debug。
  ```python
    from rpdb import Rpdb
    rpdb = Rpdb("0.0.0.0", 5555)
    rpdb.set_trace()
  ```
  这样我们就可以通过其他主机也能进行调试了，不过要注意环境安全性，毕竟这样的话又没有密码，任何人都可以连接上来调试了。

#### 调试过程
  由于是远程调试，因此需要借助远程工具来连接一下，我在win环境下使用的是自带的telnet工具。
  运行模块代码之后，会出现如下提示：
  ```
  pdb is running on 127.0.0.1:4444
  ```
  可以看到他默认监听了本地的4444端口。  
  然后打开一个命令行窗口，用系统自带的telnet工具连接上去进行调试：
  ```bash
  telnet 127.0.0.1 4444
  ```
  连接成功，熟悉的pdb界面就回来了：
  ```cmd
  > c:\users\sangfor\pycharmprojects\sxf-study-python2\rpdb_test\test_01.py(11)<module>()
    -> import random
    (Pdb)
  ```

#### 总结
RPDB是对PDB的包装拓展，让PDB拥有了远程调试的功能。使用RPDB调试的代码在远程服务器上启动时，本地可以通过telnet工具进行连接调试，除了需要连接之外的调试过程和普通pdb是没有什么区别的。

### 3.其他调试工具
- PyCharm 同时提供了较为完善的调试功能，支持多线程，远程调试等，可以支持断点设置，单步模式，表达式求值，变量查看等一系列功能。 
针对远程调试功能，PyCharm提供了pydevd模块，该模块以pycharm-debug.egg的形式存在于PyCharm的安装路径中。远程计算机安装该库文件后，然后就可以调用pydevd.settrace方法，该方法会指定IDE所在机器的IP地址和监听的端口号，用于与IDE建立连接；建立连接后，便可在IDE中对远程在远程计算机中的程序进行单步调试。  
图形化界面下，Pycharm的调试功能是非常好用的。
- PyDev 是一个开源的的 plugin，它可以方便的和 Eclipse 集成，提供方便强大的调试功能。同时作为一个优秀的 Python IDE 还提供语法错误提示、源代码编辑助手、Quick Outline、Globals Browser、Hierarchy View、运行等强大功能。
