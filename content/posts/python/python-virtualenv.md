---
weight: 5500
title: "python-virtualenv 虚拟环境使用总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python-virtualenv 虚拟环境使用总结"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

python 虚拟环境总结，总有一款适合你

使用虚拟环境是很有必要的，另外在windows和Linux上还是有区别的，windows上的支持要差些，我用过的虚拟环境太多了，和IDE的配合也很熟练了，总结下来虚拟环境还是只能用来创建运行环境，至于有些宣称的包管理工具，还是有很多欠缺的地方，pip才是最强的！ :fire:

<!-- more -->

## Linux python virtualenv

大致和windows上是一样的，安装virtualenv

virtualenv -p 解释器目录 virtual目录

ls /usr/bin/python*  一般现在的linux都有python3，在这里指定解释器就可以了，没有就装一个

source py34env/bin/activate  激活

deactivate  退出    

windows上的虚拟环境还是不能直接在linux上用，虽然可以跑，但是创建这个虚拟环境的时候是指定了解释器的，直接跑解释器不对，包的内容和版本也不对，所以还是在linux上创建虚拟环境吧。

## virtualenvwrapper 

virtualenvwrapper 是对virtualenv的扩展，利用它管理虚拟环境，最好的特征就是直接用命令就可以进入虚拟环境，不用像原来一样需要切换到目录下进入。[简书](https://www.jianshu.com/p/9f47a9801329)

基本命令

workon   查看虚拟环境

workon  环境名称  进入对应虚拟环境

deactivate 退出环境

export WORKON_HOME=/  环境安装路径

export VIRTUALENVWRAPPER_PATHON= /  python解释器路径

source  /  virtualenvwrapper.sh  路径

我的流程： pip 安装  virtualenvwrapper  virtualenv  配置  ~/.bash_profile 为上面内容  把virtualenv添加符号链接

source ~/.bash_profile(激活环境变量，让workon命令可以被执行到，仅本次登陆有效)

## Anaconda

该软件有新的包管理工具 conda 这个不仅是python，是一个其它语言也可以用的包管理工具。用conda命令安装的python包，会去寻找相关依赖，提示你需要安装依赖，并一起安装。而pip虽然也会连同依赖一起装（听说没有conda好），但是有些包不会（猜测依赖的机制是包自身的，有些第三方包没有做这个处理，导致你安装了后，运行报错还要去装其它的，个人猜测）。

`conda env export > environment.yaml` 命令导出当前虚拟环境，可以用这个文件恢复虚拟环境。这个文件中有一个pip相关的信息，记录了该环境用pip安装的包。    

虽然用了conda，但是还是有一些包没法安装，还得用pip安装（猜测是一些个人写的包，不出名，没在conda上记录，或者就是单纯的没有记录）conda 和 pip ，conda可以管理pip和自己安装的包（用conda list查看），pip好像不行，只能管理自己的。

关于安装的时候是否选择添加到path，如果你电脑已经有了python，就不要选了。选了这个直接在命令行输入python，就会使用Anaconda的虚拟环境。

在新的虚拟环境中执行 `pip install -r requirements.txt` 导入pip安装的包

- activate 环境名称：进入对应环境    
- conda env list：列出当前环境
- mac下进入环境，前面加 source

心得：Anaconda也用了一段时间，感觉并没有网上说的那么强大，对于一些科学计算，或者说是由其它语言编写，Python来调用的那种包，对，就是那种很高端的，是可以用conda安装管理的，但是像一些小包，尤其是纯Python写的，只有pip才能安装，这样你还是摆脱不了pip，重点软件非常大，太大了，比较适合做开发用，数据分析方向用。

## pipenv

又是一个新的虚拟环境工具，相比virtualenv功能上更加强大，由于是较新的工具，设计上考虑就比较全面。通过pip install来安装，在项目对应目录执行 `pipenv install --dev` 使用系统Python版本来为此项目创建虚拟环境，生成Pipfile，Pipfile.lock文件。使用 `pipenv python 3.6` 的形式指定版本。通过帮助可以查看更多命令。

Pipfile & Pipfile.lock：

Pipfile是用来代替原来的requirements.txt的，source部分用来设置仓库地址，packages部分用来指定项目依赖的包，dev-packages部分用来指定开发环境需要的包，这样分开便于管理。而Pipfile.lock中记录了当前环境中安装的依赖的版本号以及哈希，以保证每次装出来的依赖都是一致的。

`pipenv install --dev` 用来安装当前项目中dev-packages中的包，没有环境的会创建虚拟环境。

在 Dockerfile 中安装依赖，加--system参数表示使用 pip 直接安装相应依赖，不创建虚拟环境。

`RUN pipenv install --deploy --system`

`pipenv install --dev django`，`pipenv install django` 安装django包，第一种安装在dev-packages里面，这样在部署的时候通过 `pipenv install` 安装，只会安装packages里面的，把开发环境的包过滤了，这很有用，要全部安装，应该 `pipenv insyall --dev`

`pipenv --venv` 来查看环境所在目录

在项目目录中编写 `.env`，可以在进入虚拟环境后，把env文件中的环境变量加载，这个就很有用了

关于IDE支持：pycharm中，最新版本已经支持了，然而我用的mac版试了还是不行，而且我看源码的执行文件，明确说明不能直接执行此文件，这就很尴尬了，从使用virtualenv的经验来看，配置操作是没问题的，而且我查了官方文档，发现官方的情况和我的不符合。我推测是Windows环境的版本才支持，可能mac版还不行（但是软件中是有配置项，这就很尴尬了，暂时使用常规虚拟环境配置吧）。

无敌的环境还是有bug，在安装amqp的时候，生成Pipfile文件是叫pyamqp，而且我看有些书上也是叫pyamqp的，不过官方的包库中，找不到pyamqp，这导致了锁定版本的时候出了问题，估计是改了名字pipenv没有反应过来。

>关于Pipfile的问题，在我查阅了很多资料后，发现这就是个弟弟，并没有完全实现作者的构思的蓝图，因为你只要有一个包有问题，操作就不是全自动的，乖乖用pip来安装吧

## autoenv

通俗讲，就是用来在进入目录时，执行`.env`里面的命令，这样可以实现进入目录就自动进入虚拟环境。

1. git clone git://github.com/kennethreitz/autoenv.git ~/.autoenv  克隆项目到用户目录
2. echo 'source ~/.autoenv/activate.sh' >> ~/.zshrc  写入用户环境变量
3. 在项目下创建.env文件，并写入命令，如果是用pipenv，那么命令为pipenv shell

## pip 导出当前项目依赖

1. 导出
pip freeze > requirements.txt

2. 安装
pip install -r requirements.txt