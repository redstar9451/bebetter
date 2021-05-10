# Python decorate

平时写decorate时，经常从其他decorate拷贝过来修改，对于其原理、语法、类型等并未做过深入的研究。多少有些知其然而不知其所以然，因此抽时间系统的总结了一下。

## 背景知识

1. 在python中，函数定义是object类型的对象。对象有如下特性：

   1. 可以作为函数参数传递
   2. 可以赋值给变量，并通过变量进行调用
   3. 可以作为函数的返回值

   这部分概念比较简单，不写示例代码了

2. python支持在函数内定义函数，且inner函数可以访问outer函数中的局部变量

   ```python
   def func_outer(message_arg):
       message_local_var = "local variable in outer function"
       def func_inner():
           print(message_arg)
           print(message_local_var)
       func_inner()
   func_outer("argument variable passed to outer function")
   
   # output
   """
   argument variable passed to outer function
   local variable in outer function
   """
   ```

## 不通过语法糖实现decorator

上述两点是python实现decorator的原理，使用decorator时的@符是语法糖，语法糖就是简便写法的意思。不要纠结这个词。

在介绍decorator的语法糖语法前，咱们先看一下不使用@符如何实现装饰器

```python
# 原始函数
def func_original():
    print("print in function_original")
    
# 在不改动原函数的情况下，通过”装饰器“增加新的功能
def decorator_append_message(func):
    def wrapper():  # 背景特性2，在函数内可以定义函数
        print("before print in wrapper")
        ret = func()  # 背景特性2，内部函数可以访问外部函数中的局部变量
        print("after print in wrapper")
        return ret
    return wrapper  # 背景特性1，函数是Object对象，可以作为函数返回值

func_original = decorator_append_message(func_original)  # 背景特性1，func_original保存wrapper函数对象

func_original()  # 调用decorator包装后的函数
”“”
执行结果
before print in wrapper
print in function_original
after print in wrapper
“”“
```

**这里的关键点在于decorator_append_message返回的是一个包装后的函数对象，还涉及不到func_original的参数。**传递参数发生在调用函数时，即func_original()时才涉及到传递参数。因此func_original有参数时，仅需将wrapper函数增加参数即可。

```python
def func_original(arg1, arg2="arg2"):
    print(f"print in function_original {arg1} {arg2}")

def decorator_append_message(func):
    def wrapper(*args, **kwargs):
        print("before print in wrapper")
        ret = func(*args, **kwargs)
        print("after print in wrapper")
        return ret
    return wrapper

func_original = decorator_append_message(func_original)

func_original("arg1", "arg2")  # 调用decorator包装后的函数
```

## 如何理解decorator的语法糖

我们还是以前面的例子来深入，如果使用语法糖的话如何使用decorator呢？

```python
def decorator_append_message(func):
    def wrapper(*args, **kwargs):
        print("before print in wrapper")
        ret = func(*args, **kwargs)
        print("after print in wrapper")
        return ret
    return wrapper

@decorator_append_message
def func_original(arg1, arg2="arg2"):
    print(f"print in function_original {arg1} {arg2}")
    
func_original("arg1", "arg2")  # 调用decorator包装后的函数
```

可以看出语法糖只是简化了 func_original = decorator_append_message(func_original) 这句而已。然而语法糖真的这么好理解吗？那看看下面两种写法

```python
@decorate_func
def abc():
  pass

@decorate_func
def abc_2(args, **kwargs):
  pass

@decorate_func_with_args(arg=1)
def abc_3():
  pass

@decorate_class
class CAbc(object):
 	pass
```

你一定见过前三种写法，前两种是不带参数的装饰器，第三种是带参数的装饰器。先让我们定义两个decorator，一个带参数，一个不带参数

```python
def decorator_maker_with_arguments(decorator_arg1, decorator_arg2, decorator_arg3):
    def decorator(func):
        def wrapper(function_arg1, function_arg2, function_arg3):
            # do something based on decorator_argx
            return func(function_arg1, function_arg2,function_arg3)
        return wrapper
    return decorator

def decorator(func):
    def wrapper(function_arg1, function_arg2, function_arg3) :
        return func(function_arg1, function_arg2,function_arg3)
    return wrapper
```

可以看出，带参数的装饰器是把不带参数的装饰器又包了一层，同时在wrapper函数中根据参数可以做一些逻辑，即”# do something based on decorator_argx“注释部分可以填写具体的代码，这些代码可以使用decorator_arg1等参数。这是第一个区别。

第二个区别是带参数的装饰器返回的是decorator()函数，而不带参数的装饰器返回的是wrapper()函数。

第三个区别是在使用上的区别，带参数的装饰器语法糖是@decorator_maker_with_arguments(1, 2, 3), 而不带参数的装饰器语法糖是@decorator

看到这里有木有顿悟到什么呢？

装饰器的语法糖从语法角度来说是统一的，其统一格式如下：

@callable_object

name

可调用对象就是装饰器本身，name就是装饰的符号，object本身，语法糖的本质是在**import阶段**，执行如下代码

name = callable_object(name)

总结一下：

1. 装饰器的语法糖是在import阶段执行的，执行套路就是name = callable_object(name)进行名称替换
2. callable对象后加()，表示调用该调用对象

```python
@decorator_maker_with_arguments(1, 2, 3)
def func(args, **kwargs):
	pass
# 语法糖等同于 func = decorator_maker_with_arguments(1, 2, 3)(func)

@decorator
def func(args, **kwargs):
	pass
# 语法糖等同于 func = decorator(func)
```

现在明白为什么带参数的装饰器要再包装一层了吧，因为带参数的装饰器要先调用装饰器函数本身，然后返回一个装饰器函数。

了解了这个统一的模型，对于各种装饰器的理解就统一了，想忘也忘不掉了。

## 装饰器的类型

装饰器的类型划分可以有多种维度，第一个维度是带参数的装饰器和不带参数的装饰器，第二个维度是根据装饰对象的不同划分为函数装饰器，类装饰器，类函数装饰器。

### 从是否带参数的维度

带参数的装饰器和不带参数的装饰器

```shell
def decorator_maker_with_arguments(decorator_arg1, decorator_arg2, decorator_arg3):
    def decorator(func):
        def wrapper(*args, **kwargs):
            # do something based on decorator_argx or not
            ret = func(*args, **kwargs)
            # do something based on decorator_argx or not
            return ret
        return wrapper
    return decorator

def decorator(func):
    def wrapper(*args, **kwargs) :
    		# do something
        ret = func(*args, **kwargs)
        # do something
    return wrapper
```

### 从装饰场景的维度

1. 装饰普通函数

   即上一个章节的两个装饰器即可

2. 装饰类成员函数

   类成员函数只是永远有参数，且第一个参数为self，除此之外，与装饰普通函数并无太大区别，不过也可以”刻意“将self函数标出来。为什么有时候要把self从args, **kwargs中提出来，单独定义呢？ 如果装饰器要使用被装饰对象的属性时，使用self可读性会更高一些。

   ```python
   def decorator_maker_with_arguments(decorator_arg1, decorator_arg2, decorator_arg3):
       def decorator(func):
           def wrapper(self, *args, **kwargs):
               # do something based on decorator_argx or not
               ret = func(self, *args, **kwargs)
               # do something based on decorator_argx or not
               return ret
           return wrapper
       return decorator
   
   def decorator(func):
       def wrapper(self, *args, **kwargs) :
       		# do something
           ret = func(self, *args, **kwargs)
           # do something
       return wrapper
   ```

3. 装饰类

   ```python
   def decorator_for_class(cl):
       def new_init(self, *args, **kwargs):
           self.name = "new class"
       cl.__init__ = new_init
       return cl
   
   @decorator_for_class
   class CDecoratorTest(object):
       def __init__(self):
           self.name = "class"
   
   if __name__ == '__main__':
       obj = CDecoratorTest()
       print(obj.name)
       
   # output new class
   ```

### 另外的维度

从装饰器语法糖本身来说，仅要求装饰器是一个callable对象，name是一个object即可。因此装饰器也可以是一个自定义了\_\_call__函数的类，只是将装饰器实现放到\_\_call\_\_函数中即可。

```python
class MyDecorator:
    def __init__(self, function):
        self.function = function

    def __call__(self):
        print("Inside Function Call")
        self.function()
 
@MyDecorator
def function():
    print("GeeksforGeeks")
```



## 装饰器在调试、troubleshooting中的问题

装饰器的本质是返回一个新的对象，并保存在原对象的名称中。因此使用装饰器后，原函数的metadata(\_\_name\_\_等)都会被替换为装饰器函数，比如下面的代码片段

```python
import logging
format = '%(asctime)s - %(name)s - %(funcName)-12s %(levelname)s - %(message)s'
logging.basicConfig(filename='example.log', format=format, level=logging.DEBUG)
logger = logging.getLogger(__name__)

def decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@decorator
def test_decorator():
    logger.error("This is log text")
    raise Exception("This is an exception")

if __name__ == '__main__':
    print(test_decorator.__name__)
    help(test_decorator)
    test_decorator()
"""
输出结果的前半段：
wrapper
Help on function wrapper in module __main__:

wrapper(*args, **kwargs)
"""
```

可以看到test_decorator的metadata都被修改了，我们再看看日志中的函数名和exception打印

```shell
# 日志中的函数名未被替换
2021-05-10 19:50:14,595 - __main__ - test_decorator ERROR - This is log text
# 异常中的函数名也未替换
Traceback (most recent call last):
  File "/Users/redstar/workspace/learning/sqlalchemy-learn/decorate.py", line 23, in <module>
    test_decorator()
  File "/Users/redstar/workspace/learning/sqlalchemy-learn/decorate.py", line 12, in wrapper
    return func(*args, **kwargs)  # 调用栈中出现func
  File "/Users/redstar/workspace/learning/sqlalchemy-learn/decorate.py", line 18, in test_decorator
    raise Exception("This is an exception")  # 但是因为有文件名和行号，找到抛出异常的位置还是比较容易的。
Exception: This is an exception
```

从测试情况来看，logging库显然专门对装饰器进行了处理，异常明显是没有处理的，但是因为有文件名和行号，找到异常原因并不困难。看起来比较有影响的只是help(test_decorator)了，完全看不到有效信息。

这种情况通过functool.wrap装饰器解决，这个装饰器会将test_decorator的metadata复制给wrapper函数，使用方法如下：

```python
def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

使用functools.wraps后的效果

```shell
# 输出前半段
test_decorator
Help on function test_decorator in module __main__:

test_decorator()

# exception打印
Traceback (most recent call last):
  File "/Users/redstar/workspace/learning/sqlalchemy-learn/decorate.py", line 25, in <module>
    test_decorator()
  File "/Users/redstar/workspace/learning/sqlalchemy-learn/decorate.py", line 14, in wrapper
    return func(*args, **kwargs)
  File "/Users/redstar/workspace/learning/sqlalchemy-learn/decorate.py", line 20, in test_decorator
    raise Exception("This is an exception")
Exception: This is an exception
```

由于exception要打印实际的调用栈，因此exception时并未涉及函数metadata。但是help()函数可发现wrapper的metadata已经替换为test_decorator的metadata。

## 最佳实践

最佳实践一定是看过，做过很多实践后才能总结出来的，可是我见过，写过的装饰器并不多。先把仅有的rules列在这里

1. 永远都使用functool.wraps
2. 在一般情况下，装饰器透传被装饰对象的所有参数，即使用args, kwargs透明传递，不要根据函数参数写逻辑
3. 如果一定要，那么尽量避免主动定义self参数

## 参考资料

https://www.datacamp.com/community/tutorials/decorators-python?utm_source=adwords_ppc&utm_campaignid=1455363063&utm_adgroupid=65083631748&utm_device=c&utm_keyword=&utm_matchtype=b&utm_network=g&utm_adpostion=&utm_creative=278443377095&utm_targetid=dsa-429603003980&utm_loc_interest_ms=&utm_loc_physical_ms=9061376&gclid=EAIaIQobChMI1aHrnZO-8AIVUUFgCh1m5g1gEAAYASAAEgLK7_D_BwE

https://www.geeksforgeeks.org/decorators-in-python/

https://zhuanlan.zhihu.com/p/78500405
