---
title: 简述软件开发中的单元测试
date: 2019-03-24 23:04:00
categories: 
 - 测试
tags:
 - unittest
 - 单元测试
 - 测试
 - mock
 - nose
---

## 单元测试
---

### 简介
#### 单测介绍与目的
单元测试就是编写一些测试代码，用来对一个函数，类对象，或者模块进行检测。一来可以保证程序执行的正确性，二来也可以尽量避免低级别的bug以及改动引发的其他模块bug（比如修1个bug，多出100个bug这种）。并且还有大名鼎鼎的TDD（Test-Driven Development）测试驱动开发。所以用测试来保证我们代码符合预期需求是非常重要的，也能让我们对产品的迭代更加有信心巴拉巴拉...是吧，反正很重要就是了。但是很多程序员（包括我）都是不怎么喜欢写测试的，毕竟是测试而不是功能代码，对自己的功(la)能(ji)代码自信得一批。幸好是刚出来的应届生还可以慢慢适应，不然不写单测怕是没人要了。  
<!-- more -->

#### 单测工具(unittest)
单测框架工具有很多，像Java就有Java的Junit，unittest就是python中的Junit。这个框架是Python自带的，其官方文档地址如下：  
> https://docs.python.org/3.6/library/unittest.html

文档是英文的，不过基本的还是能看懂的，虽然我目前还没去啃，毕竟英语也是半斤八两，别看了半天手头上的工作还没完成。先找找一些中文的博客看看，顺便借(cao)鉴(xi)一下他们漂亮的图，是吧。  
对于unittest主要有6个概念要搞清：  
- Test Loader：用来加载测试案例，即扫描以test开头的Test case方法，加载到run中运行。
- Test Case：一个TestCase即为一个测试用例，包括了测试用例函数执行前的准备工作setUp，执行代码run，以及执行完之后的tearDown还原工作。
- Test Suite：多个测试用例的集合，就是将多个Test Case组合起来，共同执行的一套集合，可以用addTest或者addTests进行测试用例的添加。
- Test Runner：执行测试用例，既可以是Test Case，也可以是Test Suite。执行之后将结果送入Test Result中。
- Test Result：管理Test Runner执行后的用例结果，是正确还是错误还是跳过，执行状态是：F(失败)，.(成功)，E(执行出错)，S(跳过执行)
- Test Fixture：测试用例的环境准备。
##### 基本方法
###### 编写测试用例class的方法：
```python
# 某个测试类中所有方法(被load过)执行前执行一次
def setUpClass(self):pass
```
```python
# 某个测试类中所有方法(被load过)执行后执行一次
def tearDownClass(self):pass
```
```python
# 某个测试类中每个test开头方法执行前都执行一次
def setUp(self):pass
```
```python
# 某个测试类中每个test开头方法执行后都执行一次
def tearDown(self):pass
```
```python
# 测试函数
def test_***(self):pass
```

###### 断言
我们测试一个过程正确或者错误都是通过断言。  
    
| 方法                      |检测例子               |首次出现的Python版本  |
| ------------------------- | -------------------- | --- |
| assertEqual(a, b)         | a == b               |     |
| assertNotEqual(a, b)      | a != b               |     |
| assertTrue(x)             | bool(x) is True      |     |
| assertFalse(x)            | bool(x) is False     |     |
| assertIs(a, b)            | a is b               | 3.1 |
| assertIsNot(a, b)         | a is not b           | 3.1 |
| assertIsNone(x)           | x is None            | 3.1 |
| assertIsNotNone(x)        | x is not None        | 3.1 |
| assertIn(a, b)            | a in b               | 3.1 |
| assertNotIn(a, b)         | a not in b           | 3.1 |
| assertIsInstance(a, b)    | isinstance(a, b)     | 3.2 |
| assertNotIsInstance(a, b) | not isinstance(a, b) | 3.2 |
| assertEqual(a, b)         | a == b               |     |
| assertNotEqual(a, b)      | a != b               |     |

###### 跳过执行装饰器
- @unittest.skip(reason)
直接跳过，不执行

- @unittest.skipIf(condition, reason)
condition为真时跳过

- @unittest.skipUnless(condition, reason)
condition为假时跳过

- @unittest.expectedFailure
测试执行失败时跳过统计

##### 代码练习
偷偷看看官方文档，然后借鉴几个例子。  
- 初步尝试
官方的第一个例子，简单感受一下测试的魅力。这里主要是测试了字符串大小写以及异常分割的断言。
```python
import unittest

class TestStringMethods(unittest.TestCase):

    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())

    def test_split(self):
        s = 'hello world'
        self.assertEqual(s.split(), ['hello', 'world'])
        # 以一个整数split会报TypeError
        with self.assertRaises(TypeError):s.split(2)

if __name__ == '__main__':
    unittest.main()
```
看下输出结果：
```bash
...
----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```
这里3个案例都执行成功了，我们来搞两个错的试试。
```python
import unittest

class TestStringMethods(unittest.TestCase):
    def test_upper(self):
        self.assertEqual('foo'.upper(), 'fOO')  # 错误1

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())
        
    def test_split(self):
        s = 'hello world'
        self.assertEqual(s.split(), ['hello', 'world'])
        # 以一个整数split会报TypeError
        with self.assertRaises(ZeroDivisionError):s.split(2)    # 错误2

if __name__ == '__main__':
    unittest.main()
```
输出结果如下：

```bash
.EF
======================================================================
ERROR: test_split (__main__.TestStringMethods)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test2.py", line 15, in test_split
    with self.assertRaises(ZeroDivisionError):s.split(2)
TypeError: must be str or None, not int

======================================================================
FAIL: test_upper (__main__.TestStringMethods)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "test2.py", line 5, in test_upper
    self.assertEqual('foo'.upper(), 'fOO')
AssertionError: 'FOO' != 'fOO'

----------------------------------------------------------------------
Ran 3 tests in 0.002s

FAILED (failures=1, errors=1)

```
这里我们制造了两个错误，一个是函数功能测试错误，另一个是抛出异常错误，注意E和F的区别。F是指你的测试案例执行失败（案例执行成功），E是指你的测试案例中存在执行错误(一般是测试代码写错)。现在看到测试结果出现全对的时候也是有那么点小激动的。

- 方法执行流程
刚刚前面说了那么多方法，那么每个方法的执行流程是怎么样的呢？让我们来尝试一下。
```python
import unittest

class MyTest(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        print('setUpClass')
    
    @classmethod
    def tearDownClass(cls):
        print('tearDownClass')
    
    def setUp(self):
        print('--setUp--')
    
    def tearDown(self):
        print('--tearDown--')
    
    def test_haha(self):
        print('原来我才是最终boss')
    
    def test_haha2(self):
        print('原来我才是最终boss2')


if __name__ == "__main__":
    unittest.main()
```

执行结果如下：

```bash
setUpClass
--setUp--
原来我才是最终boss
--tearDown--
.--setUp--
原来我才是最终boss2
--tearDown--
.tearDownClass

----------------------------------------------------------------------
Ran 2 tests in 0.005s

OK
```

- 加载多个
假设我现在有多个testcase，我要一起加载执行怎么办呢。这时候就可以用到我们的load和suite了。
- suite.addTest() # 加载一个测试用例
- suite.addTests() # 加载系列测试用例（按加载过程有序）

单个添加  
```python
suite = unittest.TestSuite()
suite.addTest(MyTest('test_upper'))
runner = unittest.TextTestRunner(verbosity=2)  # 设置verbosity=2可以显示测试详情
runner.run(suite)
```
多个添加  
```python
suite = unittest.TestSuite()
tests = [MyTest('test_upper'), MyTest('test_split')]
suite.addTests(tests)
runner = unittest.TextTestRunner(verbosity=2)  # 设置verbosity=2可以显示测试详情
runner.run(suite)
```
利用Load添加  
```python
suite = unittest.TestSuite()
tests1 = unittest.TestLoader().loadTestsFromNames(['MyTest.test_upper', 'MyTest.test_solit'])
# tests2 = unittest.TestLoader().loadTestsFromTestCase(MyTest)
# 更多用法参考官方文档
suite.addTests(tests)
runner = unittest.TextTestRunner(verbosity=2)  # 设置verbosity=2可以显示测试详情
runner.run(suite)
```


##### 原理分析
让我们来看看框架的类图(来自网络)：
![](/resources/images/2019-03-31/unittest.png)
首先，由TestLoader进行扫描加载，把所有test开头的TestCase测试案例都加入到TestSuite，再由TestRunner运行suite，最后将执行结果输出到result。对于更详细的执行原理，更加详细的执行过程，可以通过runner.py文件源码进行查看。

#### 覆盖率(coverage)

##### 简介
Coverage是一种用于统计Python代码覆盖率的工具，通过它可以检测测试代码对被测代码的覆盖率如何。Coverage支持分支覆盖率统计，可以生成HTML/XML报告。官方文档地址：
> https://coverage.readthedocs.io/en/latest/

##### 用法
- 控制台显示覆盖率：
```bash
(venv) C:\tests>coverage report -m
Name                Stmts   Miss  Cover   Missing
-------------------------------------------------
HTMLTestRunner.py     200     26    87%   118, 121, 124, 578, 581-591, 604, 614, 618, 664, 719, 725, 768, 774, 813-815, 824
main.py                30     11    63%   4, 7, 11, 15, 19-22, 35-36, 48-50
-------------------------------------------------
TOTAL                 230     37    84%
```

- 生成HTML网页报告(用HTMLTestRunner也可以生成报告，不过对于py3的支持要修改部分代码)
```
coverage html
```

##### 原理分析
分支检测原理：在这里，Coverage利用了trace追踪机制，你肯定接触过这东西，就是调试的时候。而对于web程序就更厉害了，web程序一般是循环监听，除非异常情况否则不会自动退出。而Coverage在实现上使用了atexit模块注册一个回调函数，在Python退出时将内存中的覆盖率结果写到文件中。被测脚本只有正常退出或者以SIGINT2信号退出才能出发atexit，才能得到覆盖率结果。CTRL+C发的即是SIGINT2信号，所以前台启动的服务用CTRL+C停止后可以出结果。而atexit它内部又是通过sys.exitfunc来实现的。

#### 模拟数据(mock)
##### 简介
我们每次跑测试，肯定需要一些资源数据，但是不可能每次都用真实数据去跑。例如，测试资源不可用，或者不适合，他正开发中，测试资源太昂贵，不可预知，等等情况。这时候我们就需要使用mock来进行数据的模拟。官方文档地址：
> https://docs.python.org/3.6/library/unittest.mock.html

> 注意Python3中mock是自带的，python2中还要自己安装，pip install mock 一下即可，这里我使用的是python3

##### 基本方法
这里我们主要介绍一下，mock时的传参：
```
class Mock(spec=None, side_effect=None, return_value=DEFAULT,...)
```
name：这个是用来命名一个mock对象，只是起到标识作用，打印mock对象name的时候可以看到。
spec: 可以是字符串列表，也可以是充当模拟对象规范的现有对象（类或实例）。如果传入一个对象，则通过在对象上调用dir来形成字符串列表（不包括不受支持的魔术属性和方法）。访问不在此列表中的任何属性将引发AttributeError。如果spec是一个对象（而不是一个字符串列表），那么__class__将返回spec对象的类，允许模拟通过isinstance（）测试。
spec_set: 比sepc更严格，用sepc你还可以设置一个未指定的属性。 使用spec_set后尝试在mock上设置或获取不在spec_set传递的对象上的属性将引发AttributeError。
side_effect: 例如：`mock.Mock(return_value=10, side_effect=code.my_mvalue)`设置此属性之后mock的return_value不生效，调用原my_value真正的返回值
return_value: 例如：`mock.Mock(return_value=10)`设置之后，调用时将返回设定的值，原函数的返回值将无效
unsafe: 3.5版本新特性。默认情况下，如果任何属性以assert或assret开头，则会引发AttributeError。 传递unsafe = True将允许访问这些属性。

##### 代码练习
- 常规操作
让我们来mock一个价值百万的数据返回
```python
from unittest import mock
import unittest


def my_mock():
    return '价值100w的返回值'


class TestMyMock(unittest.TestCase):
    def test_01(self):
        '''测试返回成功'''
        # mock一个成功的数据
        my_mock = mock.Mock(return_value='100w')
        self.assertEqual(my_mock(), '100w')

    def test_02(self):
        '''测试返回失败'''
        # mock一个失败的数据
        my_mock = mock.Mock(return_value='想得美')
        self.assertEqual(my_mock(), '100w')

if __name__ == "__main__":
    unittest.main()

```
看下输出，发现一个是成功的，一个是失败的，说明我们的mock数据是生效了的：
```bash
.F
======================================================================
FAIL: test_02 (__main__.TestMyMock)
测试失败场景
----------------------------------------------------------------------
Traceback (most recent call last):
  File "c:\test4.py", line 21, in test_02
    self.assertEqual(my_mock(), '100w')
AssertionError: '想得美' != '100w'
- 想得美
+ 100w
----------------------------------------------------------------------
Ran 2 tests in 0.001s
FAILED (failures=1)
```

- 使用装饰器mock一个模块下的类  
这里我们先mock一下Code类，然后测试Use类，由于Use类调用了Code，因此可以达到我们想要的mock结果。
> 注意：1. 参数中的mock_Code不是像test那样限制开头，可以随便命名
> 2. 这里我把类都写在了一个文件里面，所以使用了'__main__'(当前运行文件会设为这个值，下回有机会讨论一下)

```python
import unittest
from unittest import mock


class Code:
    def my_value(self):
        return '100w'

class Use:
    def get_my_money(self):
        code = Code().my_value()
        msg = None
        if code == '100w':
            msg = '发财啦'
        elif code == '10cents':
            msg = '你个穷逼'
        else:
            msg = '常规操作'
        return msg


class MyTest(unittest.TestCase):
    def test_01(self):
        '''测试普通方法'''
        use = Use()
        msg = use.get_my_money()
        self.assertEqual('发财啦', msg)
    
    @mock.patch('__main__.Code')
    def test_02(self, mock_Code):
        '''测试mock方法'''
        c = mock_Code()  # 由于是个类对象，因此先实例化
        c.my_value.return_value = '10cents'
        use = Use()
        msg = use.get_my_money()
        self.assertEqual('你个穷逼', msg)


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

输出  

```bash
test_01 (__main__.MyTest)
测试普通方法 ... ok
test_02 (__main__.MyTest)
测试mock方法 ... ok

----------------------------------------------------------------------
Ran 2 tests in 0.004s

OK
```

##### 原理分析
让我们看下Mock的类继承关系：
![](/resources/images/2019-03-31/mock.png)
Mock继承自CallableMixin, NonCallableMock，而CallableMixin, NonCallableMock又是继承自Base，Base里面的代码其实只有4行，定义了return_value和side_effect的默认值，而具体的一些方法则是写入了CallableMixin和NonCallableMock：
```python
class Base(object):
    _mock_return_value = DEFAULT
    _mock_side_effect = None
    def __init__(self, *args, **kwargs):
        pass
```
CallableMixin中主要是负责调用逻辑封装，签名检查，对象调用统计等工作以及最重要的我们要使用的一些调用，而NonCallableMock中则是重写一些魔法方法来达到mock数据的目的，魔法方法的一些用法目前还不是很懂就不献丑了。  

#### 集成框架(nose)
nose框架主要是把一些测试的流程都给串了起来，包括自动发现test命名的测试案例进行测试，覆盖率，报告生成之类的，在大型项目上可以应用上。这个不是python的标准库，需要使用`pip install nose`来进行安装。安装之后使用`nosetests -h`可以查看相关的命令。  

### 总结

怎么说，这一套流程搞下来其实感觉花的时间并不会比开发过程少，甚至可能开发一小时，测试两小时都有可能。不过为了保证代码能够如期执行，多人协作的时候不容易出岔子，还是搞一搞比较好。毕竟写测试的时候需要你往很多方面去思考这个功能的执行状态，能够尽可能地避免bug（大神请绕道）。希望以后的工作中能养成写优秀测试的习惯吧。  