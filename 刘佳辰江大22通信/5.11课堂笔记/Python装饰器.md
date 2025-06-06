# Python装饰器

在 Python 中，**装饰器**是一个非常强大的功能，它允许你在不修改函数代码的情况下，动态地修改函数或方法的行为。装饰器本质上是一个函数，它接受一个函数作为参数并返回一个新的函数。

简单来讲：让其他函数在不需要做任何代码变动的前提下，增加额外的功能（开放封闭原则），装饰器的返回值也是一个函数对象。 装饰器的应用场景：比如插入日志，性能测试，事务处理，缓存等等场景。

## 案例切入

已知有一个函数`func1()`，作用是输出一句话。现在我们想要给他增加额外的功能。但是为了保障已有功能的稳定，不能更改原函数。

```python
def func1():
    print("in func1")

# 新的需求，能够打印如下内容...
# hello world
# in func1
# hello python
```

可以想想应该怎么做？

让我们一起研究一下：

```python
# 实现
def func2(func):
    def inner():
        print("hello world")
        func()
        print("hello python")
    return inner

func1 = func2(func1)
print(func1) #<function func2.<locals>.inner at 0x00000260EBE19760>
func1() #inner()

# Output:
hello world
in func1
hello pythons
```

## 装饰器形成的过程

如果我想**测试某个函数**的执行时间

```python
import time

def func1():
    print('in func1')

def timer(func):
    def inner():
        start = time.time() #返回值时间戳
        func()
        print(time.time() - start)  #打印函数执行的时间
    return inner

func1 = timer(func1)  # 将函数本身做为参数传递进去
func1()
```

这个时候，如果有很多个函数都需要测试他们的执行时间，岂不是每次都需要`func1 = timer(func1)`？这样是比较麻烦的，而且不利于代码的可读性和后期维护。这里我们可以使用python中的一种特殊的语法结构`语法糖`，来更为简便的使用装饰器。

我们将上述代码修改如下：

```python
import time

def timer(func):
    def inner():
        start = time.time()
        func()
        print(time.time() - start)
    return inner

@timer
# 当我们在某个函数上方使用@my_decorator的时候，python会自动将下面定义的函数做为参数传递给my_decorator。
# 等价于func1 = timer(func1)
def func1():
    time.sleep(1)
    print('in func1')

func1()
```

## 装饰带参数的函数

装饰一个带参数的函数与装饰一个不带参数的函数类似，但需要在装饰器中处理传递给被装饰函数的参数。

**示例：**

```python
import time

def timer(func):
    def inner(a):
        start = time.time()
        func(a)
        print(time.time() - start)
    return inner

@timer
def func1(a):  
    time.sleep(1)
    print(a)

func1('hello world')
```

## 装饰带多个参数的函数

这里我们利用了函数里面的动态参数进行传参

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        # 打印传入的参数
        print(f"调用 {func.__name__} 函数，参数: {args}, 关键字参数: {kwargs}")

        # 调用原始函数并获取结果
        result = func(*args, **kwargs)

        # 打印返回结果
        print(f"{func.__name__} 函数返回: {result}")

        return result
    return wrapper

@my_decorator
def add(x, y):
    """返回两个数的和"""
    return x + y

# 测试
result = add(5, 3)
print(f"最终结果: {result}")
```

# wraps装饰器

回到我们最开始的案例

```python
import time

def func1():
    print('in func1')

def timer(func):
    def inner():
        start = time.time()
        func()
        print(time.time() - start)
    return inner

func1 = timer(func1)  # 将函数本身做为参数传递进去
func1()
```

思考一个问题：这里虽然我们最后还是执行`func1`函数，但是这里的`func1`函数还是我们最初的`func1`函数吗？

......

我们先来看一下最后的`func1`他的函数名

```python
import time

def func1():
    print('in func1')

def timer(func):
    def inner():
        start = time.time()
        func()
        print(time.time() - start)
    return inner

func1 = timer(func1)          # 将函数本身做为参数传递进去
func1()  #inner
print(func1.__name__)        # 查看函数的名称

# Output:
inner
```

## 导入wraps装饰器

`wraps` 装饰器，用于帮助创建装饰器时**保留被装饰函数的元数据**（如名称、文档字符串等）。使用 `@wraps` 可以确保装饰后的函数看起来像原始函数，这样有助于调试和文档生成。

我们将上方的案例使用wraps装饰器装饰

```python
from functools import wraps
import time

def func1():
    print('in func1')

def timer(func):
    @wraps(func)
    def inner():
        start = time.time()
        func()
        print(time.time() - start)
    return inner

func1 = timer(func1)          # 将函数本身做为参数传递进去

print(func1.__name__)        # 查看函数的名称
```

# 带参数的装饰器

带参数的装饰器允许你在装饰器中接受参数，从而增强装饰器的灵活性和功能性。实现带参数的装饰器通常需要使用嵌套函数。

我们将创建一个装饰器，它接受一个参数，用于指定**是否打印函数的执行时间**。

```python
import time
from functools import wraps


def timing_decorator(print_time=True):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            start_time = time.time()  # 记录开始时间
            result = func(*args, **kwargs)  # 调用原始函数
            end_time = time.time()  # 记录结束时间

            if print_time:
                execution_time = end_time - start_time
                print(f"{func.__name__} 执行时间: {execution_time:.4f}秒")

            return result

        return wrapper

    return decorator


@timing_decorator(print_time=True)
def add(x, y):
    """返回两个数的和"""
    time.sleep(1)  # 模拟耗时操作
    return x + y


@timing_decorator(print_time=False)
def multiply(x, y):
    """返回两个数的积"""
    time.sleep(1)  # 模拟耗时操作
    return x * y


# 测试
result_add = add(5, 3)
print(f"加法结果: {result_add}")

result_multiply = multiply(5, 3)
print(f"乘法结果: {result_multiply}")

# Output:
add 执行时间: 1.0128秒
#没有输出multiply执行时间，print_time为False，if print_time部分的代码不执行
加法结果: 8
乘法结果: 15
```

# 多个装饰器装饰一个函数

可以将多个装饰器应用于同一个函数。这种情况下，装饰器会**按照从内到外的顺序**依次应用。

```python
def wrapper1(func):
    def inner1():
        print('第一个装饰器，在程序运行之前')
        func()
        print('第一个装饰器，在程序运行之后')
    return inner1

def wrapper2(func):
    def inner2():
        print('第二个装饰器，在程序运行之前')
        func()
        print('第二个装饰器，在程序运行之后')
    return inner2

@wrapper1
@wrapper2
def f():
    print('Hello')

f()

'''
f=wrapper2,对应的形参func是f，返回inner2
f=wrapper1,对应的形参func是inner2，返回inner1
执行时，先执行inner1(),print('第一个装饰器，在程序运行之前')，调用func()，func 是 wrapper2 的 inner2；
再执行inner2(),print('第二个装饰器，在程序运行之前')，调用func(),即f(),print('Hello')；
然后print('第二个装饰器，在程序运行之后')，回到inner1函数中，print('第一个装饰器，在程序运行之后')
'''
# Output:
第一个装饰器，在程序运行之前
第二个装饰器，在程序运行之前
Hello
第二个装饰器，在程序运行之后
第一个装饰器，在程序运行之后
```

`@wrapper2` 首先应用于 `f`，然后 `@wrapper1` 应用于 `wrapper2` 返回的结果。

当你调用 `f()` 时，实际执行的过程如下：

- `f` 首先被 `wrapper2` 装饰，返回一个新的函数 `inner`，这个 `inner` 函数会在调用时执行 `f`。
- 然后，`wrapper1` 装饰了 `wrapper2` 返回的 `inner` 函数，返回了另一个 `inner` 函数。

最终，`f` 实际上是一个被 `wrapper1` 和 `wrapper2` 装饰过的函数。

## 示例：多个装饰器

创建两个装饰器，一个用于打印函数的执行时间，另一个用于打印调用的参数。

```python
import time
from functools import wraps

# 装饰器 1：打印执行时间
def timing_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        execution_time = end_time - start_time
        print(f"{func.__name__} 执行时间: {execution_time:.4f}秒")
        return result
    return wrapper

# 装饰器 2：打印函数参数
def logging_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"调用 {func.__name__} 函数，参数: {args}, 关键字参数: {kwargs}")
        return func(*args, **kwargs)
    return wrapper

@timing_decorator
@logging_decorator
def add(x, y):
    """返回两个数的和"""
    time.sleep(1)  # 模拟耗时操作
    return x + y

# 测试
result = add(5, 3)
print(f"加法结果: {result}")

# Output:
调用 add 函数，参数: (5, 3), 关键字参数: {}
add 执行时间: 1.0001秒
加法结果: 8
'''
先使用@logging_decorator装饰add函数，再使用@timing_decorator装饰被@logging_decorator装饰过的add函数，timing_decorator的形参func是logging_decorator的wrapper。
调用时，add(5,3) -> timing_decorator.wrapper(5,3)
-func(5,3) -> logging_decorator.wrapper(5,3) 
-print(f"调用 {func.__name__} 函数，参数: {args}, 关键字参数: {kwargs}")
-return func(*args, **kwargs) -> add(5,3) ->return x + y
-print(f"{func.__name__} 执行时间: {execution_time:.4f}秒")
'''
```

# 开放封闭原则

开放封闭原则（Open/Closed Principle, OCP）是面向对象设计中的一个重要原则，它是 SOLID 原则之一。这个原则的核心思想是：

## 定义

- **开放性**：软件实体（如类、模块、函数等）应该对扩展开放。这意味着你应该能够通过增加新功能来扩展这些实体，而不是修改现有的代码。
- **封闭性**：软件实体应该对修改封闭。也就是说，应该避免直接修改已存在的代码，以防止引入错误或破坏现有功能。

## 目的

开放封闭原则的主要目的是提高代码的可维护性和可扩展性。遵循这一原则可以减少对现有代码的影响，从而降低引入新错误的风险。

# 装饰器的固定结构

```python
def outer(func):
    def inner(*args,**kwargs):
        '''执行函数之前要做的'''
        re = func(*args,**kwargs)
        '''执行函数之后要做的'''
        return re
    return inner


# 下面是加上wraps的固定结构
from functools import wraps

def outer(func):
    @wraps(func)
    def inner(*args,**kwargs)
        '''执行函数之前要做的'''
        re = func(*args,**kwargs)
        '''执行函数之后要做的'''
        return re
    return inner
```

```python
from functools import wraps
def decorator_with_args(param):
    def actual_decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"装饰器参数：{param}")
            return func(*args, **kwargs)
        return wrapper
    return actual_decorator

@decorator_with_args("配置参数")
def my_function():
    pass
```

