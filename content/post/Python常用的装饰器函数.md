---
title: "Python常用的装饰器函数"
date: 2023-08-02T15:32:03+08:00
toc: true
tags:
  - python
  - 装饰器
categories:
  - python
keywords:
  - python
  - 装饰器
---

## 1. python装饰器有什么用处

Python装饰器是一种用来修饰或扩展函数或类的工具，它可以在不修改被修饰对象的源代码的情况下，为其添加额外的功能或行为。装饰器可以用于以下几个方面：

1. 函数增强：装饰器可以在不改变函数原始定义的情况下，为函数添加额外的功能，例如日志记录、性能分析、输入验证等。

2. 权限控制：装饰器可以用于验证用户权限，限制某些用户或角色访问特定的函数或方法。

3. 缓存数据：装饰器可以用于缓存函数的计算结果，避免重复计算。

4. 类扩展：装饰器可以用于扩展类的功能，例如为类添加缓存、日志记录、属性验证等。

5. 路由映射：装饰器可以用于将函数或类映射到特定的URL路由，实现Web应用的路由功能。

总之，装饰器提供了一种灵活且无侵入性的方式，用于修改或扩展函数或类的行为，使得代码更加模块化、可复用和易于维护。

## 2. python装饰器的基本用法

### 2.1 函数装饰器

装饰器函数是一个接受函数作为参数，并返回一个新函数的函数。通常，装饰器函数会接受一个函数作为参数，并返回一个修改后的函数。

例如，下面是一个简单的装饰器函数：

```python
def my_decorator(func):
    def wrapper():
        print("Before the function is called.")
        func()
        print("After the function is called.")
    return wrapper
```

装饰器函数接受一个函数作为参数，并返回一个新函数。新函数的行为由装饰器函数的代码决定。例如，上面的装饰器函数会在被修饰函数执行前后打印一条消息。

下面是一个使用装饰器函数的例子：

```python
@my_decorator
def say_whee():
    print("Whee!")
```

上面的代码等价于：

```python
def say_whee():
    print("Before the function is called.")
    print("Whee!")
    print("After the function is called.")
```

### 2.2 类装饰器

类装饰器是一个接受类作为参数，并返回一个新类的函数。类装饰器通常会重写类的构造函数，以便在类实例化时修改类的行为。

例如，下面是一个简单的类装饰器：

```python
def my_decorator(cls):
    class wrapper(cls):
        def __init__(self):
            print("Before the class is called.")
            super().__init__()

        def some_method(self):
            print("decorator method called.")

    return wrapper
```

类装饰器接受一个类作为参数，并返回一个新类。新类的行为由类装饰器的代码决定。例如，上面的类装饰器会在被修饰类实例化时打印一条消息。

下面是一个使用类装饰器的例子：

```python
@my_decorator
class Foo:
    def __init__(self):
        print("Foo class created.")

    def some_method(self):
        print("Some method called.")
```

上面的代码等价于：

```python
class Foo:
    def __init__(self):
        print("Before the class is called.")
        print("Foo class created.")

    def some_method(self):
        print("decorator method called.")
```

## 2.3 装饰器类

装饰器类就是使用类来实现的装饰器。它们通常通过在类中定义 `__call__` 方法来实现。当我们使用 `@` 语法应用装饰器时，Python
会调用装饰器类的 `__init__` 方法创建一个实例，然后将被装饰的函数或类作为参数传递给 `__init__` 方法。当被装饰的函数或方法被调用时，Python
会调用装饰器实例的 `__call__` 方法。

例如，下面是一个简单的装饰器类：

```python
class MyDecorator:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("Before call")
        result = self.func(*args, **kwargs)
        print("After call")
        return result

@MyDecorator
def hello():
    print("Hello, world!")

hello()
```

上面的代码等价于：

```python
def hello():
    print("Before call")
    print("Hello, world!")
    print("After call")
```

装饰器类相对于装饰器函数的优点是，它们可以保存状态。例如，下面的装饰器类会记录被装饰函数被调用的次数：

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.num_calls = 0

    def __call__(self, *args, **kwargs):
        self.num_calls += 1
        print(f"Call {self.num_calls} of {self.func.__name__!r}")
        return self.func(*args, **kwargs)

@CountCalls
def say_whee():
    print("Whee!")


say_whee()
say_whee()
```

上方代码输出如下：

```shell
Call 1 of 'say_whee'
Whee!
Call 2 of 'say_whee'
Whee!
```

装饰器类可以利用 Python 的面向对象特性，将相关的方法和数据封装在一起，这使得代码更易于理解和维护。

装饰器类可以利用继承来复用和扩展代码。例如，你可以创建一个基础的装饰器类，然后通过继承这个类来创建特定的装饰器。

虽然装饰器类有很多优点，但是它们也有一些缺点。例如，装饰器类的代码通常比装饰器函数的代码更复杂。此外，装饰器类的行为和装饰器函数的行为有一些细微的差别。例如，装饰器类不能像装饰器函数那样接受参数。

## 3. 常用的装饰器

### 3.1 内置装饰器

Python中常用的装饰器有以下几种：

1. `@staticmethod`：将一个方法转换为静态方法，可以直接通过类名调用，无需创建实例。

```python
class MyClass:
    @staticmethod
    def my_static_method():
        pass
```

2. `@classmethod`：将一个方法转换为类方法，第一个参数为类本身，可以通过类名或实例调用。

```python
    @classmethod
    def my_class_method(cls):
        pass
```

3`@property`：将一个方法转换为属性，可以像访问属性一样访问该方法。

```python
class MyClass:
    @property
    def my_property(self):
        return self._my_property
```

4. `@abstractmethod`：用于声明抽象方法，子类必须实现该方法。

```python
from abc import ABC, abstractmethod

class MyAbstractClass(ABC):
    @abstractmethod    
    def my_abstract_method(self):
        pass
```
5. `@staticmethod`：用于声明静态方法，可以直接通过类名调用，无需创建实例。

```python
class MyClass:
    @staticmethod
    def my_static_method():
        pass
```

6. `@functools.wraps`：用于保留被装饰函数的元信息，如函数名、参数列表等。
```python
def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        pass    
    return wrapper
```

这些是Python中常用的装饰器，每个装饰器都有不同的用途和特点，可以根据实际需求选择使用

### 3.2 自定义装饰器
1. 重试装饰器
```python
import time
from functools import wraps

def retry(max_tries=3, delay_seconds=1):
    def decorator_retry(func):
        @wraps(func)
        def wrapper_retry(*args, **kwargs):
            tries = 0
            while tries < max_tries:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    tries += 1
                    if tries == max_tries:
                        raise e
                    time.sleep(delay_seconds)
        return wrapper_retry
    return decorator_retry
```

2. 缓存函数结果

如果输入相同，则该函数仅运行一次。在每个后续运行中，结果将从缓存中获取。因此，我们不必一直执行昂贵的计算。

```python
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args in cache:
            return cache[args]
        else:
            result = func(*args)
            cache[args] = result
            return result
    return wrapper
```

3. 计时器

```python
import time

def timing_decorator(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"Function {func.__name__} took {end_time - start_time} seconds to run.")
        return result
    return wrapper
```

4. 记录函数调用

```python
import logging
import functools

def log_execution(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        logging.info(f"Executing {func.__name__}")
        result = func(*args, **kwargs)
        logging.info(f"Finished executing {func.__name__}")
        return result
    return wrapper
```
5. 检查函数参数

```python
def validate_arguments(*arg_types, **kwarg_types):
    def decorator_validate_arguments(func):
        @functools.wraps(func)
        def wrapper_validate_arguments(*args, **kwargs):
            for (arg, arg_type) in zip(args, arg_types):
                if not isinstance(arg, arg_type):
                    raise TypeError(f"Expected {arg_type}")
            for (kwarg, kwarg_type) in kwarg_types.items():
                if not isinstance(kwarg, kwarg_type):
                    raise TypeError(f"Expected {kwarg_type}")
            return func(*args, **kwargs)
        return wrapper_validate_arguments
    return decorator_validate_arguments
```


