---
weight: 1000
title: "pytest web 开发中的单元测试工具"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "pytest web 开发中的单元测试工具"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python编程]

lightgallery: true

toc:
  auto: false
---

pytest，Python的单元测试工具，用于web开发中做测试，使用也比较简单。关于测试又是一个大的分支，编写测试代码也有一定的套路，比如测试细节步骤，测试各个接口，但是个人认为测试一个整体的模块结果会比较容易。

另外不要为了测试而写测试，测试应该和业务代码相互结合，有些业务代码确实不好测试，有些过于简单又不用测，这导致测试的效率，测试的覆盖程度大大折扣，所以又衍生出`基于测试驱动开发的模式`，一开始就考虑好如何测试，如何编写代码，集成的时候以测试结果作为评审依据，让整个系统更加健壮。

所以，好的架构要有测试用例，测试用例还能用来理解代码，不要在系统都开发到一大半的时候，才来考虑测试的问题，这个时候测试代码很难接入，要么对各个接口做单独测试，要么把业务抽象出来，测试大功能。

<!-- more -->

## 命令

```sh
py.test
py.test --version
py.test name.py
py.test --resultlog=report  保存测试报告

py.test               # run all tests below current dir  
py.test test_mod.py   # run tests in module  
py.test somepath      # run all tests below somepath  
py.test -k stringexpr # only run tests with names that match the  
                      # the "string expression", e.g. "MyClass and not method"  
                      # will select TestMyClass.test_something  
                      # but not TestMyClass.test_method_simple  

py.test test_mod.py::test_func  # only run tests that match the "node ID",  
                                # e.g "test_mod.py::test_func" will select  
                                # only test_func in test_mod.py  
```

- 测试文件以test_开头（以_test结尾也可以）
- 测试类以Test开头，并且不能带有 __init__ 方法
- 测试函数以test_开头
- 断言使用基本的assert即可

## 相关装饰器讲解

>@pytest.fixture(scope="module")

这个装饰器会让被装饰的函数只执行一次，通常用来装饰需要在多个测试直接使用到的同一个变量，例如：

{% codeblock lang:py %}
@pytest.fixture(scope='module')
def get_result():
    return 1
{% endcodeblock %}

>把上面的代码放入conftest.py 文件中（经测试这个文件名字不能改，不知道为什么）

这样在其它测试文件中，我们可以把get_result做为参数传入，例如：
文件  test_func.py

```python
class TestApi:
    def test_func(self, get_result):
        a = get_result
        assert a != 1
```        

pytest.mark.usefixtures  装饰器的使用，就是显式的调用，先用@pytest.fixture()把需要公用的函数做装饰，使用的时候通过pytest.mark.usefixtures装饰，就可以得到公用函数了，下面展示了使用三种方法：

```python
import pytest
@pytest.fixture()
def before():
    print('\nbefore each test')
@pytest.mark.usefixtures("before")
def test_1():
    print('test_1()')
@pytest.mark.usefixtures("before")
def test_2():
    print('test_2()')
class Test1:
    @pytest.mark.usefixtures("before")
    def test_3(self):
        print('test_1()')
    @pytest.mark.usefixtures("before")
    def test_4(self):
        print('test_2()')
@pytest.mark.usefixtures("before")
class Test2:
    def test_5(self):
        print('test_1()')
    def test_6(self):
        print('test_2()')
```

关于 fixture 的scope参数

- function：每个test都运行，默认是function的scope 
- class：每个class的所有test只运行一次 
- module：每个module的所有test只运行一次 
- session：每个session只运行一次

比如你的所有test都需要连接同一个数据库，那可以设置为module，只需要连接一次数据库，对于module内的所有test，这样可以极大的提高运行效率。

autouse 参数

这个参数默认是Fals, 设置为True的时候，一个session的所有test都会自动调用autouse设置为True的fixture。就是自动调用了，你都不需要把fixture作为参数传入，比如你的fixture写成打印一些友好提示，这样在每个测试执行的时候，这些提示都会打印出来。