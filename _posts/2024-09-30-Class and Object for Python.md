---
layout:     post
title:      Class and Object for Python
subtitle:   
date:       2024-09-30
author:     Deng You
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - python
---

# Class and Object for Python

## 面向对象基础

- 定义
  - 新式类： `class A:`
  - 经典类：`class A(Object)`

```python
class Aclass:
    # 类变量
    var = 'class_value'
    
    def __init__(self, number, status):
        # 实例变量
        self.number = number
        self.status = status
        
    # 实例方法
    def method_of_instance(self):
        print('this is instance method.')
        
    # 类方法
    @classmethod
    def method_of_cls(cls):
        print('this is cls method.')
        
    # 静态方法   
    @staticmethod
    def static_method():
        print('this is static method.')
```

- 初始化

```python
a = Aclass(1, 'open')
```
- 调用变量

```python
Aclass.var # 通过类调用类变量
a.var # 通过实例调用类变量
```
```python
a.status # 通过实例调用实例变量
Aclass.var  #类无法调用实例变量，报错 AttributeError
```
- 调用方法

```python
a.method_of_instance() # 实例调用实例方法
Aclass.method_of_instance(a) # 类调用实例方法，需传入实例
```

```python
Aclass.method_of_cls() # 类调用类方法
a.method_of_cls() # 实例调用类方法
```
```python
a.static_method() #  实例调用静态方法
Aclass.static_method() # 类调用静态方法
```

###  封装

- 私有属性

以双下划线开始的且非双下划线结尾的变量/方法是**私有变量**。 通常不定义双下划线开始双下划线结尾的变量/方法，因为这在Python有特殊含义。

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.__age = age
        
    def __private_method(self):
        print('private method')
    
    @property #  此装饰器将方法变成属性， 不支持赋值
    def age(self):
        return self.__age
    
    @age.setter #  此装饰器实现被property装饰的属性可进行赋值
    def age(self, value):
        if value <0 or value > 150:
            raise ValueError(value)
        else:
            self.__age = value
            
    @age.deleter # 同setter， 当删除此属性时会调用此方法，这里表示禁止删除age属性
    def age(self):
        raise NotImplementedError('can not delete age of person ')
```

- 私有属性 在类的内部均可访问，无论是类方法，还是实例方法或者是静态方法

```python
class Bclass:
    __private_cls_var = 'private cls var'
    
    def __init__(self):
        self.__private_instance_var = 'private instance var'

    @classmethod
    def public_cls_method(cls):
        print(cls.__private_cls_var)
        
    def public_instance_method(self):
        print(Bclass.__private_cls_var)
        print(self.__private_instance_var)

    @staticmethod
    def static_method():
        print(Bclass.__private_cls_var)
    
    # 调用方式必须使用类 Bclass.public_method()进行调用
    def public_method(): 
        print(Bclass.__private_cls_var)
```

> 对于python而言，类/实例没有真正的私有属性，而是通过变量`__field`重新命名为`_CLASS__field`，所以均可以使用特殊的方法对类/实例的私有属性进行访问。

- 通过类访问私有属性：`Bclass._Bclass__private_cls_var`

```python
>>> Bclass.__dict__
mappingproxy({'__module__': '__main__',
              '_Bclass__private_cls_var': 'private cls var',
              '__init__': <function __main__.Bclass.__init__(self)>,
              'public_cls_method': <classmethod(<function Bclass.public_cls_method at 0x10ed4aa70>)>,
              'public_instance_method': <function __main__.Bclass.public_instance_method(self)>,
              'static_method': <staticmethod(<function Bclass.static_method at 0x10ed48a60>)>,
              'public_method': <function __main__.Bclass.public_method()>,
              '__dict__': <attribute '__dict__' of 'Bclass' objects>,
              '__weakref__': <attribute '__weakref__' of 'Bclass' objects>,
              '__doc__': None})
```
- 通过实例访问私有属性: `b._Bclass__private_instance_var`

```python
b = Bclass()

>>> b.__dict__
{'_Bclass__private_instance_var': 'private instance var'}
```

###  继承

Python3 中所有类继承自 object，子类获取父类的一些方法和属性。

- 定义
```python
class A: # 父类
    pass
class B(A): # 子类/派生类
    pass 
```

- 类/实例的变量都可以继承，但是私有变量除外
- 类/实例的方法都可以继承，但是私有变量除外

```python
class Animal:
    public_cls_var = 'public cls var'
    __private_cls_var = 'private cls var' # 不可继承
    
    def __init__(self):
        self.a = 'A'
        self._b = 'B'
        self.__C = 'C' # 不可继承

    def public_instance_method(self):
        print('public instance method')

    def __private_instance_method(self): # 不可继承
        print('private instance method')

    @classmethod
    def cls_method(cls):
        print('cls method')

    @staticmethod
    def static_method():
        print('static  method')

    def general_method():
        print('general  method')
        
class Cat(Animal):
    pass
```

- **方法重写**

方法重写,子类覆盖父类的方法，在 Python 中方法重写也算一种多态的体现。


```python
class Animal:
    def p(self):
        print('i am animal')

class Cat(Animal):
    def p(self):
        print('i am cat')
        
```

- **super**

super 方法返回 supper对象，可以使用 supper对象调用父类的方法

```python
class Animal:
    def p(self):
        print('i am animal')

class Cat(Animal):
    def p(self):
        print('i am cat')

class TomCat(Cat):
    def p(self):
        super(TomCat, self).p() # 调用父类
        # 同上等价
        super(__class__, self).p()
        super(Cat, self).p() # 调用祖先类, 跨类调用
```

父类定义了 `__init__` 方法时，子类要显示的定义初始化方法，并且要在初始化方法里面初始化父类

```python
class Animal:
    def __init__(self, x):
        self.x = x
        
    def p(self):
        print('i am animal')

class Cat(Animal):
    def __init__(self, y):
        super().__init__(y)
        
    def p(self):
        print('i am cat')
        super().p() # super 方法返回 supper对象，可以使用 supper对象调用父类的方法
        super(Animal, self).p()
```

不能使用 super 方法调用父类的变量， 要访问父类的属性需使用 `__dict__`。

```python
class Animal:
    def __init__(self, x):
        self.x = x
        
    def p(self):
        print('i am animal')

class Cat(Animal):
    def __init__(self):
        super().__init__(3)
        
    def p(self):
        print('i am cat')
        super().p() 
        # print(super().x)
        print(super().__dict__['x'])
```

### 多继承

- 定义

```python
class A:
    def p(self):
        print(
            
            'A')
class B:
    def p(self):
        print('B')
class C(A,B):
    pass
```
- 方法解析顺序MRO

多继承中，父类继承的顺序不一样，最后的结果也会有差异，例如对于类`C(A,B)`与 类`C(B,A)`，其实例 `c.p`调用结果完全不同，这是MRO导致的，MRO使用C3算法实现。C3 算法是一种广度优先搜索算法，它可以保证满足多重继承的方法解析顺序。

```python
class C(A,B):
    pass

>>> C.__mro__
(__main__.C, __main__.A, __main__.B, object)

class C(B,A):
    pass

>>> C.__mro__
(__main__.C, __main__.B, __main__.A, object)
```

- MRO merge过程

遍历序列，取出首元素（首元素需要在其他序列中存在且也是首元素, 如果存在且不是首元素，则继承就会抛出异常 `TypeError`），直到循环所有的元素。
```python
class C(A, B) ==>

MRO(C) => [C] + merge(MRO(A), MRO(B), [A, B])
       => [C] + merge([A, O), (B, O), (A, B)])
       => [C, A] + merge([O], [B, O], [B])
       => [C, A, B] + merge([O], [O])
       => [C, A, B, O]
```

> 应该尽量避免使用多继承，以免 mro 定义类报错

### Mixin

Mixin 是一种组合，在python 中通过多继承的方式实现。（待补充）



## 面向对象进阶

方法前后带双下滑线的方法，称之为**专有方法**或**魔术方法**。

### 运算符重载

通过执行 help(int) ,可以查看所有可重载运算符。

- 算术运算符
- 比较运算符
- 位运算符

定义一个复数类，定义 `__add__` 和 `__sub__`专有方法，实现两个类实例之间的运算。

```python
class Complex:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Complex(self.x + other.x, self.y + other.y)
        
    def __sub__(self, other):
        return Complex(self.x - other.x, self.y - other.y)
    
    def __str__(self):
        return "<{}:{}>".format(self.x, self.y)


c1 = Complex(2, 3)
c2 = Complex(4, 5)

new = c1+c2

>>> print(new)
<6:8>
```

定义一个Person类，实现 两个Person实例通过年龄属性进行比较

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __gt__(self, other): # 实现两个类之间 > 比较符号
        return self.age > other.age
    
    def __lt__(self, other): # 实现两个类之间 < 比较符号
        return self.age < other.age
        
    def __eq__(self, other): # 实现两个类之间 == 比较符号
        return self.age == other.age
```

实现一个栈
- `__len__`: 实现 len 方法
- `__hash__`： 实现 hash方法，有这个方法的类，称为可 hash 对象
- `__bool__`: 实现 bool 方法

```python
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None
        
class Stack:
    def __init__(self):
        self.top = None
        self.length = 0

    def push(self, val):
        node = Node(val)
        node.next = self.top
        self.top = node
        self.length += 1

    def pop(self):
        if self.top is not None:
            top = self.top
            self.top = self.top.next
            self.length -= 1
            return top.value
        return None

    def __len__(self):
        return self.length
    
    def __hash__(self):
      return  self.top
    
    def __bool__(self):
      return true

s = Stack()
s.push(1)
s.push(2)

>>> len(s)
2
>>> s.pop()
1
>>> len(s)
1
```

### 可调用对象

当一个对象实现 `__call__` 方法，这样的对象称为可调用对象。可调用对象，实际上调用的是 `__call__` 方法。可以通过内置函数 `callable` 判断一个对象是否是可调用对象。

```python
class Add:
    def __call__(self, a, b):
        print('{} + {}'.format(a, b))
        return a+b
a = Add()

>>> a(2, 5)
7
```

通过 `__call__ `方法，可实现复杂的单例类装饰器

```python
from functools import  wraps
class Singleton:
    def __init__(self, cls):
        wraps(cls)(self)
        self.instance = None

    def __call__(self, *args, **kwargs):
        if self.instance is None:
            self.instance = self.__wrapped__(*args, **kwargs)
        return self.instance

@Singleton
class A:
    '''this A class'''
    pass

a = A()
a1 = A()

>>> a is a1
True
```

```python
import time 
class Timeit:
    def __init__(self, fn):
        self.fn = fn

    def __call__(self, *args, **kwargs):
        start = time.time()
        ret = self.fn(*args, **kwargs)
        end = time.time()
        print(end-start)
        return ret

@Timeit
def sleep(n):
    time.sleep(n)

>>> sleep(3) # 等价 Timeit(sleep(3))
3.0021469593048096
>>> sleep.__name__
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
Cell In[317], line 1
----> 1 sleep.__name__

AttributeError: 'Timeit' object has no attribute '__name__'
```

优化 Timeit 类，实现 wraps, wraps 传递一些属性，通过`assigned=` 传递 ,具体可通过 `help(warp)` 查看

```python
import time 
from functools import wraps

class Timeit:
    def __init__(self, fn):
        self.wrapped = wraps(fn)(self)

    def __call__(self, *args, **kwargs):
        start = time.time()
        ret = self.wrapped.__wrapped__(*args, **kwargs)
        end = time.time()
        print(end-start)
        return ret

@Timeit
def sleep(n):
    '''sleep some time '''
    time.sleep(n)

>>> sleep(3)
3.0030908584594727
>>> sleep.__doc__
'sleep some time '
>>> sleep.__name__
sleep
```
### 对象可视化

`__repr__`和`__str__`方法实现实例对象的可视化,返回字符串，增加实例的可读性。
- `__repr__`: 返回的字符串更多的反映解释器相关的内容
- `__str__`: 返回的字符串更接近自然语言

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return '[{}:{}]'.format(self.name, self.age)

    # __repr__ = __str__
    def __repr__(self):
        return '<{}:{}:{}:{}_{} at {}>'.format(self.__module__, self.__class__.__name__, self.name, self.age, self, hex(id(self)))
        
p = Person('jerry', 18)
>>> p
<__main__:Person:jerry:18_[jerry:18] at 0x10ef621d0>
>>> print(p)
[jerry:18]
```

### 上下文管理

类要实现上下文管理，必须实现 `__enter__` 和 `__exit__` 方法，这样就可以使用 with xx as f 的语法。

进入 with 块之前执行 `__enter__` 方法， 退出with块之后，立刻执行 `__exit__`方法

as 子句用于把 __enter__ 的返回值复制给 变量 f

```python
class Context:
    def __init__(self):
        print('1. init ')

    def __enter__(self):
        print('2. enter')
        return 123
        
    def __exit__(self, *args, **kwargs):
        print('3. exit')

>>> with Context() as f:
>>>     print(f)
>>>     print('do somethings')
1. init 
2. enter
do somethings
3. exit

```

标准库 **contextlib** 实现上下文管理

```python
import contextlib
@contextlib.contextmanager
def test():
    print('enter')
    try:
        yield 123 # 必须为 iterator 对象
    finally:
        print('exit')

with test() as x:
    print(x)
```

### 反射

在代码中获取对象本身的一些属性，例如对象的字段，方法。例如 dir 就实现了反射。

| 方法           | 说明               | 案例             | 输出      |
|--------------|------------------|----------------|---------|
| `__class__`  | 获取当前实例的类名        | `a.__class__`  | `__main__.A` |
| `__dict__`   | 获取当前实例的持有的所有变量   | `a.__dict__ `  | `{'x': 3}` |
| `__dir__`    | 获取当前实例的所有属性      | `a.__dir__()`  | 同dir(a) |
| `__module__` | 获取当前实例所在的模块      | `a.__module__` | `'__main__'   ` |
| `__name__`   | 获取当前类的名字，实例没有此方法 | `A.__name__`   | `'A'`   |
| `__doc__`      | 获取当前实例的文档        | `a.__doc__`      | -  |


```python
class A:
    cls_var = 'class value'
    def __init__(self, x):
        self.x = x

    def get_x(self):
        return self.x

a = A(3)

>>> dir(a)
['__class__',
 '__delattr__',
 '__dict__',
 '__dir__',
 '__doc__',
 '__eq__',
 '__format__',
 '__ge__',
 '__getattribute__',
 '__gt__',
 '__hash__',
 '__init__',
 '__init_subclass__',
 '__le__',
 '__lt__',
 '__module__',
 '__ne__',
 '__new__',
 '__reduce__',
 '__reduce_ex__',
 '__repr__',
 '__setattr__',
 '__sizeof__',
 '__str__',
 '__subclasshook__',
 '__weakref__',
 'cls_var',
 'get_x',
 'x']
```

**使用字符串操作对象的属性**

- `__getattribete__`

当定义`__getattribete__`方法时，不管属性存在或者不存在，都会调用 `__getattribete__`,且不管是方法还是变量，都只能通过字符串访问类的属性，否则会报错 `TypeError`。

```python
class A:
    cls_var = 'class value'
    def __init__(self, x):
        self.x = x

    def get_x(self):
        return self.x
        
    def __getattribute__(self, name, default=None):
        return 'getattribute proptery'

a = A(3)

>>> getattr(a, 'testxxx')
'getattribute proptery'

>>> a.testxxx
'getattribute proptery'

>>> a.x
'getattribute proptery'

>>> a.get_x
'getattribute proptery'

>>> a.get_x()
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
Cell In[419], line 29
     26 a.get_x
     27 'getattribute proptery'
---> 29 a.get_x()

TypeError: 'str' object is not callable
```

- `getattr`

当未定义`__getattribete__`方法时，只定义了 `__getattr__` 方法时, 属性不存在时，访问 `__getattr__` 方法, 属性存在，正常调用该属性。

```python
class A:
    cls_var = 'class value'
    def __init__(self, x):
        self.x = x

    def get_x(self):
        return self.x

    def __getattr__(self, name, default=None):
        return 'missing proptery'

a = A(3)

>>> getattr(a, 'testxxx')
'missing proptery'

>>> a.testxxx
'missing proptery'

>>> a.x
3
>>> a.get_x()
3
```

`getattr` 可以传入默认值，当属性不存在是，则调用 默认值，这是一种设计模式。
- `getattr(object, name[, default]) -> value`

```python
class WorkerInterface:
    def method1(self):
        print('WorkerInterface 1')

    def method2(self):
        print('WorkerInterface 2')

    def method3(self):
        print('WorkerInterface 3')

class WorkerImpl:
    def method2(self):
        print('WorkerImpl 2')

interface = WorkerInterface()

impl = WorkerImpl()

>>> getattr(impl, 'method2', interface.method2)()
WorkerImpl 2
>>> getattr(impl, 'method1', interface.method1)()
WorkerInterface 1
```

- `setattr(obj, name, value, /)`

setattr 通过属性名修改属性，实际调用 `__setattr__` 方法,属性存在时修改属性值，属性不存在时，添加属性。

```python
class A:
    cls_var = 'class value'
    def __init__(self, x):
        self.x = x

    def get_x(self):
        return self.x

a = A(3)

>>> a.x
3

>>> setattr(a, 'x', 30)
>>> a.x
30

>>> setattr(a, 'y', 40)
>>> a.y
40
```

- `delattr(obj, name, /)`

delattr 实际调用 `__delattr__` 方法,属性存在时删除属性值，属性不存在时，`抛出AttributeError`。

```python
class A:
    cls_var = 'class value'
    def __init__(self, x):
        self.x = x

    def get_x(self):
        return self.x

a = A(5)

class A:
    cls_var = 'class value'
    def __init__(self, x):
        self.x = x

    def get_x(self):
        return self.x


a = A(5)

>>> delattr(a, 'x')
>>> hasattr(a, 'x')
False

>>> a.x
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
Cell In[453], line 1
----> 1 a.x

AttributeError: 'A' object has no attribute 'x'
```

###  描述器

一个具有 __get__ 和 __set__ 方法的类就是一个描述器，IntClass 是一个描述器。描述器可用于需要控制对属性的访问、赋值，删除。描述器单独出现时无意义的，它总是和其他类一起出现，它一般**用于描述其它类属性的访问赋值和删除行为**。

当一个类属性具有 `__get__` 和 `__set__` 方法的时候,当访问这个类属性时，调用的是类属性的 `__get__` ；当对类属性进行赋值时，调用`__set__` 方法。

```python
class IntClass: # 描述器
    def __init__(self, name):
        self.name = name 
        
    def __get__(self, instance, cls):
        if instance is None: #  A.x 调用时， instance 为 None
          return self
        print('get')
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        if not isinstance(value, int): # 描述器的用法，类型检查
          raise TypeError("{} require int".format(self.name))
        print('set')
        instance.__dict__[self.name] = value
        
    def __delete__(self, instance):
        raise TypeError("不允许删除")
    
class A:
    x = IntClass('x') # 类变量， 类变量的类型为IntClass

    def __init__(self, x):
        self.x = x

>>> a = A(3)
set

>>> a.x # A.x.__get__(a, A)
get
3

>>> a.x = 15 # A.x.__get__(a, 15)
set 
```

@property 就是由描述器实现, 自行实现代码如下。

```python
class Property:
    def __init__(self, fget=None, fset=None, fdel=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel

    def __get__(self, instance, cls):
        if instance is None:
            return self
        if not callable(self.fget):
            raise AtrributeError()
        return self.fget(instance)

    def __set_(self, instance, value):
        if not callable(self.fset):
            raise AtrributeError()
        self.fset(instance, value)

    def __del_(self, instance):
        if not callable(self.fdel):
            raise AtrributeError()
        self.fdel(instance)

    def setter(self, fset):
        self.fset = fset
        
    def deleter(self, fdel):
        self.fdel = fdel   

class A:
    def __init__(self, x):
        self.__x = x

    @Property
    def x(self):
        return self.__x
    
    # x = Property(A.x)
    
    @x.setter
    def x(self, value):
        self.__x = value
```

实现类型检查版本

```python
class Typed:
    def __init__(self, name, required_type):
        self.name = name
        self.required_type = required_type
    
    def __get__(self, instance, cls):
        if instance is None:
            return self
        return instance.__dict__[self.name]
    
    def __set__(self, instance, value):
        if not isinstance(value, self.required_type):
            raise TypeError('{} required type {}'.format(self.name, self.required_type))
        instance.__dict__[self.name] = value

class A:
    x = Typed('x', str)
    y = Typed('y', int)

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

python3 中使用 inspect 实现类型检查

```python
from inspect import signature

class Typed:
    def __init__(self, name, required_type):
        self.name = name
        self.required_type = required_type
    
    def __get__(self, instance, cls):
        if instance is None:
            return self
        return instance.__dict__[self.name]
    
    def __set__(self, instance, value):
        if not isinstance(value, self.required_type):
            raise TypeError('{} required type {}'.format(self.name, self.required_type))
        instance.__dict__[self.name] = value

def typeassert(cls):
    sig = signature(cls)
    for k, v in sig.parameters.items():
        setattr(cls, k, Typed(k, v.annotation))
    return cls

@typeassert
class Person:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age
```

### 元类






