# pyenv and pipenv管理python版本

问题：

1. 不同的项目使用不同的python解释器版本，比如python2.7, python3.6.5
2. 即使使用相同的python版本，使用的库、库版本也会不同

## 管理python版本与库版本的方法

### python解释器版本

1. macos下可以使用brew安装不同的python版本，缺点是并非所有的python版本都可以安装。而且brew当前正在使用的python版本有可能因为安装软件而导致升级。比如brew install fpp时，bew会把python版本升级到3.X。
2. 手工编译，太麻烦了
3. pyenv。就是设计用来管理python解释器版本的，强烈推荐

### 项目虚拟python环境

不同项目使用不同的virtualenv，python virtualenv加上pip freeze > requirements.txt一般就足够了。但是pipenv更好用。pipenv会在统一的位置保存virtualenv，使用起来更方便，依赖关系更清晰。

## pyenv与pipenv的区别

pyenv是python解释器版本管理工具，使用pyenv可以方便快捷的安装、卸载不同版本的python解释器。

pipenv是virtualenv的管理工具，为不同的项目创建不同的virtualenv环境。

这里举一个典型的场景，pipenv shell --python 3.6.5可以指定使用3.6.5版本的解释器。但是pipenv不能安装python3.6.5版本，能使用的前提是已经通过其他方式安装好了python3.6.5. pyenv就是用来安装python3.6.5的。

## 如何使用

pyenv、pipenv的安装与使用请参考官方文档即可。本文记录在macos big sur 11.2.3版本下的坑。

无论是在本地运行，还是为了在pycharm中编写、阅读代码（库没安装或者版本不对时，使用IDE的代码检查或者自动提示会遇到问题），本质上就是创建出virtualenv即可。这句“本质上”如何理解呢？virtualenv的本质就是通过PATH环境变量实现的。因此只要创建出virtualenv后，pycharm或项目都可以直接使用。不限制如何创建virtualenv的方式。

1. 安装python3.6.5

部分版本在macos下安装需要打patch。python3.6.5安装方式如下：

```shell
pyenv install --patch 3.6.5 < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch\?full_index\=1)
```

2. 为项目启用pyenv安装的版本

```shell
cd xxx
python local 3.6.5
```

3. 启用pipenv

```shell
# 如果pipenv是安装在pyenv环境中，则直接pipenv shell即可
# 如果pipenv是安装在system python环境中，则需指定python版本
pipenv shell --python 3.6.5
```

### 常用命令

pyenv install --list : 显示可安装的python版本

pyenv local x.x.x : 为目录启用某个python版本，进入此目录自动切换为启用的python版本

pyenv versions ： 显示已安装的python版本，并显示global系统版本



pipenv shell --python x.x.x : 创建pipenv虚拟环境

pipenv --venv ： 显示当前目录使用的virtualenv

pipenv install ： 安装Pipfile中的软件包

pipenv install xxx ： 安装xxx软件包





