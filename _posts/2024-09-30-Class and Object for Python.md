---
layout:     post
title:      Class and Object for Python
subtitle:   
date:       2024-09-30
author:     Deng You
header-img: img/post-bg-minio.jpg
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
# 通过类调用类变量
Aclass.var
# 通过实例调用类变量
a.var
```
```python
# 通过实例调用实例变量
a.status
#类无法调用实例变量，报错 AttributeError
Aclass.var 
```
- 调用方法

```python
# 实例调用实例方法
a.method_of_instance()
# 类调用实例方法，需传入实例
Aclass.method_of_instance(a)
```

```python
# 类调用类方法
Aclass.method_of_cls()
# 实例调用类方法
a.method_of_cls()
```
```python
#  实例调用静态方法
a.static_method()
# 类调用静态方法
Aclass.static_method()
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
Bclass.__dict__
输出:
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
b.__dict__
输出如下：
{'_Bclass__private_instance_var': 'private instance var'}
```

###  继承

Python3 中所有类继承自 object，子类获取父类的一些方法和属性。

- 私有属性无法继承
- 

```python

```



