---
weight: 5500
title: "python 学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python 学习笔记"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

基础永远是重中之重，虽然在应用开发中，很少会去使用语言的一些特性，比如一些高级话题，描述符，元类。但是掌握这些可以帮助理解框架源代码，从更深层次理解语言。

<!-- more -->

## callable(object)

检查对象object是否可调用。如果返回True，object仍然可能调用失败；但如果返回False，调用对象ojbect绝对不会成功。
- 注意：类是可调用的，而类的实例实现了__call__()方法才可调用。
- 版本：该函数在python2.x版本中都可用。但是在python3.0版本中被移除，而在python3.2以后版本中被重新添加。

## dir() 函数

dir() 函数不带参数时，返回当前范围内的变量、方法和定义的类型列表；带参数时，返回参数的属性、方法列表。如果参数包含方法 `__dir__()`，该方法将被调用。如果参数不包含 `__dir__()`，该方法将最大限度地收集参数信息。

与之对应的属性有 `__dict__`，也可以查看对象的属性，实例的 `__dict__` 仅存储与该实例相关的实例属性，正是因为实例的 `__dict__` 属性，每个实例的实例属性才会互不影响。类的__dict__存储所有实例共享的变量和函数(类属性，方法等)，类的`__dict__` 并不包含其父类的属性。所以不能通过 `__dict__` 在一个继承关系中，尤其是还动态修改属性后，判断属性是否存在，要获取完整的属性列表，使用dir()。

## 标准库 inspect

inspect 作为Python的标准库，主要有以下作用：
1. 对是否是模块，框架，函数等进行类型检查。

2. 获取源码

3. 获取类或函数的参数的信息

4. 解析堆栈

一般来说，可以得到对象的各种信息，函数的参数，类的文档字符串等。

## 继承一个类

如果已经定义了Person类，需要定义新的Student和Teacher类时，可以直接从Person类继承：

```python

    class Person(object):

        def __init__(self, name, gender):
            self.name = name
            self.gender = gender

```

定义Student类时，只需要把额外的属性加上，例如score：

```python

    class Student(Person):

        def __init__(self, name, gender, score):
            super(Student, self).__init__(name, gender)
            self.score = score

```

一定要用 `super(Student, self).__init__(name, gender)` 去初始化父类，否则，继承自 Person 的 Student 将没有 name 和 gender。
函数 `super(Student, self)` 将返回当前类继承的父类，即 Person ，然后调用__init__()方法，注意self参数已在super()中传入，在__init__()中将隐式传递，不需要写出（也不能写）。

`super(Teacher,self).__init__(name,gender)` 和 `Person.__init__(self,name,gender)` 等价的，二者选一就好。

:::tip
在新的py3中，使用super变的简洁了，`super().__init__(name, gender)`，类的形式`Person.__init__(self,name,gender)`没变
:::

假如一个类C继承了类A和类B，类A和类B有不同的属性，并且类C在创建时要初始化这些属性，此时在类C的构造函数__init__中使用super(C，self).__init__调用就无法实现了

```python

    class A(object):
        def __init__(self, a):
            self.a = a

    class B(object):
        def __init__(self, b):
            self.b = b

    class C(A, B):
        def __init__(self, a, b):
            super(C, self).__init__(a, b)  # <----这样写是错误的

```

正确的写法:

```python

    class C(A, B):

        def __init__(self, a, b):
            A.__init__(self, a)
            B.__init__(self, b)

```

**建议养成习惯，不要使用super()这个函数，即便是单继承，也使用上面的方式**

## 反射 & 自省

当执行对象的方法，或者对某个字段赋值的时候，你要操作的字段名或者方法名在编码的时候不确定，这时候需要通过某种机制访问未知的属性。

这个机制被称为反射（反过来让对象告诉我们他是什么），或是自省（让对象自己告诉我们他是什么）。

1. 使用反射获取到的函数和方法可以像平常一样加上括号直接调用
2. 获取到类后可以直接构造实例
3. 不过获取到的字段不能直接赋值，因为拿到的其实是另一个指向同一个地方的引用，赋值只能改变当前的这个引用而已

## 单下划线/双下划线

Python 用下划线作为变量前缀和后缀指定特殊变量/方法。

主要存在四种情形：
1. `object` # public
2. `__object__` # special, python system use, user should not define like it
3. `__object` # private (name mangling during runtime)
4. `_object` # obey python coding convention, consider it as private;

核心风格：避免用下划线作为变量名的开始。

因为下划线对解释器有特殊的意义，而且是内建标识符所使用的符号，我们建议程序员避免用下划线作为变量名的开始。一般来讲，变量名 `_object` 被看作是“私有 的”，在模块或类外不可以使用，不能用 `'from moduleimport *'`(这种情况单双下划线都适用，而且只是用了*号不能导入，导入整个模块，或直接导入下划线开头的模块都是可行的) 导入。当变量是私有的时候，用 `_object` 来表示变量是很好的习惯。因为变量名 `__object__` 对Python来说有特殊含义，对于普通的变量应当避免这种命名风格。

python有关private的描述，python中不存在protected的概念，要么是public要么就是private，但是python中的private不像C++, Java那样，它并不是真正意义上的private，通过name mangling（名称改编(目的就是以防子类意外重写基类的方法或者属性)，即前面加上“单下划线”+类名，eg：instance._Class__object）机制就可以访问private了。

**"单下划线"**：使用单下划线定义的属性或变量，它表示该方法或者属性是该类型的私有方法或属性。但其实在Python中不存在真正意义上的私有方法或者属性，前面加单下划线_只是表示你不应该去访问这个方法或者属性，因为它不是接口的一部分。

这个意思其实是一种定义，约束，你非要去访问也可以，所以在代码中我们看到单下划线开头的对象，就是让你不要去访问它。举个例子，在Django关于froms的代码中：

```python

    class BaseForm(StrAndUnicode):
    
        def __init__(self, data=None, files=None, auto_id='id_%s', prefix=None):
            # ...
            self._errors = None  # Stores the errors after clean() has been called.

        @property
        def errors(self):
            """Return an ErrorDict for the data provided for the form."""
            if self._errors is None:
                self.full_clean()
            return self._errors

```

这段代码截取自Django框架，可以看到_errors属性是私有的，它在clean()方法调用后被赋值，如果要查看错误，访问errors属性，而不是（也不应该）访问_errors获取错误信息。所以多看看源代码，学习其编程风格，有些人在命名变量的时候，同名的被使用了，就加个下划线，这是错误的风格。

**"双下划线"**：双下划线开头的变量也是设计成私有的概念，不过区别于单下划线，它确实不能直接访问，Python中不存在真正意义上的私有变量。对于双下划线开头的方法和属性虽然我们不能直接引用，那是因为Python默认在其前面加了前缀_类名，通过特别的机制（变量轧压）可以访问到。官方设计双下划线的目的，可以在继承中防止方法被覆写。

```python

    class A:
        def __method(self):
            print('__method in class A')

        def _method(self):
            print('_method in class A')

        def run_method(self):
            print(hasattr(a, '__method'))  # False
            self.__method()
            self._method()


    class B(A):

        def __method(self):
            print('__method in class B')

        def _method(self):
            print('_method in class B')


    a = A()
    a.run_method()

    # output:
    # __method in class A
    # _method in class A
    # 很正常的输出，没有问题

    b = B()
    b.run_method()

    # output
    # __method in class A
    # _method in class B
    # 在解释器编译代码后，会进行变量轧压，或称为属性扩展，A类的__method变成了_A__method
    # 所以实例a，b去执行self.__method()都是去访问属性_A__method，print(dir(a))可以看到
    # _A__method在输出列表中，像hasattr等方法，要用_A__method作为属性名称

```


变量轧压：Python把以两个或以上下划线字符开头且没有以两个或以上下划线结尾的变量当作私有变量。私有变量会在代码生成之前被转换为长格式（变为公有）。转换机制是这样的：在变量前端插入类名，再在前端加入一个下划线字符。这就是所谓的私有 `变量轧压`（Private name mangling）。

属性扩张：一个属性命名前加了2个下划线，属性会扩展为`_类__属性`，属性扩展后，要用扩展后的变量名去获取属性。

注意：
- 一是因为轧压会使标识符变长，当超过255的时候，Python会切断，要注意因此引起的命名冲突。
- 二是当类名全部以下划线命名的时候，Python就不再执行轧压。

总结：理解单双下划线，Python私有变量的真正含义，它并不是语法严格限制，而是一种设计或风格，什么是变量轧压，使用了单双下划线后，`from moduleimport *` 要注意。

## 特殊方法

| name          | Description
| :-----------: | :---------------
| `__class__`   | 类对象的类型
| `__name__`    | 在模块外和类中代表不同的含义，类中调用返回类名（str），只能类调用，实例调用：`self.__class__.__name__`
| `__doc__`     | 类的文档字符串
| `__bases__`   | 类的所有父类组成的元祖，只能类调用，实例需要通过 `__class__`
| `__dict__`    | 得到一个属性字典，类和实例都可以调用，得到它们对应的属性
| `__module__`  | 类定义所在的模块（类的全名是 `__main__.className`，如果类位于一个导入模块mymod中，那么 `className.__module__` 等于 mymod）
| `__str__`     | 实例来调用，打印实例，输出实例都会执行定义的方法
| `__repr__`    | 实例调用，输出不变，打印变
| `__getattribute__` |  获取属性，针对所有属性运行
| `__getattr__` | 获取属性，针对未定义的属性运行
| `__getitem__` | 字典相关获取属性
| `__slots__`   | 限制实例能绑定的属性

## time

该模块归属在 gengrice operating system services中，接近操作系统底层。围绕着Unix  Timestamp 进行。 需要注意的是在该模块中的大多数函数是调用了所在平台C library的同名函数(比如sleep，其实是调用了平台的C代码)， 所以要特别注意有些函数是平台相关的，可能会在不同的平台有不同的效果。另外一点是，由于是基于Unix Timestamp，所以其所能表述的日期范围被限定在 1970 - 2038 之间，如果你写的代码需要处理在前面所述范围之外的日期，那可能需要考虑使用datetime模块更好。为了解决这个问题，C出里新的标准库代替原来的，这里留个心眼。

## 重定向

在Python中，文件对象sys.stdin、sys.stdout和sys.stderr分别对应解释器的标准输入、标准输出和标准出错流。在程序启动时，这些对象的初值由sys.__stdin__、sys.__stdout__和sys.__stderr__保存，以便用于收尾(finalization)时恢复标准流对象。

print语句默认写入标准输出流，也可重定向至文件或其他可写对象(所有提供write方法的对象)。这样，就可以使用简洁的print语句代替笨拙的object.write('hello'+'\n')写法。因此，在python中调用 print obj 打印对象时，缺省情况下等效于调用sys.stdout.write(obj+'\n').

控制台重定向(>和>>)：
Windows命令提示符(cmd.exe)和Linux Shell(bash等)均通过">"或">>"将输出重定向。其中，">"表示覆盖内容，">>"表示追加内容。类似地，"2>"可重定向标准错误。重定向到"nul"(Windows)或"/dev/null"(Linux)会抑制输出，既不屏显也不存盘。

示例：

    Linux 下
    python print_test.py >>out.txt


## argv 获取输入参数

获取输入参数

```python
import sys
a = sys.argv
print(a)

# 在shell输入python print_test.py 321
# 打印：['print_test.py', '321']
```

## 操作Excel

- xlrd 读取文件，但不能对其进行操作
- xlwt 生成Excel文件（可以用来控制生成的格式），但是不能在已有的Excel文件基础上进行修改
- xluntils 可以修改文件，该模块需要依赖于xlrd，xlwt

```py
wb = Workbook()
ws = wb.active #激活 worksheet，就是创建一张表，表名默认，worksheet就是该表的对象
```

xlwt 中  write_merge(5,6,4,7,'hello')  在第五行，第四列这里输入数据，结果跨1行（5+1），
跨3列（4+3）

openpyxl：也可以用来操作Excel，由于文件格式的原因，在超过最大行数的时候，xlwt将不能处理，这时候可以使用 `openpyxl`

## 内置序列函数

- map(func, seq1[, seq2,…]) 
第一个参数接受一个函数名，后面的参数接受一个或多个可迭代的序列，返回的是一个列表。

- filter(func, seq1[, seq2,…])
用法和map一样，对每个迭代的元素执行func，如果返回结果是真，这个值就留下。

- reduce()
函数接收的参数和 map()类似，一个函数 f，一个list，但行为和 map()不同，reduce()传入的函数 f 必须接收两个参数，reduce()对list的每个元素反复调用函数f，并返回最终结果值。

调用 reduce(f, [1, 3, 5, 7, 9])时，reduce函数将做如下计算：

1. 先计算头两个元素：f(1, 3)，结果为4；
2. 再把结果和第3个元素计算：f(4, 5)，结果为9；
3. 再把结果和第4个元素计算：f(9, 7)，结果为16；
4. 再把结果和第5个元素计算：f(16, 9)，结果为25；
5. 由于没有更多的元素了，计算结束，返回结果25。

上述计算实际上是对 list 的所有元素求和。虽然Python内置了求和函数sum()，但是，利用reduce()求和也很简单。`__iter__`，`__next__` 这个结合，实现一个迭代器，让对象可以迭代。

- zip([iterable, ...])
zip()是Python的一个内建函数，它接受一系列可迭代的对象作为参数，将对象中对应的元素打包成一个个tuple（元组），然后返回由这些tuples组成的list（列表）。若传入参数的长度不等，则返回list的长度和参数中长度最短的对象相同。利用*号操作符，可以将list unzip（解压）。

```s
>>> a = [1,2,3]
>>> b = [4,5,6]
>>> c = [4,5,6,7,8]
>>> zipped = zip(a,b)
[(1, 4), (2, 5), (3, 6)]
>>> zip(a,c)
[(1, 4), (2, 5), (3, 6)]
>>> zip(*zipped)
[(1, 2, 3), (4, 5, 6)]
```

## 时间模块

- utc时区：协调世界时，又称世界统一时间，世界标准时间，国际协调时间，简称UTC 不属于任意时区

- 中国大陆、中国香港、中国澳门、中国台湾、蒙古国、新加坡、马来西亚、菲律宾、西澳大利亚州的时间与UTC的时差均为+8，也就是UTC+8。

- 时间模块有调用操作系统底层的函数，所以你直接看源码看不出什么东西来，源码上的函数是对应到底层的，这个函数真正的实现在底层。

- 一般用time, datetime这两个模块。

1. time
该模块一般用的少

- 获取时间戳：time.time()
- 由给定的时间返回时间戳：time.mktime(t)，t是一个时间元组。先设一个时间元组，9个参数，最后3位可为0：

```py
t = (2016,7,21,22,47,45,0,0,0）
```

- 时间戳返回日期

```py
import time
 
timeStamp = 1557502800
timeArray = time.localtime(timeStamp)
otherStyleTime = time.strftime("%Y-%m-%d %H:%M:%S", timeArray)
print(otherStyleTime)

import datetime
 
timeStamp = 1557502800
dateArray = datetime.datetime.utcfromtimestamp(timeStamp)
otherStyleTime = dateArray.strftime("%Y-%m-%d %H:%M:%S")
print(otherStyleTime)
```


- 英文显示：time.asctime(t)
- 格式化结果：time.strftime()，time模块是封装好的一个类，可以理解为直接的time就是时间，但是你不能这么用，得调用类的方法，而strftime方法是将结果格式化的方法
- 时区转换：`time.gmtime()` 将时间戳转换成UTC时区，这里的时间戳就是time.time()

2. datetime
常用的时间对象处理模块

datetime 模块包含了几个常用的模块来处理时间，如date日期，datetime当前日期加上时间

- 获取时间：datetime.datetime.now()

```py
datetime.date.today()
```

- 时间计算（如做加减法，返回几天前的日期）：获取的时间 和 datetime.timedelta(days=1)
做计算，如果是加法就可以得到当前时间一天后的时间，也可传分钟minutes等其它变量，用他们英文的复数形式。

- 时间对象格式化，字符串转换时间对象

strftime：这个函数是时间对象的格式化函数，只要你是时间对象就可以用这个格式化，结果是str

strptime：这个是上面是反着的，将一个字符串转换成时间对象，字符串要和格式化模式一致。比如 `‘2017-8-12’` 对应的格式化模式是 `‘%Y-%m-%d'` 这里的Y如果用小写y，就不对了，小写对应的是 `%y`

- time模块可以直接调用上面的函数
datetime模块你要先创建一个时间对象才能使用上面的函数。因为datetime格式化是类的方法，你要先创建一个实例（时间对象）才能调用方法，而对于time来说这个两个是模块里面的函数，不是类的方法。直接看源码即可理解。

| 格式符	|说明
| :-----:  | ----
| %a	   | 星期的英文单词的缩写：如星期一， 则返回 Mon
| %A	   | 星期的英文单词的全拼：如星期一，返回 Monday
| %b	   | 月份的英文单词的缩写：如一月， 则返回 Jan
| %B	   | 月份的引文单词的缩写：如一月， 则返回 January
| %c	   | 返回datetime的字符串表示，如03/08/15 23:01:26
| %d	   | 返回的是当前时间是当前月的第几天
| %f	   | 微秒的表示： 范围: [0,999999]
| %H	   | 以24小时制表示当前小时
| %I	   | 以12小时制表示当前小时
| %j	   | 返回 当天是当年的第几天 范围[001,366]
| %m	   | 返回月份 范围[0,12]
| %M	   | 返回分钟数 范围 [0,59]
| %P	   | 返回是上午还是下午–AM or PM
| %S	   | 返回秒数 范围 [0,61]。。。手册说明的
| %U	   | 返回当周是当年的第几周 以周日为第一天
| %W	   | 返回当周是当年的第几周 以周一为第一天
| %w	   | 当天在当周的天数，范围为[0, 6]，6表示星期天
| %x	   | 日期的字符串表示 ：03/08/15
| %X	   | 时间的字符串表示 ：23:22:08
| %y	   | 两个数字表示的年份 15
| %Y	   | 四个数字表示的年份 2015
| %z	   | 与utc时间的间隔 （如果是本地时间，返回空字符串）
| %Z	   | 时区名称（如果是本地时间，返回空字符串）

- 时间戳和时间对象转换

```py
d = datetime.datetime.fromtimestamp(timeStamp) 
```

`d.timestapo()`，3.0 datetime模块时间对象的方法，如果是2.0，先把时间对象转换成时间元组，利用time模块的time.mktime(t)也可以得到时间戳。

由于time模块的时间对象是 `struct_time` 元组，并不是很好用，所以推荐获取时间戳用time.time()，其余时间处理都使用datetime，然后利用 `datetime.datetime.fromtimestamp(time.time())` 可以由时间戳转换为时间对象。

时间对象处理注意：

`seconds` 是 `timedelta` 对象的属性，datetime不能调用这个属性，即对时间计算才能使用这个属性。另外利用这个去做时间比较不是很靠谱，如果你的差超过了一天，那么转换的秒数溢出了，会显示0，而且如果是负数（理论值），但是时间是没有负秒数的，所以显示会和实际不符（猜测显示的可能为补码的结果）最好的时间差处理是，利用datetie.timedelte加出时间差和现在时间做比较大小。

:sunny:总结：时间对象，时间戳，时间对象格式化成字符串，字符串转换为时间对象，时间对象计算（可以将结果转换成秒，在比较的时候有作用，只要是时间对象，都可以进行算数运算）

## 基本数据结构

1. 基本顺序存储结构——列表与元组

- 切片一般是字符串，不过列表也是可以切片的，从a到b, 这个b可以用负数来做索引。关于倒叙排列 a [ : : -1]，只操作最后一步“步进”
 
- isdigit()，字符串才有的方法，判断是不是数字

- `a[1:20] = [5]` 可以对列表某部分切片了进行值修改，属于在原处修改

- 复制（浅拷贝）`s[:]   or  s.copy()`

- `s.extend(t)  or  s += t` 在后面插入一组，相当于 `+=`，区别于append，append的参数是一个元素，元素可以是list，dict

- `s.insert(i, x)` 在i处插入x(可以是任意对象)

- 清空：`del s[:]   or   s.clear()`

- 删除：`s.pop([i])` 默认删除尾部元素，否则删除i处的元素(在删除索引处元素时，相比于del感觉风格要好一点)，`s.remove(x)` 删除第一个等于x的元素
`s.reverse()` 列表反向

:sunny:上面这几个操作 `pop` 有返回值，其它都没有，所以你不能把v = list.append(1) v的值为append方法的返回值None

- range快速生成列表，在3x中，返回的是迭代器(3.x中很多类似函数都返回了迭代器)，如果要步进是小数，需要自己实现，Python不提供这样的函数

2. 基本哈希存储结构——字典

| name                       | description
| -------------------------- | ------------
| len(d)                     | 返回字典元素个数 
| d[key]                     | 返回key对应的value 
| d[key]=value               | 为字典元素赋值，如果没有则增加元素 
| del d[key]                 | 删除字典元素 
| key ind/key not ind        | 查看key是否在d中 
| iter(d)                    | 返回一个迭代器，具有__next__()方法 
| clear()                    | 清空 
| copy()                     | 浅复制 
| fromkeys(seq[,value])      | 以seq作为键，value作为值建立字典，默认value为None 
| get(key[,default])         | 安全的get方法，如果不存在返回default，如果不指定default则报错 
| items()                    | 列出一个键值对的view 
| keys()                     | 列出key的view,通常用于遍历 
| values()                   | Return a new view of the dictionary's values. 
| pop(key[,default])         | 如果键值key存在与字典中，删除dict[key]，返回dict[key]的value值。key值必须给出。否则，返回default值。如果default值没有过出，就会报出KeyError异常。pop()方法至少接受一个参数，最多接受两个参数。
| popitem()                  | 弹出一个键值对，为key的哈希序列中的第一个 
| setdefault(key[,default])  | 安全的添加操作，如果存在就返回value不更改值，如果不存在添加一个key:default的表项，default默认为0 
| update([other])            | 更改操作，other可以是键值对的列表或元组（二级的），也可以是字典，用other中的键值对添加到或替换原有键值对 

字符串：`s.upper()` 小写转大写

- sort：修改原列表 
- sorted：产生一个新的列表

多条件排序，按set_name升排序，然后按cluster_type降序，cluster_type是字符串的，如果是数字的，可以 `-x['cluster_type']`

sorted(b, key=lambda x: (x['set_name'], x['cluster_type'][::-1]))

字符串处理，分割用split()，替换用replace() ，strip，lstrip, rstrip分别用于去除首位，左边，右边的指定字符串。替换可以把换行去了，str.replace('\n', '')，或者去空格。

python是没有null类型的，这个类型其它语言的，如js，sql。所以当前端传过这样的类型来需要注意，sql也是，如果一个没有赋值的字段，他可能是null的，但是判断布尔会出错，你会取得一个‘null’字符串。

python dict函数：一般用法传入关键字，其它方法：
1. dict(zip(['one', 'two', 'three'], [1, 2, 3]))   # 映射函数方式来构造字典（也是利用了传入可迭代对象的方法）
2. dict([('one', 1), ('two', 2), ('three', 3)])    # 可迭代对象方式来构造字典

`python setdefault（key, default）` 不是设置值，而是返回结果，当键不存在的时候，才会去设置值，并返回设置了的值。如果你没写default，而键不存在，则返回None，必须注意返回的值是字典的键对应的值，不是这个字典。

:sunny:切片补充：

总有人面试出一些诡异的问题，在切片中，做了以下的总结：
1. 切片超过索引不会报错
2. 关于步进：步进的值不能是0(0报错)，可以是正数或负数，符号影响切片的方向，正号从左向右切，符号相反，值代表步进多少
3. start和end，start不能大于end（这里的大于应该从切片的位置来考虑，不能看数值，即开始切片的位置不能在end后面）否则切片结果为空，注意在负步进中，也是从start切到end，不过从右边开始切
4. start和end可以是负索引，和列表操作一样，当list很长，你不知道最后的索引是多少时，可以用负索引，10为list长度，正索引范围是0到9负索引范围是-1到-11
5. 切片是不会循环的，步进决定切片方向，正负索引要从它的实际位置来做运算，字符串和tuple也可以切片

举例：

```python

    list_data = list(range(0, 10))

    print(list_data)
    print(list_data[20:30])
    # [] # 超范围为空
    print(list_data[-1:9])
    # [] 根据第3点，数值满足，但是切的位置不满足，-1相当于是索引9的位置
    print(list_data[-9:-4])
    # [1, 2, 3, 4, 5]
    print(list_data[-9:-4:-1])
    # [] 一定要注意，步进不是加负号就倒叙
    print(list_data[9:-8:-2])
    # [9, 7, 5, 3]，负步进是反向切片，所以用正步进实现不了

```

## any

内置函数，any(iterable)，可迭代序列每个元素都是False，返回False，如果其中一个是True，返回True。这可以用在对多个对象判断逻辑，把它们写在元组里面，使用any函数。

## all

内置函数，all(iterable)，可迭代的每个元素都是True，返回True，否则返回False

## 浮点数精度

以下情况就是精度损失造成的

    print(0.1 * 3) # 0.30000000000000004

为什么会有精度损失：

Python 中使用双精度浮点数来存储小数。在 Python 使用的 IEEE 754 标准（52M/11E/1S）中，8字节64位存储空间分配了52位来存储浮点数的有效数字，11位存储指数，1位存储正负号，即这是一种二进制版的科学计数法格式。虽然52位有效数字看起来很多，但麻烦之处在于，二进制小数在表示有理数时极易遇到无限循环的问题。

所以在使用float的时候，千万要小心精度，推荐金钱计算使用decimal

精度控制round函数，注意它的四舍五入比较特殊，不是单纯的记一个四舍五入，round() 如果只有一个数作为参数，不指定位数的时候，返回的是一个整数，而且是最靠近的整数（这点上类似四舍五入）。但是当出现.5的时候，两边的距离都一样，round()取靠近的偶数，这就是为什么round(2.5) = 2。当指定取舍的小数点位数的时候，一般情况也是使用四舍五入的规则，但是碰到.5的这样情况，如果要取舍的位数前的小树是奇数，则直接舍弃，如果偶数这向上取舍。(这个函数的用法非常的扯淡，不如decimal好)

{% blockquote %}
Note The behavior of round() for floats can be surprising: for example, round(2.675, 2) gives 2.67 instead of the expected 2.68. This is not a bug: it’s a result of the fact that most decimal fractions can’t be represented exactly as a float. See Floating Point Arithmetic: Issues and Limitations for more information.
根据官方的说法，是看浮点数被省略后，尽量取接近的值
{% endblockquote %}

1. 不指定小数点位数, 即取整数, 四舍五入: (取到哪一位的后面一位, 若遇到.5 奇进偶不进)
```
round(2.3)   2.0
round(2.6)   3.0
round(2.5)   2.0
round(1.5)   2.0
```
2. 指定小数点位数, 即有小数位, 四舍五入: (取到哪一位的后面一位, 若遇到.5 偶进奇不进)
```
round(2.635,2)  2.63
round(2.645,2)  2.65
round(2.655,2)  2.65
round(2.665,2)  2.67
round(2.675,2)  2.67
```

## / 和 //

6.0 / 3.0 = 2.0，6.0，3.0是浮点数，那么结果也是浮点数2.0，更精确的说，只要" / " 两边有一个数是浮点数，那么结果就是浮点数。

在Python2.2版本以前也是这么规定的，但是，Python的设计者认为这么做不符合Python简单明了的特性，于是乎就在Python2.2以及以后的版本中增加了一个算术运算符" // "来表示整数除法，返回不大于结果的一个最大的整数，而" / " 则单纯的表示浮点数除法，但是，为了折中，所有2.X版本中，也是为了向后兼容，如果要使用" // "，就必须加上一条语句：`from __future__ import division`
一看到这句，" / "就表示 浮点数除法，返回浮点结果，" // "表示整数除法。

但是，在Python3.0时，就没有这种折中情况了，" / "就一定表示`浮点数除法`，返回浮点结果，" // "表示`整数除法`。

## __slots__

在类中定义，用来限制类能被绑定的属性。需要注意一些情况：

- 正常使用，没有继承关系，除了限制的属性其它的不能被绑定

- 继承情况，分为两种，一是类定义了 `__slots__`，被其它类继承；二是类继承自其它类，父类没有定义 `__slots__`，而类自己定义了 `__slots__`。

1. 第一种情况下，类自己的slots是可以发挥作用的，但是继承这个类的子类就没有属性绑定限制了，但是子类的 `__slots__`属性是可以访问的（实例和类都可以打印 `__slots__` 子类没定义，打印父类的值，子类定义了，打印子类的值），如果想要父类的绑定限制，需要在子类中定义 `__slots = ()` 一个空元组，这样子类的属性绑定就被触发了，能被添加的属性由父类的元组和子类的元组构成，但是子类是空元组，就是需要子类来显示定义 `__slots__` 来触发熟悉绑定限制。

2. 第二种情况，一个类定义了属性绑定，但是它有继承，这个时候该类的slots是发挥不了作用的。需要父类定义 `__slots = ()` 一个空元组，子类的slots才发挥作用。

```python

    class C:
        # __slots__ = ('a',)
        __slots__ = ()

        pass

    class D(C):
        __slots__ = ('b',)

        pass

```

在应用开发中，也是很少使用slots，一般在看源代码的时候会见到

## 给类添加方法

python的 `MethodType(方法， 实例（如果是给类添加方法，就用None）， 类)` 用来给类或实例绑定方法。
例子：

```py
s = Stuedent()  
s.set_name = MethodType(set_name, s, Student)
```

## 获取项目依赖

`pip freeze > requirements.txt` 获取项目依赖。 `pip instal requirements.txt` 安装依赖。

## eval & exec

可以执行动态的代码，比如给一个字符串计算结果。区别：eval返回结果，exec不返回。这就要注意了，`exec('print 1')` 但是用eval就不行，print 不返回结果，你可以做布尔运算，这样是返回结果的。

## 闭包

```py
def  function(runfun):
    def  fun(runfun):
        runfun()
    return fun
```

记住，当函数function被调用后，就产生一个闭包，闭包是fun，自由变量是runfun。

装饰器：功能函数，原函数，使用装饰器，就是将原函数的引用指向功能函数。由于装饰器是为了给原函数增加功能，所以功能函数实现的时候需要接受一个函数的引用作为参数，这个引用即为原函数。

关于自由变量：

```py
def func1(x):
    def func2(n):
        print x ** n
    return func2
```

当我们 `a = func1(5)`，内嵌的函数得到了自由变量 `5`，虽然func1的调用结束了，但是 `5` 会被记录下来，执行 `a(2)` 得到 `25`。通过闭包得到了一个自由变量，这种是运用了嵌套作用域，如果n的值在func1中赋值了，利用的是默认参数来保留嵌套作用域的状态，在低版本中这样用，当时没有引进E的概念，n按照命名空间的原则查找。保持住状态，是闭包的最大运用（js, python 语言亦如此）记住这个保持住状态，你可能会需要这样的特性来设计程序。

这里就涉及到了python作用域查找，LGB，B是内置，新的版本引入了嵌套作用域，LEGB，查找流程是 `L→B`。

js：Javascript语言特有的"链式作用域"结构（chain scope），子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。js中用闭包就可以在外部取到函数内部的变量（我会在最外层设置一个全局变量，在这个函数里面去把结果赋值给最外层，这样函数外部可以随时取这个变量，当然有闭包，并且还有这样的需求，最好是用闭包）。

对于作用域，一个函数内变量是处在这个作用域里面的，只有函数嵌套才会涉及到作用域嵌套的问题。

变量作用域，在函数里面大家是通用的，嵌套发生在函数与函数嵌套的时候。但是有一个情况，你通过if语句决定的变量，在条件不满足的时候，就无法取到了，即这个变量的作用域在不满足条件的时候，对于整个函数不是全局的。

## 命名空间

python 命名空间，用字典来记录。局部命名空间（当前函数），全局命名空间（当前模块），内置命名空间（存放内置函数和异常），理解变量查找顺序。

Python 的一个特别之处在于其赋值操作总是在最里层的作用域。赋值不会复制数据——只是将命名绑定到对象。删除也是如此："del y" 只是从局部作用域的命名空间中删除命名 y 。事实上，所有引入新命名的操作都作用于局部作用域。理解命名空间生命周期。函数的命名空间是属于自己的，递归调用的函数也有自己的命名空间。

## 模块

导入的模块会存储在sys.modules表中，重复导入只要能在表中找到，则从内存中取出，不再去执行导入操作。

在导入这里涉及到了字节码的相关知识：做为顶层文件（被执行的，导入其它模块的文件），我们通常看不到pyc，因为顶层文件的pyc在内部使用后就被丢弃了，而属于被导入的文件会创建pyc，并且保留下来。pyo也是字节码，称为最佳化字节码文件。

不要重复导入一个模块，这样做不到覆盖模块属性，应使用 `reload（）`。不要做两个模块互相导入，如果处理不好逻辑容易出错。`python3  reload` 被移到imp中，需要 `from  imp  import  reload` 才能使用。重载后import的模块对象改变了，会对使用import的有影响，但是重载之前的from不会，因为from是复制。记住对于from，它是复制了模块的变量给了新变量（两个同名），变量已经被复制出来了，reload模块不会影响复制出来的变量。

- 模块是对象，导入后，模块将拥有属性，可以使用自省获取模块属性。对于使用from的导入，要重载，先重载对应模块，在重新执行from

- 递归导入：两个模块相互导入对方。逻辑上处理不对，极其容易发生错误，尽量避免这种导入尤其是在其中还使用from

- 单下划线开头的变量只是不能被from module import *  形式导入，from module import _变量名是可以导入的

- 程序直接执行和导入也是有区别的，第五部分7题（python学习手册4）（对于这些非常绕的概念，没有必要完全理解，实际开发运用我们是不会这样做的，还是那句话，书上讲的不要和实际混淆，例如模块名字和系统名字重复，导致导入失败，既然你都知道了，就不要取同名，也省的使用非常绕的导入方法，就为了同名，还不如重新取个名字）

理解 `from & import`

sys.path

模块的搜索路径，在程序启动时进行配置，自动将顶级文件的主目录（或者指定当前工作目录的一个空字符串）、任何PYTHONPATH目录、已创建的任何 `.pth` 文件路径的内容，标准库目录合并。得到一个目录列表。

对于sys.path 修改环境变量是永久影响的，sys.path.append()在程序结束后失效。

`import from` 是赋值语句

1. import将整个模块对象赋值给一个变量名，
2. from将一个或多个变量名赋值给另一个模块中同名的对象

理解模块对象。记住from是赋值，import是导入。在使用上仍有些许区别（只需理解from是copy就行了，理解2的意义）

- 注意：from的第一步也是普通的导入操作。因此，from总是会把整个模块导入到内存中（如果还没导入的话），无论是从这个文件中复制出多少变量名。它不是你想的只加载你需要的部分，如一个函数，虽然导入全部，但是你没指明的模块还是用不了。可能在效率上差了点，但是影响不大。
- 或者这样理解：from也是导入，先导入，然后将变量复制过来。

**变量的含义一定是由源代码中的赋值语句的位置决定的**

- 使用import导入的模块，它的全局作用域一定是其所在的文件
- 函数绝对无法看见其它函数内的变量名，除非它们从物理上处于这个函数内
- 模块程序代码绝对无法看见其它模块内的变量名，除非明确的进行了导入
- 一段程序的作用域完全由程序所处的文件中实际位置决定，作用域绝不会被函数调用或模块导入影响。（这体现了python的静态作用域，有些语言是动态作用域，作用域依赖于运行期间的调用）

命名空间嵌套：`a.py  b.py  c.py   b导入了a，  c 导入了b，  c中通过b.a可以访问到a模块`。

包：理解 `__all__`，`__init__.py` 跨目录导入的时候就应该用包导入。包文件里面可以赋值和执行代码，会在导入包的时候执行，赋值的变量也会加到命名空间里。这里和执行文件所处的位置有重要关系。

例子：dir1是一个文件夹，下面有包dir2。代码在dir1下执行，可以导入dir2包，`import dir2` 不要这样写 `import dir1.dir2`。若dir1也是一个包，代码在它的上层目录执行，就应该是 `import dir1.dir2`。这样两个包的包文件的代码会被执行，变量也可以取到。

该部分也许用到的较少，实际结合下书607页（python学习手册第四版）内容。主要是：

- 代码的执行位置
- 包文件（只有被导入的包，包文件才被执行，变量才能加到命名空间，如果代码本身处在一个包下，那么这个包的包文件因为你不导入所以不被执行）
- 跨文件：有些时候想从另一个文件中拿一个模块来，此时代码所处的是文件或者包亦无所谓，你跨的目录就得是包，一级一级得包结构下去，而且这个包的上一级目录必须在`sys.path` 中，注意这句话，这个包的上级目录必须在 `sys.path `中，只能到上级目录，把自身（包）加进去就找不到了。A下有 `a，b`，b想拿a，`sys.path` 的路径就要到A，到A/a不可以。若路径只到A的上一级，应该是 `import A.b` Ab都要是包。

注意：sys.path 路径在搜索的时候，是在这个列表上从左到右来进行的。

包相对导入和绝对导入：

绝对就是完整路径，从sys.path路径开始搜索。p2.6默认是相对，p3默认是绝对。这样带来问题，`import  module`，p3默认使用绝对路径，如果是系统模块，则同目录下的包导入失败。

## 模块导入

顶层语句直接被执行，定义的函数将被创建，函数的引用写入内存，变量被创建命名空间，python会检查代码并填充局部命名空间。在python运行那行代码之前，就发现了对变量的赋值，并把它添加到局部命名空间中。

在函数中做这样的语句 `i = i +1`，它的本质为：python 是静态作用域，程序在编译的时候就已经知道了变量作用范围。（在代码中给一个变量赋值的地方决定了这个变量将存在于哪个命名空间，也就是它可见的范围）比如定义函数：
```py
def  func2():
    i = i +1
```
这是一个函数，它被定义了，函数所处的模块被导入，或者自身被执行的时候，就要引入这个函数到内存，并且创建它的命名空间，所以说它是静态作用域，等到做func2()，即执行函数的时候，解释器会知道i是静态作用域。这里 `i = i + 1`，是对 i 进行赋值，记住 i 是被赋值的，所以不是查找变量，i 被加入函数自己的局部作用域，后面的 `i + 1` 是执行语句，函数在创建命名空间时，只管创建，还不执行，等到执行的时候，计算 `i + 1`，因为i 是局部的，该函数没有i 的值，所有会报错。记住命名空间先创建，执行的时候去找变量。
```py
i = 1
def  func2():
    a = i +1
```    
这样是对的, 执行的时候，`i` 在局部没找到，去找全局

但是如果是这样：
```py
i = 1
    def  func2()
    a = i +1
    i = a
```
`i` 会被当成局部变量，导致执行 `a = i+1` 时出错。 `i` 必须要声明才能用，要在函数里面取得外部的i, 可以在开始指定 `global  i`。

## 动态类型与对象引用

变量类型的概念只存在对象中。例如 `a = 3`。创建一个对象来表示 `3`，这个对象由值3和头信息组成，头信息是一个指向 `int` 的对象的指针。它的步骤应该是这样的：

- 创建一个变量a，如果它还没创建的话。
- 进行引用，a变量的引用是对象3。

深刻理解 `变量，对象，引用`
- 变量：变量是一个系统表的元素，拥有指向对象的链接空间
- 对象：对象是分配的一块内存，有足够的空间去表示它们所代表的值
- 引用：是自动形成的从变量到对象的指针
- 引用计数器：对象拥有引用计数器

理解共享引用，两个对象都指向同一个对象

对于赋值，每次都是创建新的引用，但是列表支持在原处修改（修改某个元素，不会创建新的对象，把变量赋值为3，则原来指向列表对象，想再指向整形对象，此时创建的新对象），原处修改的，如果想不改变原来的值，可以进行对象拷贝。

加深：a=3, b=a也是实现了共享引用，和列表一样，但是它不支持在原处修改。

## 多态

python 依赖于类型的行为称为多态。为对象来编写接口，而不是为数据类型来编写接口，注意多态的弊端，如果你的接口某个运算只能使用整形，传字符串就出错了，但是多态的灵活性是一种实实在在的好处，是否需要判断类型，可做权衡。一般说python是动态类型语言。

## 作用域

每次对函数调用都会创建一个新的本地作用域。

理解 `LEGB` 作用域，E中闭包的运用（2.2中加入的功能），这是一种高级的技术，是函数式编程的运用。为了取得函数内部的参数，实现一个闭包，在外部来调用，js中的讲解。

嵌套作用域的变量在嵌套的函数被调用的时候才进行查找，在循环中，会以最后循环的值为准。使用默认参数，避开嵌套作用域查找。

分析一个函数的时候，先分析变量的赋值，看其所处的作用域。即先看这个变量会处于什么
作用域，再去分析代码。

```python

    def func1(parm1):
        parm1 = parm1
        def func2(s):
            print parm1, s
        return func2
    # 这样是正确的

    def func1(parm1):
        parm1 = parm1
        def func2(s):
            print parm1, s
            parm1 = parm1 + 1
        return func2
    # 这样就出错了，以为函数第一，先创建，第二对赋值的变量创建作用域（静态作用域），
    # print 语句等函数执行的时候才去执行，次数需要的参数parm1是局部的（因为下一句赋值的
    # 关系），所以程序出错。

```

python3  nonlocal ：

在python2 中只有 `global` 被声明的变量会将作用域变成全局。`nonlocal` 的作用：如果在嵌套函数中赋值，该变量会被判定为局部作用域，如果不先赋值就会出错，如果外层有同名变量，在不改变内层变量名的情况下（内层变量名和外层一样），想引用外层的变量，只有声明变量是 `nonlocal`。其实改个变量名也可以解决事情。

```python

    def func1(parm1):
        parm1 = parm1
        def func2(s):
            parm2 = parm1
            parm2 = parm2 + 'ss'
            print parm2
        return func2

```

若要 `parm1 = parm1` ,要先 `nonlocal  param1`。局部作用域里的代码可以读外部作用域（包括全局作用域）里的变量，但不能更改它。一旦进行更改，就会将其当成是局部变量。而如果在更改前又进行了读取操作，则会抛出异常。这里的改变，即赋值，给想变的变量赋值（是不行的，只能使用关键字，如果外部有个x，将x进行 `parm = x`，是读取了x的值，后面用parm，不属于改变）

如果上面的例子不好理解nonlocal，可以这样理解：

```python

    def mufunc():
        a = 123
        def func():
            a = a(标记1)+1

```

在这个函数里面，对于 `func a` 根据上面所学，应该知道这里a是声明，那么a的作用域被确定，当然因为没有赋值变量就引用，是要报错的。为了让a（标记1）可以拿到上层函数的a，就可以使用关键字：

```python

    def mufunc():
        a = 123
        def func():
            nonlocal a
            a = a(标记1)+1
            # a = 123+1

```

## 函数的参数

- 理解位置参数和关键字参数，针对调用 `*` 和 `**` 接收参数时的使用（编写函数）
- 调用的时候，传递参数也可以用 `*` 和 `**`，python 将进行参数解包
- `function（1，2，a，b=2）`，`*` 得到 `1，2，a`，`**` 得到 `b=2`

keyword-only(python3)在函数定义的时候，跟在 `*`args，或直接用 `*`，后面的参数在调用时，必须用关键字的形式。
```py
def dog(name, host, *, age):
    print(name, host, age)
func(1,2,b=2)
```
理解函数参数是通过赋值传递到函数中，赋值的方式是通过对象引用。

## 链式编程

链式编程 是将多个操作（多行代码）通过点号"."链接在一起成为一句代码。 例如：`a.v.fun( )`

## 函数注释 Function Annotations

python3 新增加的特性，可以对函数增加变量注释，以及返回值注释，然后可以通过 `__annotations__` 获取到注释信息的字典。

注意：使用该特性并不会对参数以及返回值做限制，lambda不支持函数注释

例子：

```python

    def func(a: 'x', b: 5 + 6, c: list, *args: 'xx', **kwargs: float) -> max(2, 9):
        pass

    print(func.__annotations__)
    # {'a': 'x', 'b': 11, 'c': <class 'list'>, 'args': 'xx', 'kwargs': <class 'float'>, 'return': 9}
    
```

## lambda
也称为匿名函数（因为没有函数名）

特性：是一个表达式，而不是一个语句；主体是一个单个的表达式，而不是一个代码块，只能编写简单的函数，不能用像if这样的语句（但可以写if关联的表达式）。由于是表达式，可以出现在列表中，def不能。一个表达式，得到的是函数的引用。

## 文件流与二进制流

`stringIO  BytesIO`，文本流的文件（txt,world,excel）才用StringIO，二进制流（视频，图片）用BytesIO

## 动态载入模块

可能会有需求根据需要载入不同的模块，利用多态的特性进行程序设计

1. exec()函数
```py
modelname = 'string'
exec('import'+modelname)
```
2. 或者利用 `__import__` （推荐）
```py
modelname = 'string'
string = __import__(modelname)
```

## 处理json

json是按照一定格式的字符串，json模块有 `loads，dumps，load，dump`

后两个是文件相关的，前两个是直接转换成字符串的。

dumps将基本类型转换成json格式字符串，转换后的类型是str，需要注意的是，dumps的参数里面如果包含对象，就不是基本类型，无法进行转换，转换的对象必须是serializable(可序列化)的，不然会报类型错误。这时候需要设置参数 `json.dumps(mydata, default=changefun)` 接受一个函数的引用，这个函数用来处理不能直接转换的类型。函数示例：
```PY
def changefun(obj):
    if hasattr(obj, 'display'):
        return obj.display
```

## 面向对象

面向对象相关内容

### 委托

委托的概念，就是你想访问A类的时候，通过B类来访问，一般实现在B类中创建一个私有属性，指向A的实例。

class ：class语句是对象的创建者并且是一个隐含的赋值运算---执行时，它会产生类对象，并把其引用值存储在前面所使用的变量名。就是说，只有代码跑到这里，或者是导入class语句才发挥作用。

注意：一个对象的属性查找顺序遵循首先查找实例对象自己，然后是类，接着是类的父类。如果有一个类变量叫“data”, 实例也有一个变量同名，那么通过self.data去找到的只是实例的，当实例没有这个变量，才会找到类变量。一般类变量访问通过类名来获取。

加深：类本身拥有自己的类变量（保存在内存），当一个TestClass类的对象被构造时，会将当前类变量拷贝一份给这个对象，当前类变量的值是多少，这个对象拷贝得到的类变量的值就是多少；而且，通过对象来修改类变量，并不会影响其他对象的类变量的值，因为大家都有各自的副本，更不会影响类本身所拥有的那个类变量的值；只有类自己才能改变类本身拥有的类变量的值。这也是python的特别，不同于C++的静态变量。

类变量是不是共享？当我们创建的实例，类变量会被复制一份给实例，各个实例用的是自己的，要修改类变量，只能用类去访问，不能用实例去访问，实例访问的是自己的类变量（由类复制一份给它的）。所以实例操作类变量，是操作自己的类变量，影响不了其它实例，要真正修改类变量，还用用类本身去访问。

理解隐式调用：方法第一个参数总是接收方法调用的隐性主体，也就是实例对象。python会自动把实例方法的调用对应到类方法函数：
instance.method(args...)           class.method(instance, args...)

对于类来说，写在def语句外的都是类变量，想在方法中引用这个变量，需要加上类名。如果变量在class之外，像函数查找变量一样来操作。

类的构造函数可以有多个，但是只会使用最后一个

在python中，当对对象进行点号运算时，就会发生继承，而且涉及了搜索属性定义树（一个或多个命名空间）

类接口技术常用的技巧：超类，继承，重写，扩展，提供（提供者模式）

| name      | description
| --------- | ----------          
| Super     | 定义一个method函数以及在子类中期待一个动作delegate 
| Inheritor | 没有提供任何新的变量名，因此会获得Super中定义的一切内容 
| Replacer  | 用自己的模版覆盖Super的method 
| Extender  | 覆盖并回调默认method，从而定制Super的method 
| Provider  | 实现Super的delegate方法预期的action方法 

研究这些子类来了解它们定制的共同的超类的不同的途径，下面就是这个文件：

```python

class Super:
    def method(self):
        print('in Super.method') # Defaulr behavior

    def delegate(self):
        self.action()            # Expected to be definde


class Inheritor(Super):          # Inherit method verbatim
    pass


class Replacer(Super):           # Replace method completely
    def method(self):
        print('in Replacer.method')


class Extender(Super):
    def method(self):
        print('staring Extender.method')
        Super.method(self)
        print('ending Extender.method')


class Provider(Super):           # Fill in a required method
    def action(self):
        print('in Provider.action')

```

关于抽象超类：超类Super中定义了一个函数test。调用了自身的action函数。但是Super中并没有定义action函数。这个超类也会称为抽象超类。意思是说，类的部分行为由子类来提供。

委托，拦截了对实例属性的访问。即在当前类里面，访问委托类的属性，都会被拦截。

```python
class Wrapper(object):
    def __init__(self, object):
        self.wrapper = object
        
    def __getattr__(self, attname):
        print 'do some thing'
        return getattr(self.wrapper, attname)
```

属性名称：对象命名空间

点号的属性名指的是特定对象定属性，并且遵循模块和类定规则。就类和实例对象而言，引用规则增加了继承搜索这个流程。

赋值语句（object.X = value）

在进行点号运算的对象的命名空间内创建或修改属性名X，并没有其他作用。继承树的搜索只发生只属性引用时，而不是属性的赋值运算时。

引用（object.X）

就基于类的对象而言，会在对象内搜索属性名X，然后是其上所有可读取的类（使用继承搜索流程）

对于目前类来说，结合着函数部分的内容，理解作用域是关键，赋值和引用也是关键，看到一条语句要知道它是赋值还是引用，将大大加深你对程序的理解。

加深：作用域总是由赋值语句的位置决定。这里引申一下python3 关键字 nolocal 可以让变量查找作用域到达外层，这样就可以取函数外层的变量了，但不是将其设置为全局（global）。一定要理解赋值和引用。

```
def  A():
    x = 'a'
    def B():
        x += 1
```

在这个例子里面，内层的x还没赋值就参与了运算，是不对的，一般像 y = x + 1是把外层的x拿来，是不能改变x的，要改变，是对变量操作，所以变量要先赋值。所以要用global ，或者nolocal 。还是那句话，这些例子和概念是为了理解LEGB，实际开发我们没必要设置一样的变量名。

命名空间字典：

模块和类一样，它们的命名空间是以字典的形式来实现的。属性点号的运算其实内部是字典的索引运算，而属性继承其实就是搜索链接的字典。

内省的方法对函数也适用

模块和类：模块是数据or逻辑包，通过编写python or C 实现，通过导入来使用。类，由class语句创建，通过调用来使用，总是位于一个模块中。

抽象超类：

抽象类是会调用方法的类，但没有继承或定义该方法，而是期待该方法由子类填补。当行为无法预测，非得等到更为具体的子类编写时才知道，通常可以用这种方式把类通用化。OOP软件框架也使用这种方式作为客户端定义，可定制的运算的实现方法。

通过在子类中重新定义方法，然后再调用超类的方法，可以给这个方法增加功能，而不是完全覆盖。

python 方法的绑定和未绑定：一般情况，我们都是直接调用方法的。方法是对象，可以获取方法对象而不调用。实例去获取的时候，方法被绑定到实例，得到新对象加上括号就可以调用了。而用类去获取，得到的就是无绑定方法对象，要执行需要把实例作为第一个参数传入。注意，在python3中，无绑定方法将变成函数，它不再是方法对象。显示类型测试程序可能受到影响，如果你打印一个非实例的类方法，它在Python2.6中显示“无绑定方法（unbound method）”，在Python3.0中显示“函数”（function）

python 扩展内置类型：一般情况，通过继承内置类型，重载运算符，但是最后还是要调用超类的方法。最好有这种需要的时候多查阅下资料。

### 继承搜索机制

只在新式类中，继承搜索是从左到右，广度优先。

py2继承object的才是新式类，py3写不写object都是新式类

经典类py2中：

```python

class P1:
    def foo(self):
        print('p1-foo')


class P2:
    def foo(self):
        print('p2-foo')

    def bar(self):
        print('p2-bar')


class C1(P1, P2):
    pass


class C2(P1, P2):
    def bar(self):
        print('C2-bar')


class D(C1, C2):
    pass

```

新式类：

```python

class P1:
    def foo(self):
        print('p1-foo')


class P2:
    def foo(self):
        print('p2-foo')

    def bar(self):
        print('p2-bar')


class C1(P1, P2):
    pass


class C2(P1, P2):
    def bar(self):
        print('C2-bar')


class D(C1, C2):
    pass

```

1. 经典类
```py
    d = D()
    d.foo() # 输出 p1-foo 
    d.bar() # 输出 p2-bar 
```
- 实例d调用foo()时，搜索顺序是 `D => C1 => P1`
- 实例d调用bar()时，搜索顺序是 `D => C1 => P1 => P2`
换句话说，经典类的搜索方式是按照“从左至右，深度优先”的方式去查找属性。d先查找自身是否有foo方法，没有则查找最近的父类C1里是否有该方法，如果没有则继续向上查找，直到在P1中找到该方法，查找结束。

2. 新式类
```py
    d=D() 
    d.foo() # 输出 p1-foo 
    d.bar() # 输出 c2-bar 
```
- 实例d调用foo()时，搜索顺序是 `D => C1 => C2 => P1`
- 实例d调用bar()时，搜索顺序是 `D => C1 => C2`
可以看出，新式类的搜索方式是采用“广度优先”的方式去查找属性。

现在是新式类，使用钻石查找规则，在一个父类里面，如果执行了self.method()，这时将从头开始查找一遍。新类是广度优先，A继承了B，C。找了B，然后找C，再找B的父类。如果是旧类，是深度优先，A继承了B，C，B里面找不到就去找B的父类，一直向上，找不到才来C里面找。

### 描述符

Python支付属性描述符的概念---带有 `__get__` 和 `__set__` 方法的类，分配给类属性并且由实例继承，这拦截了对特定属性的读取和写入访问。描述符在某种意义上是特性的一种更加通用的形式。实际上，特性是定义特定类型描述符的一种简化方式，该描述符运行关于访问的函数。描述符还用来实现slots特性

注意区分描述符状态和实例状态，描述符自身也是一个类，它有自己的实例，已经调用描述符的实例，所以有两个实例，要注意区分。

描述符结合 `__slots__` 使用

### 装饰器

装饰器是利用语法糖@把原来繁琐的方法简化了，装饰器可以分为以下几种：
1. 普通装饰器
2. 被装饰对象带参数
3. 装饰器带参数（被装饰对象同时也可以带参数）
4. 基于类的装饰器
5. 带参数的类装饰器
6. 对类作用装饰器
7. 装饰器之上的装饰器

对于1，2，3种情况，都是在前面的基础上进行扩展，是比较常见的情况，为了适应函数不确定的参数，推荐使用可变参数来设计装饰器：

```python

def get_run_time(show_time=False):
    def ff(f):
        def _func(*args, **kwargs):
            import time
            import datetime
            if show_time:
                print(datetime.datetime.now())
            a = time.time()
            f(*args, **kwargs)
            b = time.time()
            print((b - a), type(a))

        return _func

    return ff


@get_run_time(True)
def func(*args, **kwargs):
    print('hello')
    print(args, kwargs)


# func = get_run_time(True)(func) # 不使用语法糖的情况
# func(a=12, b=15)

func(a=1, b=2)

```

装饰器也可以是类，由于装饰函数要能callable(因为函数就是能call的，所以类要考虑这个问题)，不然会报错 `object is not callable` 此时类需要实现 `__call__`，实例如下（混合了装饰器之上的装饰器，就是把上个装饰器后的结果（一个对象的引用）作为装饰函数的参数传入）：

```python

def get_run_time(debug):
    """
    获取运行时间
    """
    flag = 'able' if debug else 'disable'
    print(flag)

    def f(func):
        def _func(*args, **kwargs):
            import time
            start = time.time()
            func(*args, **kwargs)
            end = time.time()
            run_time = end - start
            print(run_time)

        return _func

    return f


class Logging:
    """
    类装饰器
    """

    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print('logging call')
        self.func()


@Logging
@get_run_time(debug=False)
def my_func():
    _list = [i for i in range(30 * 300)]
    print('this is my_func')


# my_func = Logging(get_run_time(False)(my_func)) # 不使用语法糖的情况
my_func()

```

下面来看带参数的类装饰器：

```python

def get_run_time(debug):
    """
    获取运行时间
    """
    flag = 'able' if debug else 'disable'
    print(flag)

    def f(func):
        def _func(*args, **kwargs):
            import time
            start = time.time()
            func(*args, **kwargs)
            end = time.time()
            run_time = end - start
            print('run_time: {}'.format(run_time))

        return _func

    return f


class LoggingConfig:
    """
    带参数的类装饰器
    """

    def __init__(self, level='info'):
        self.level = level

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            print(self.level)
            func(*args, **kwargs)

        return wrapper


@LoggingConfig('waring')
@get_run_time(debug=False)
def my_func():
    _list = [i for i in range(30 * 300)]
    print('this is my_func')


# my_func = LoggingConfig()(get_run_time(False)(my_func)) # 不使用语法糖的情况
my_func()

```

带参数的类装饰器情况还是比较特别的，之所以要这么写，理解 `LoggingConfig()(get_run_time(False)(my_func))` 就很明了了，类的参数要在类实例化的时候传入，所以把参数写在类 `__init__` 方法内，然后类需要实现`__call__` 方法，这个时候就是类的 `__call__` 要接收一个函数的引用，就是被装饰函数对象的引用。需要注意`@LoggingConfig()` 的括号不能去掉，不然 `__init__` 方法出错。

来看最后一种情况，对类作用装饰器，这个时候可以把类也当成一个对象的引用，类似函数对象的引用，特别注意类要实例化，所以设计装饰器的时候，最后的返回很重要，大致会有两种情况：

1. 返回类对象的引用，这种情况下，一般是利用装饰器对类进行类属性的操作，比如利用 `hasattr`判断类是否有特定属性，然后执行一些逻辑，最后返回类对象的引用：

```python

def as_view(need_login=False):
    def _auth(view_clz):
        if need_login:
            setattr(view_clz, 'need_login', True)
        return view_clz

    return _auth


@as_view() # 即使不传参数也不能省去括号，否则as_view(need_login=False) 的need_login就指向A
class A(object):
    a = 1

    def __init__(self, x=0):
        self.x = x
        if hasattr(A, 'need_login'):
            print('need_login is {}'.format(getattr(A, 'need_login')))
        else:
            print('A not has need_login')


a = A(2)
print(a.x)

```

2. 返回类实例，这种情况，最内侧是返回一个函数对象的引用，这个函数对象的引用现在指向类，当我们实例化类的时候，就会调用这个函数对象的内容，而这个内容就是返回一个类的实例：

```python

def as_view(login=False):
    print('as_view is {}'.format(login))

    def _wrapper(cls):
        print('cls is {}'.format(cls))

        def _instance(*args, **kwargs):
            # setattr(cls, 'need_login', True) # 如果打开这一句，你会发现A仍然没有属性need_login
            print('param is {}--{}'.format(args, kwargs))
            return cls(*args, **kwargs)
            # return cls 不能在这里返回类对象的引用，因为上一层返回的是_instance，是一个对象的引用

        return _instance

    return _wrapper


@as_view()
class A(object):
    a = 1

    def __init__(self, x=0):
        self.x = x
        if hasattr(A, 'need_login'):
            print('need_login is {}'.format(getattr(A, 'need_login')))
        else:
            print('A not has need_login')


print(A.__name__) # 此时A已经指向_instance，即使你在上面添加了属性need_login，这个属性也不是A的，这是装饰器需要注意的地方，这对于情况一是不存在的，情况一是把对象的引用进行属性添加，返回的也是这个对象的引用，所以情况1 A.__name__ 仍然是 A。
a = A(2)
print(a.x)

# 结果：
# as_view is False
# cls is <class '__main__.A'>
# _instance
# param is (2,)--{}
# A not has need_login
# 2

```

一般在应用开发中，会使用情况一对类做一些属性限制，非常好用。

- 函数装饰器，类装饰器：@staticmethod  类似这样的形式，其中staticmethod 也称为元函数，运用于类的称为类装饰器。装饰器在编写的时候，我们只要知道，装饰器是接受被装饰对象为参数，并将其重新赋值给装饰对象，依照此基础就可以来写装饰器。相对于类装饰器，更像是对类进行扩展，把类在装饰函数里面执行操作后（一般是添加属性等），然后返回本身。

- 装饰器的理念是对原函数、对象的加强，相当于重新封装，所以一般装饰器函数都被命名为wrapper()，意义在于包装。函数只有在被调用时才会发挥其作用。比如@logging装饰器可以在函数执行时额外输出日志，@cache装饰过的函数可以缓存计算结果等等。

- 而注解和特性则是对目标函数或对象添加一些属性，相当于将其分类。这些属性可以通过反射拿到，在程序运行时对不同的特性函数或对象加以干预。比如带有Setup的函数就当成准备步骤执行，或者找到所有带有TestMethod的函数依次执行等等。

:green_apple:**装饰器陷阱：**

使用装饰器需要注意两个地方，最好不要在装饰函数之外添加逻辑功能；使用了装饰器，函数签名会被改变。

1. 第一种情况，如下实例代码：

```python

def html_tags(tag_name):
    print('begin outer function.')

    def wrapper_(func):
        print("begin of inner wrapper function.")

        def wrapper(*args, **kwargs):
            content = func(*args, **kwargs)
            print("<{tag}>{content}</{tag}>".format(tag=tag_name, content=content))

        print('end of inner wrapper function.')
        return wrapper

    print('end of outer function')
    return wrapper_


# @html_tags('b')
def hello(name='Toby'):
    return 'Hello {}!'.format(name)


hello = html_tags('b')(hello)

hello(name='Toby')
hello(name='Toby')

```

上述代码结果：

    begin outer function.
    end of outer function
    begin of inner wrapper function.
    end of inner wrapper function.
    <b>Hello Toby!</b>
    <b>Hello Toby!</b>

因为装饰器在定义后就会执行了，因为Python的代码在定义的时候就会确定作用域，变量等，所以 `'end of inner wrapper function.'` 之上的代码在对hello使用装饰器后就会打印了。调用hello两次，只会执行装饰器函数里面的代码。

即：`'print("<{tag}>{content}</{tag}>".format(tag=tag_name, content=content))'` 所以代码结果是这样的。

所以最好不要在装饰器函数之外添加逻辑功能，这里的逻辑功能是print，如果是一些复杂的逻辑，`它只会在定义的时候执行一次`。

2. 获取被装饰函数的函数签名，文档字符串等返回的将是闭包的函数信息。

```python

    def html_tags(tag_name):
        print('begin outer function.')

        def wrapper_(func):
            print("begin of inner wrapper function.")

            def wrapper(*args, **kwargs):
                """
                doc string
                """
                content = func(*args, **kwargs)
                print("<{tag}>{content}</{tag}>".format(tag=tag_name, content=content))

            print('end of inner wrapper function.')
            return wrapper

        print('end of outer function')
        return wrapper_

    # @html_tags('b')
    def hello(name='Toby'):
        """
        hello doc string 
        """
        return 'Hello {}!'.format(name)


    hello = html_tags('b')(hello)

    hello(name='Toby')
    hello(name='Toby')

    import inspect

    doc = inspect.getdoc(hello)

    print(hello.__name__, doc) # wrapper doc string

```

原因就是因为使用@语法糖，原函数的引用被修改了：`hello = html_tags('b')(hello)`，那么新的hello的函数信息指向 `wrapper`。这是最容易发生问题的地方，尤其是使用多次装饰器，如果你需要这样的复杂逻辑，一定要小心对象的引用早已改变，如果还使用原对象名称进行操作，无疑是出BUG。

如果需要原函数的信息，可以使用标准库 `functool.wrapt`，对闭包的函数进行装饰：

```python

    import functools

    def html_tags(tag_name):
        print('begin outer function.')

        def wrapper_(func):
            print("begin of inner wrapper function.")

            @functools.wraps(func) # 在此运用装饰器
            def wrapper(*args, **kwargs):
                """
                doc string
                """
                content = func(*args, **kwargs)
                print("<{tag}>{content}</{tag}>".format(tag=tag_name, content=content))

            print('end of inner wrapper function.')
            return wrapper

        print('end of outer function')
        return wrapper_


    # @html_tags('b')
    def hello(name='Toby'):
        """
        hello doc string
        """
        return 'Hello {}!'.format(name)


    hello = html_tags('b')(hello)

    hello(name='Toby')
    hello(name='Toby')

    import inspect

    doc = inspect.getdoc(hello)

    print(hello.__name__, doc)

    print(inspect.signature(hello)) # py3之前的版本仍然获取到闭包函数信息

    print(inspect.getsource(hello)) # py3之前的版本仍然获取到闭包函数信息

```

输出如下：

    begin outer function.
    end of outer function
    begin of inner wrapper function.
    end of inner wrapper function.
    <b>Hello Toby!</b>
    <b>Hello Toby!</b>
    hello hello doc string
    (name='Toby')
    def hello(name='Toby'):
        """
        hello doc string
        """
        return 'Hello {}!'.format(name)

除了functool.wrapt模块，还可以使用第三方模块wrapt，如果使用该模块，要按照给定的规则来重新包装函数，和functool.wrapt直接使用装饰器相比需要改动代码，另外Python3.x之前functool.wrapt模块并不能得到函数签名和函数源码等，在3.x之后变得可行了。

### 利用 __dict__ 做缓存

实例方法可以利用装饰器变成属性，其实质是，方法本身也是类的一个属性，通过装饰器，让访问这个属性的时候传回方法计算后期待的值，实现了将方法变成属性。

实例和类都有 `__dict__` 属性，写一个装饰器，让方法变成属性可以直接访问，同时将方法作为属性添加到实例的 `__dict__` 中去，这样下次再访问这个属性方法的时候，根据属性查找规则，会先去实例里面找，因为实例已经添加了方法的属性键值对，所以直接取到了，不会再去类里面调用方法来计算属性了，从而实现了缓存（在运用的时候，代码运行起来，就每次都是去取缓存了，必须要停下来才去重新计算，所以如果你想要用一些动态技术来生成属性的值，记得把 `__dict__` 里面的原属性删除，这样你的方法才会被再次调用，重新做缓存）

以下的例子中，hello只会被执行一次，因为下次从实例里面取，不再调用类的方法了。

```python

    from cached_property import cached_property

    class A(object):
        acs = '11'
        patt = property()

        def __init__(self, z):
            self.z = z
            self.info = None

        @cached_property
        def exinfo(self):
            print('hello')
            if type(self.info) == dict:
                res = self.info
            else:
                res = {}
                self.info = res
            return res

        def mydata(self):
            return 'zxc'

    a = A('ss')
    print(a.__dict__, A.__dict__)
    print(a.exinfo, a.__dict__, A.__dict__)
    print(a.exinfo, a.__dict__, A.__dict__)

```

### 避免循环递归

```python

    def __getattribute__(self, item):
        return object.__getattribute__(self, 'other')  

```

利用超类，传递的实例是本对象的。理解方法调用的时候传递的self很重要，你可以改实例，不使用隐式传参。

对于赋值和删除也需要考虑这个问题，用超类来避免循环。或者用 `self.__dict__['other'] = other`，这个对于取属性不能用，因为取属性就要循环了。总结：取属性要特别注意，获取和删除不能直接对实例的属性进行赋值，要用超类的setattr方法或者实例的 `__dict__` 来赋值。

## 逻辑表达式

和一般语言返回布尔值不同，python更加灵活：

在Python中，空字符串为假，非空字符串为真。非零的数为真。

数字和字符串之间、字符串之间的逻辑操作规律是：
1. 对于and操作符：
只要左边的表达式为真，整个表达式返回的值是右边表达式的值，否则，返回左边表达式的值
2. 对于or操作符：
只要两边的表达式为真，整个表达式的结果是左边表达式的值。

- 如果是一真一假，返回真值表达式的值
- 如果两个都是假，比如空值和0，返回的是右边的值。（空值或0）

## !/usr/bin/env python

脚本语言的第一行，目的就是指出，你想要你的这个文件中的代码用什么可执行程序去运行它，就这么简单。

- #!/usr/bin/python是告诉操作系统执行这个脚本的时候，调用/usr/bin下的python解释器
- #!/usr/bin/env python这种用法是为了防止操作系统用户没有将python装在默认的/usr/bin路径里。当系统看到这一行的时候，首先会到env设置里查找python的安装路径，再调用对应路径下的解释器程序完成操作
- #!/usr/bin/python相当于写死了python路径
- #!/usr/bin/env python会去环境设置寻找python目录,推荐这种写法

env是Linux命令，可以启动python。env命令用于显示系统中已存在的环境变量，以及在定义的环境中执行指令。

## 断言

类似C的概念。`assert group is None, "msg"` 其中group是一个变量，如果断言表达式正确，则代码继续，错误则除非断言异常，`AssertionError` 会把信息打印出来。

## 对象拷贝

浅拷贝和深拷贝的加深理解：因为没有这样的需要，所以一直没有注意。再来看看两个点的概念
- 浅拷贝(copy)：拷贝父对象，不会拷贝对象的内部的子对象。
- 深拷贝(deepcopy)： copy 模块的 deepcopy 方法，完全拷贝了父对象及其子对象。

要完全获得一个对象的复制，用深拷贝（深拷贝比较费时间，可能会影响效率），浅拷贝不会拷贝对象内部的子对象，这句话很重要，如果你要拷贝的对象是一个列表，并且列表每一个元素都是一个不可变的，那么浅拷贝效果和深拷贝一样。但是如果你的元素是一个可变对象，比如一个列表，那么原值和新值的这个列表对象都是指向同一个引用的，改其中一个另一个结果也会影响，但是如果是不可变对象，改变其中一个不影响另一个，因为这个不可变对象的拷贝已经是新的引用。

解析
1. b = a: 赋值引用，a 和 b 都指向同一个对象。

![image](/images/Python/python-copy-1.png)

2. b = a.copy(): 浅拷贝, a 和 b 是一个独立的对象，但他们的子对象还是指向统一对象（是引用）。

![image](/images/Python/python-copy-2.png)

3. b = copy.deepcopy(a): 深度拷贝, a 和 b 完全拷贝了父对象及其子对象，两者是完全独立的。

![image](/images/Python/python-copy-3.png)

## 垃圾回收

除了利用引用计数回收外，还有一个垃圾回收机制，gc(garbage collection)。gc这种机制很多语言都会用，但是python兼顾二者，引用计数回收能在对象计数为0的时候就马上回收。但是引用计数会出现循环引用的情况，只有容器类型对象会出现循环引用，不可变对象不会。例如：

```
def myfunc():
    l = []
    l.append(l)
```

这个操作会循环引用对象I，对于循环引用的情况，利用引用计数来回收就不行了，所以出现了gc机制。这个机制是自动的，可以认为触发。垃圾回收机制会根据内存的分配和释放情况的而被调用，比如分配内存的次数减去释放内存的次数大于某一个阈值的时候。

## Python 常见文件格式

Python源代码遵循 GPL (GNU General Public License) 协议，由 Guido van Rossum 于 1989 年底发明，第一个公开发行版发行于 1991 年。

Python 常被称为胶水语言，能把用其他语言编写的各模块 (尤其是 C/C++) 轻松地联结在一起。常见情形是，用 Python 快速生成程序原型 (有时甚至是程序最终界面)，然后对其中有特别要求的部分，用更合适的语言改写；譬如：3D 游戏中的图形渲染模块，性能要求特别高，就可用 C/C++ 重写，而后封装为 Python 可调用的扩展类库。需要注意的是，在您使用扩展类库时可能需要考虑平台问题，某些扩展类库可能不提供跨平台实现。

一般认为，Python 是一种解释性语言，Python 在执行时，会先将 .py 文件中的源代码编译成  byte code (字节码)，然后再由 Python Virtual Machine 来执行这些编译 byte code。

这种机制的基本思想跟 Java、.NET 一致；但 Python Virtual Machine 与 Java 或 .NET 的 Virtual Machine 不同的是：Python 的 Virtual Machine 是一种更高级的 Virtual Machine。这里的高级并不是通常意义上的高级，不是说 Python 的 Virtual Machine 比 Java 或 .NET 的功能更强大，更拽，而是说和 Java 或 .NET 相比，Python 的 Virtual Machine 距真实机器的距离更远。或者可以这么说，Python 的 Virtual Machine 是一种抽象层次更高的 Virtual Machine。

Python 提供了一种中间编译结果保存机制，即 byte code，或更准确地说，保存 PyCodeObject。事实上，Python 确实提供了这样一种机制 —— .pyc 文件。在执行一个 .py 文件的源代码之后，Python 并不会自动生成与该 .py 文件对应的 .pyc 文件。Python 会根据需要自动触发、创建 .pyc 文件，原理很简单，就是利用 Python 的 import 机制。

在 Python 运行过程中如碰到 import abc 这样的语句，Python 将会在设定好的 path 中寻找 abc.pyc 或 abc.dll 文件。若没有这样的文件，而只是发现了 abc.py 文件，那么，Python 会首先将 abc.py 编译成相应的 PyCodeObject 中间结果，然后再创建 abc.pyc 文件，并将中间结果写入该文件。接下来，Python 才会对 abc.pyc 文件进行一个 import 动作，实际上也就是将 abc.pyc 文件中的 PyCodeObject 重新在内存中复制出来，加以运行。

由于 Python .pyc 文件的此种运行机制，所以，可随时删除所有中间过程 .pyc 文件，当 Python 再次运行时，还会重生成这些 .pyc 文件，对您所编辑程序的运行并无任何影响。当然，得有源代码才行。

视窗用 python.exe 运行 .py ，用 pythonw.exe 运行 .pyw。因为安装视窗版 Python 时，扩展名 .py 自动被登记为用 python.exe 运行， 而 .pyw 则被登记为用 pythonw.exe 运行。 

- `.py 文件`
以 .py 作扩展名的文件是 Python 源代码文件，由 python.exe 解释，可在控制台下运行。

当然，也可用文本编辑器或其它专用 Python IDE (集成开发环境) 工具进行修改。

- `.pyc 文件`
以 .pyc 作扩展名的文件是 python 编译文件。.pyc 文件是不能直接用文本编辑器进行编辑，其优点是 .pyc 文件的执行速度要远快于 .py 文件。

至于为什么要有 .pyc 文件，因为 .py 文件是可直接看到源码的。若是软件开发商，不可能把源码泄漏出去？所以，就需编译成 .pyc 后再发布。

但 .pyc 文件只是经简单编译，并未加密，因此，有些工具还能反编译它得出源代码。

- `.pyw 文件`
.pyc 文件执行时，桌面会出现类似 CMD 命令的黑色 shell 窗口，十分难看，于是 .pyw 文件就应运而生了。.pyw 文件与 .pyc 文件的执行，本质上并没什么区别，只是 .pyw 文件执行的时候不会出现类似 CMD 命令的黑色 shell 窗口。

.pyw 文件格式主要是设计用来运行 Python 纯 GUI (图形用户界面) 程序的。纯 GUI (图形用户界面) 程序的用户不需要看到类似 CMD 命令的黑色 shell 控制台窗口。当 .pyw 文件运行时，所有 stdout、stderr 输出无效，所有原 stdin 的读取只会得到 EOF。

值得一提的是，开发纯 GUI (图形用户界面) 程序时，可暂时把 .pyw 改成 .py，以便运行时调出控制台窗口，看到所有错误信息，方便修改、调试。

注意：采用 PyQt、PySide 等 GUI (图形用户界面) 开发框架开发的程序，源文件仍可采用 .py，无需单独使用 .pyw。

- `.pyo 文件`
"python -O 源文件" 即可将 Python 源程序编译成 .pyo 文件，但有时仍得采用 .pyc 作后缀，才能正常运行。

.pyo 文件是相对 .pyc 而言的，优化编译后的 Python 文件。 同样，.pyo 文件也不能直接用文本编辑器进行编辑。

- `.pyd 文件`
.pyd 文件是非 Python，由其它编程语言 "编写-编译" 生成的 Python 扩展模块。

Python 要导入 .pyd 文件，实际上是在 .pyd 文件中封装了一个 module。在 python 中使用时，把它当成 module 来用就可以了，即："import 路径名.modulename" 即可，路径名为 .pyd 文件所在的路径。

基于 Qt/C++ 的 PyQt、PySide GUI (图形用户界面) 开发框架绑定，会为每个 Qt/C++ 库 .dll 文件生成一个中间 .pyd 文件。PyWin32 项目分发包中，也存在很多 MicroSoft Windows OS 相应 .dll 文件的中间 .pyd 文件。

Cython 可将个人基于 Python 语言编写的 Python 模块编译成具有 C 语言特性的 .pyd 文件。

.pyd 文件有时也是用 D 语言按照一定格式编写，编译生成的二进制文件。

那么什么是 "D 语言" 呢？它是 C/C++ 的综合进化版，不仅具有二者的全部优点，且整体性能更佳，但其抽象程度高。D 语言最初由 Digital Mars 公司就职的 Walter Bright 于 2001 年发布，意图改进 C++ 语言。目前最新 D 语言被简称为 D2。最主要的 D 语言实现是 DMD。

D 语言源自 C/C++，借鉴了众多编程语言的特色和现代编译器技术，融会贯通了设计者丰富的实践经验，使之具备了非凡的威力 ── 既有 C/C++ 语言的强大威力，又有 Python 和 Ruby 的开发效率。它集众多系统级编程所需的功能于一身，例如垃圾回收、手工内存操作、契约式设计、高级模板技术、内嵌汇编、内置单元测试、Mixin 风格多继承、类 Java 包管理机制、内置同步机制、内建基本运行时信息。

## 文本与二进制

在 Python 2 中，文本类型（也就是 unicode）和二进制类型（也就是 str）的边界非常模糊。很多函数的参数既可以是文本，也可以是二进制。

但是在 Python 3 中，文本类型和二进制类型的字符串被完全的区分开了。Python 3对文本和二进制数据作了更为清晰的区分。

文本总是Unicode，由str类型表示，二进制数据则由bytes类型表示。

不能拼接字符串和字节包，也无法在字节包里搜索字符串（反之亦然），也不能将字符串传入参数为字节包的函数（反之亦然）。

## 位运算

a = 0011 1100

b = 0000 1101

| name | Description | Example
| ---  | :---------: | :------
| &    | 按位与运算符  | 参与运算的两个值,如果两个相应位都为1,则该位的结果为1,否则为0	(a&b)输出结果12，二进制解释：00001100 
|  {% raw %}|{% endraw %}  | 按位或运算符  | 只要对应的二个二进位有一个为1时，结果位就为1。	(a{% raw %}|{% endraw %}b)输出结果61，二进制解释：00111101 
| ^    | 按位异或运算符 | 当两对应的二进位相异时，结果为1	(a^b)输出结果49，二进制解释：00110001 
| ~    | 按位取反运算符 | 对数据的每个二进制位取反,即把1变为0,把0变为1。~x类似于-x-1	(~a)输出结果-61，二进制解释：11000011，在一个有符号二进制数的补码形式。 
| <<   | 左移动运算符  | 运算数的各二进位全部左移若干位，由"<<"右边的数指定移动的位数，高位丢弃，低位补0。	a<<2输出结果240，二进制解释：11110000 
| >>   | 右移动运算符  | 把">>"左边的运算数的各二进位全部右移若干位，">>"右边的数指定移动的位数	a>>2输出结果15，二进制解释：00001111 

关于异或：

异或是一种基于二进制的位运算，用符号XOR或者 ^ 表示，其运算法则是对运算符两侧数的每一个二进制位，同值取0，异值取1。它与布尔运算的区别在于，当运算符两侧均为1时，布尔运算的结果为1，异或运算的结果为0。

简单理解就是不进位加法，如1+1=0，,0+0=0,1+0=1。

性质

1、交换律 可任意交换运算因子的位置，结果不变

2、结合律（即(a^b)^c == a^(b^c)）

3、对于任何数x，都有x^x=0，x^0=x，同自己求异或为0，同0求异或为自己

4、自反性 A ^ B ^ B = A ^ 0 = A ，连续和同一个因子做异或运算，最终结果为自己

异或运算，只看运算方法，会觉得很奇怪，但是由方法引申出一些性质，再把这些性质作为公式或口诀熟记，异或才能真正去解决问题。

比如：

1. 两个变量交换
```
a = a^b

b = a^b

a = a^b
```

2. 判断奇偶数更简单高效的做法
奇数二进制的最低位一定是1，偶数二进制的最低位一定是0，所以拿一个数和1作异或就可以判断奇偶性

3. 可以去除一些数组中重复的数，利用4个特性

## ASCII

`ord()` 函数接受一个字符，转换成ASII码。`chr()` 接受一个整型ASII变量，转换成字符

## logging 模块

logging模块很灵活，是项目必备的模块，日志级别 `CRITICAL > ERROR > WARNING > INFO > DEBUG`，默认logger的level是logging.WARNING，低于该级别的就不输出了。

在导入模块后，通过logging.basicConfig(level=logging.NOTSET)来配置日志级别，这里设置为NOTSET，所有级别都会被输出（貌似和设置DEBUG级别是一样的效果，感觉作用在于重写类对象，新增加级别的时候设置NOTSET可以不受控制）

## mappingproxy（不可变映射类型）

python3.3开始,types模块中引入了一个封装类名叫MappingProxyType，如果给这个类一个映射,它会返回一个只对映射视图。虽然是个只读的视图,但是它是动态的,这意味着如果对原映射做出了改动，我们可以通过这个视图观察到,但是无法通过这个视图对原映射做出修改。

```python

    #示例
    from types import MappingProxyType
    #创建一个集合
    index_a = {'a' : 'b'}
    #创建index_a的映射视图
    a_proxy = MappingProxyType(index_a)
    print(a_proxy)
    a_proxy['a']
    # #不能对a_proxy视图进行修改
    # a_proxy['b'] = 'bb'
    #但是可以对原映射进行修改
    index_a['b'] = 'bb'
    print(a_proxy)

```

另外值得注意的是，类的 `__dict__` 属性也是mappingproxy的，之所以这么做是为了保证类级别的属性和方法只能是字符串，也帮助解释器更快的查找类级别的属性（因为它是字符串的）。类的属性是实例共享的，这也保证了类的统一性。所以你不应该设计这样的程序：通过`class.__dict__.update(dict()STATUS=0)`去修改类的属性，可以做的是修改实例的属性，记住创建实例会把类的属性复制一份给实例。

## 利用偏函数设计程序

偏函数是将所要承载的函数作为partial()函数的第一个参数，`原函数的各个参数依次作为partial()函数后续的参数`，除非使用关键字参数。

```python

    from functools import partial
    
    def mod( n, m ):
        return n % m
    
    mod_by_100 = partial( mod, 100 ) 
    
    print mod( 100, 7 )  # 2
    print mod_by_100( 7 )  # 2  mod(100, 7) 原函数的各个参数依次作为partial()函数后续的参数

```

进制转换

```python

    from functools import partial
    
    bin2dec = partial( int, base=2 )
    print bin2dec( '0b10001' )  # 17
    print bin2dec( '10001' )  # 17
    
    hex2dec = partial( int, base=16 )
    print hex2dec( '0x67' )  # 103
    print hex2dec( '67' )  # 103

```

## random模块

random.random() 生成1到0之间的随机数，可以到小数点后好几位。

random.uniform(10, 20) 可以指定随机数范围，生成的结果仍然是带小数的。

通过指定seed，可以控制随机数的生成。random.seed(time.time())，在调用随机数生成之前指定相同的seed，参数为时间戳，时间戳也必须是一样的，这样多次调用random得到的随机数就是一样的了。

random.randint(1, 10) 指定范围，并且生成的随机数为整数。

random.randrange(1, 20, 3) 限定范围，指定step，相当于从[1, 4, 7, 10, 13, 16, 19]中取一个随机数。

random.choice(sequence) 从序列中取一个，random.choices(sequence, k=3) 取三个。

random.sample(sequence, 3) 从序列中取样3个元素，不会取重复的，而choice可能会选到重复的。

## 标准库itertools进行分组

```python
from itertools import groupby
_list = []
_list.sort(key=lambda x: (x['a'], x['b']))
a = groupby(_list, key=lambda x: (x['a'], x['b']))
```

## sys.exit() os._exit()

os._exit()会直接将python程序终止，之后的所有代码都不会继续执行。
sys.exit()会引发一个异常：SystemExit，如果这个异常没有被捕获，那么python解释器将会退出。
如果有捕获此异常的代码，那么这些代码还是会执行。捕获这个异常可以做一些额外的清理工作。0为正常退出，其他数值（1-127）为不正常，可抛异常事件供捕获。

## fcntl

在 linux 环境下用 Python 进行项目开发过程中经常会遇到多个进程对同一个文件进行读写问题，而此时就要对文件进行加锁控制，在 Python 的 linux 版本下有个 fcntl 模块可以方便的对文件进行加、解锁控制。

```py
import fcntl
file_path = "/home/ubuntu/aaa.json"
f = open(file_path, 'w')
fcntl.flock(f.fileno(), fcntl.LOCK_EX)	# 加锁，其它进程对文件操作则不能成功
f.write("something")
fcntl.flock(f.fileno(), fcntl.LOCK_UN)	# 解锁
f.close()
```

fcntl.flock(f.fileno(), operation)	operation 的操作包括以下选项：

fcntl.LOCK_EX

排他锁：	除加锁进程外其他进程没有对已加锁文件读写访问权限

fcntl.LOCK_UN

解锁：	    对加锁文件进行解锁

fcntl.LOCK_SH

共享锁：	所有进程都没有写权限，即使加锁进程也没有，但所有进程都有读权限

fcntl.LOCK_NB

非阻塞锁：	如果指定此参数，函数不能获得文件锁就立即返回，否则，函数会等待获得文件锁。

LOCK_NB可以同LOCK_SH或LOCK_NB进行按位或（|）运算操作。

fcnt.flock(f.fileno(),fcntl.LOCK_EX|fcntl.LOCK_NB)

## 文件操作模式

r	以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。
rb	以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。
r+	打开一个文件用于读写。文件指针将会放在文件的开头。
rb+	以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。
w	打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
wb	以二进制格式打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
w+	打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
wb+	以二进制格式打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。
a	打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
ab	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
a+	打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
ab+	以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。

