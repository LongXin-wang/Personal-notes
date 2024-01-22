- [标准数据类型](#标准数据类型)
  - [列表\&元组](#列表元组)
  - [具名元组](#具名元组)
  - [字典和集合](#字典和集合)
    - [集合](#集合)
    - [字典](#字典)
    - [字符串格式化](#字符串格式化)
- [函数](#函数)
  - [值传递vs引用传递](#值传递vs引用传递)
  - [传参](#传参)
  - [自定义函数](#自定义函数)
    - [嵌套函数](#嵌套函数)
    - [闭包](#闭包)
  - [匿名函数](#匿名函数)
- [JSON](#json)
- [异常](#异常)
  - [处理异常](#处理异常)
  - [异常输出](#异常输出)
      - [sys.exc\_info()方法：获取异常信息](#sysexc_info方法获取异常信息)
      - [traceback进行获取](#traceback进行获取)
- [迭代器\&生成器](#迭代器生成器)
  - [迭代器](#迭代器)
  - [生成器](#生成器)
    - [列表解析](#列表解析)
    - [生成器](#生成器-1)
- [类](#类)
  - [init new call](#init-new-call)
  - [静态方法、类方法、实例方法](#静态方法类方法实例方法)
  - [\_\_slots\_\_魔法](#__slots__魔法)
  - [metaclass \& type](#metaclass--type)
    - [type](#type)
    - [metaclass](#metaclass)
- [上下文管理器](#上下文管理器)
  - [基于类实现上下文](#基于类实现上下文)
  - [@contextmanager + 生成器实现上下文](#contextmanager--生成器实现上下文)
- [装饰器](#装饰器)
  - [装饰器类型](#装饰器类型)
  - [多层装饰器](#多层装饰器)
- [反射](#反射)

# 标准数据类型

strings, tuples, 和 numbers 是不可更改的对象（可哈希），而 list, dict、set 等则是可以修改的对象（不可哈希）

## 列表&元组

- 列表是动态的，长度可变，可以随意的增加、删减或改变元素。列表的存储空间略大于元组，性能略逊于元组。
- 元组是静态的，长度大小固定，不可以对元素进行增加、删减或者改变操作。元组相对于列表更加轻量级，性能稍优。


## 具名元组

- collections.namedtuple       创建只有属性没有方法的对象
- SqlAlchemy 查出来的row对象

```python
>>> from collections import namedtuple
>>> City = namedtuple('City', 'name country population coordinates') ➊
>>> tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667)) ➋
>>> tokyo
City(name='Tokyo', country='JP', population=36.933, coordinates=(35.689722,
139.691667))
>>> tokyo.population ➌
36.933
>>> tokyo.coordinates
(35.689722, 139.691667)
>>> tokyo
```


**迭代**

```python
for i in list：
  list[i]
  
for i in range(len(list)):
  list[i]
  
for i, val in enumerate(list): 

for i, val in enumerate(list, 2): 设置开始位置为2
```

## 字典和集合

### 集合

- 不解决hash冲突的，有冲突就不能放入
  
![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401242023452.png)

### 字典

- 解决hash冲突，拉链法
  
![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202401242024844.png)

**迭代**
```python
for key in dict
for key in dict.keys ()
for values in dict.values ()

for item in dict.items ()
for key,value in dict.items () ```

## 字符串

### 字符串常用操作

字符串想象成一个由单个字符组成的数组，所以，Python的字符串同样支持索引，切片和遍历等等操作

```python
name = 'jason'
name[0]
'j'
name[1:3]
'as'
```

**迭代**
```python
for ch in strs:

for index, ch in enumerate(strs):

for index in range(len(strs)):
  strs[i]

for ch in iter(strs):
```

**改变**

你可能了解到，在其他语言中，如Java，有可变的字符串类型，比如StringBuilder，每次添加、改变或删除字符（串），无需创建新的字符串，时间复杂度仅为O(1)。这样就大大提高了程序的运行效率。

但可惜的是，Python中并没有相关的数据类型，我们还是得老老实实创建新的字符串。因此，每次想要改变字符串，往往需要O(n)的时间复杂度，其中，n为新字符串的长度。
```python
s = 'H' + s[1:]
s = s.replace('h', 'H') # 都是新地址
```

此外，常见的函数还有：

- string.strip(str)，表示去掉首尾的str字符串；  
- string.lstrip(str)，表示只去掉开头的str字符串；  
- string.rstrip(str)，表示只去掉尾部的str字符串。

### 字符串格式化

```python
print('no data available for person with id: {}, name: {}'.format(id, name))

print('no data available for person with id: %s, name: %s' % (id, name))

```

# 函数

## 值传递vs引用传递

参数可变的是引用传递，参数不可变，就是值传递

## 传参

参数定义的顺序必须是：必传（位置）参数、默认参数、可变参数、关键字参数。

def f1(a, b, c=0, *args, **kw)

- 必传（位置）参数：平时最常用的，必传确定数量的参数
- 默认参数：在调用函数时可以传也可以不传，如果不传将使用默认值； 关键字参数
- 可变参数：可变长度参数
- 关键字参数：长度可变，但是需要以 key-value 形式传参

## 自定义函数

### 嵌套函数

内部函数隐私保护；

```python
def outer():
    x = "local"
    def inner():
        nonlocal x # nonlocal关键字表示这里的x就是外部函数outer定义的变量x
        x = 'nonlocal'
        print("inner:", x)
    inner()
    print("outer:", x)
outer()
# 输出
inner: nonlocal
outer: nonlocal

```

### 闭包

闭包外部函数返回的是一个函数，而不是一个具体的值。返回的函数通常赋于一个变量，这个变量可以在后面被继续执行调用。

```python
ef nth_power(exponent):
    def exponent_of(base):
        return base ** exponent
    return exponent_of # 返回值是exponent_of函数

square = nth_power(2) # 计算一个数的平方
cube = nth_power(3) # 计算一个数的立方 
square
# 输出
<function __main__.nth_power.<locals>.exponent(base)>

cube
# 输出
<function __main__.nth_power.<locals>.exponent(base)>

print(square(2))  # 计算2的平方
print(cube(2)) # 计算2的立方
# 输出
4 # 2^2
8 # 2^3
```

## 匿名函数

```python
lambda [arg1 [,arg2,.....argn]]:expression
lambda 参数 ：表达式

# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2
 
# 调用sum函数
print "相加后的值为 : ", sum( 10, 20 )

l = [1, 2, 3, 4, 5]
new_list = map(lambda x: x * 2, l) # [2， 4， 6， 8， 10]

l = [1, 2, 3, 4, 5]
new_list = filter(lambda x: x % 2 == 0, l) # [2, 4]
```

# JSON

- json.dumps() 这个函数，接受 Python 的基本数据类型，然后将其序列化为 string；

- 而json.loads() 这个函数，接受一个合法字符串，然后将其反序列化为 Python 的基本数据类型。
  
```python
import json

params = {
    'symbol': '123456',
    'type': 'limit',
    'price': 123.4,
    'amount': 23
}

params_str = json.dumps(params)

print('after json serialization')
print('type of params_str = {}, params_str = {}'.format(type(params_str), params))

original_params = json.loads(params_str)

print('after json deserialization')
print('type of original_params = {}, original_params = {}'.format(type(original_params), original_params))

########## 输出 ##########

after json serialization
type of params_str = <class 'str'>, params_str = {'symbol': '123456', 'type': 'limit', 'price': 123.4, 'amount': 23}
after json deserialization
type of original_params = <class 'dict'>, original_params = {'symbol': '123456', 'type': 'limit', 'price': 123.4, 'amount': 23}
```

# 异常

异常，通常是指程序运行的过程中遇到了错误，终止并退出。我们通常使用try except语句去处理异常，这样程序就不会被终止，仍能继续执行。

处理异常时，如果有必须执行的语句，比如文件打开后必须关闭等等，则可以放在finally block中。

## 处理异常

- 不要乱用异常， 到底用try 还是 if 要根据实际情况选择
- try--except (traceback.format_exc()) 这种会z执行except后继续往下执行
- try--except （raise） 会在except时raise抛出，不往下执行了
- if a: raise 则是直接抛出，不往下执行

```python
try:
    statement(s)            # 要检测的语句块
except exception1：
    deal_exception_code # 如果在 try 部份引发了 'exception' 异常
except (Exception1[, Exception2[,...ExceptionN]]]) :
    deal_all_other_exception2_code # 处理多个异常
except:
    deal_all_other_exception2_code # 处理全部其它异常
    print('a')
    raise 
else:
    no_exception_happend_code #如果没有异常发生

try:
    # <语句>
finally:
    # <语句>    #退出try时总会执行

>>> raise
Traceback (most recent call last):
  File "<pyshell#1>", line 1, in <module>
    raise
RuntimeError: No active exception to reraise
>>> raise ZeroDivisionError
Traceback (most recent call last):
  File "<pyshell#0>", line 1, in <module>
    raise ZeroDivisionError
ZeroDivisionError
>>> raise ZeroDivisionError("除数不能为零")
Traceback (most recent call last):
  File "<pyshell#2>", line 1, in <module>
    raise ZeroDivisionError("除数不能为零")
ZeroDivisionError: 除数不能为零
```

## 异常输出

#### sys.exc_info()方法：获取异常信息

exc_info() 方法会将当前的异常信息以元组的形式返回，该元组中包含 3 个元素，分别为 type、value 和 traceback，它们的含义分别是：

- type：异常类型的名称，它是 BaseException 的子类
- value：捕获到的异常实例。
- traceback：是一个 traceback 对象。

```Python
#使用 sys 模块之前，需使用 import 引入
import sys
try:
    x = int(input("请输入一个被除数："))
    print("30除以",x,"等于",30/x)
except:
    print(sys.exc_info())
    print("其他异常...")
    
请输入一个被除数：0
(<class 'ZeroDivisionError'>, ZeroDivisionError('division by zero',), <traceback object at 0x000001FCF638DD48>)
其他异常...

```

#### traceback进行获取

- traceback.print_exc() 直接打印异常 **(类似于没有通过try捕获异常，解析器直接报错的状态。) 
- traceback.format_exc() 返回字符串  (将异常的详细信息以字符串的形式返回)
- traceback.print_exc(file=open(‘你要保存的文件名.txt’,’a+’))    （直接将详细的异常信息保存在文件中）

```Python
# 导入trackback模块
import traceback
class SelfException(Exception): pass

def main():
    firstMethod()

try:
    main()
except:
    # 捕捉异常，并将异常传播信息输出控制台
    traceback.print_exc()
    traceback.format_exc() # 用的比较多
    # 捕捉异常，并将异常传播信息输出指定文件中
    traceback.print_exc(file=open('log.txt', 'a'))
```

# 迭代器&生成器

## 迭代器

而可迭代对象，通过 iter() 函数返回一个迭代器，再通过 next() 函数就可以实现遍历。for in 语句将这个过程隐式化，所以，你只需要知道它大概做了什么就行了。

迭代器（iterator）提供了一个 next 的方法。调用这个方法后，你要么得到这个容器的下一个对象，要么得到一个 StopIteration 的错误。

list、dict等不实现迭代器，是因为：运行对象动态生成迭代器，而不是一开始就是迭代器

```python
class Iterable(metaclass=ABCMeta):

    __slots__ = ()

    @abstractmethod
    def __iter__(self):
        while False:
            yield None

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterable:
            return _check_methods(C, "__iter__")
        return NotImplemented

    __class_getitem__ = classmethod(GenericAlias)


class Iterator(Iterable):

    __slots__ = ()

    @abstractmethod
    def __next__(self):
        'Return the next item from the iterator. When exhausted, raise StopIteration'
        raise StopIteration

    def __iter__(self):
        return self

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Iterator:
            return _check_methods(C, '__iter__', '__next__')
        return NotImplemented
```

## 生成器

### 列表解析

```python
tshirts = [(color, size) for color in colors for size in sizes] 

prices2 = {key: value for key, value in prices.items() if value > 100}
```

### 生成器

```python
li = [x * x for x in range(10)]
print(li)      #[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
g = (x * x for x in range(10))   #得到g是一个生成器对象
print(g)       #<generator object <genexpr> at 0x104d7a030>
#使用next一个个取出。也可使用 for i in g   for循环可操作迭代器（生成器实现了迭代器）
next(g)    #0
next(g)   #1
```

# 类

在Python中，当实例化子类时，首先会调用子类的__new__方法来创建实例，然后调用子类的__init__方法来对实例进行初始化。如果子类没有定义__new__方法，Python会调用父类的__new__方法来创建实例，然后再调用子类的__init__方法来初始化这个实例。

## init new call

在Python中，实际创建对象的过程是由`__new__`(构造器)方法控制的，该方法接收class对象(cls)。而`__init__`（初始化器）方法则是在`__new__`方法所创建的对象实例上，进行属性的赋值或者其它操作，所以接收实例对象(self)。

当想要控制创建对象的过程时，应该使用`__new__`方法，例如常用的单例模式，而不是使用`__init__`方法:

- new: 创建类实例cls，还没有实例对象；_new__方法在类定义中不是必须写的，如果没定义，默认会调用object.__new__去创建一个对象。如果定义了，就是会覆盖,使用自定义的，这样就可以自定制创建对象的行为。
- init：创建实例对象；第一个参数是self，该self参数就是__new__()返回的实例，**init**()在__new__()的基础上可以完成一些其它初始化的动作，**init**()不需要返回值。
- call：对象通过提供__call__(slef, *args ,**kwargs)方法可以**模拟函数的行为**，如果一个对象x提供了该方法，就可以像函数一样使用它，也就是说x(arg1, arg2...) 等同于调用x.**call**(self, arg1, arg2) 。有点给对象函数性质的意思

```python
class Person(object):

    def __new__(cls, *args, **kwargs):
        print('Demo __new__')
        return super(Person, cls).__new__(cls)

    def __init__(self, name, age):
        print('Demo __init__')
        self.name = name
        self.age = age

    def __call__(self, behavior: str):
        print('Demo __call__ : ' + behavior)

person = Person(name='perter', age=18)  //调用init
person('hello') //调用call
```

## 静态方法、类方法、实例方法

- 如果你用不到self上的实例变量，就考虑类方法。如果你用不到cls上的类变量，就考虑静态方法。如果这个方法和这个类一毛钱关系都没有，就考虑脱离成员关系。
- 静态方法无法访问类和实例中的任何属性；可以在其中通过类名访问类属性
- 类方法只能访问类变量

## __slots__魔法

限制属性

Python是一门。通常，动态语言允许我们在程序运行时给对象绑定新的属性或方法，当然也可以对已经绑定的属性和方法进行解绑定。但是如果我们需要限定自定义类型的对象只能绑定某些属性，可以通过在类中定义__slots__变量来进行限定。需要注意的是__slots__的限定只对当前类的对象生效，对子类并不起任何作用。

```Python
class Person(object):

    # 限定Person对象只能绑定_name, _age和_gender属性
    __slots__ = ('_name', '_age', '_gender')

    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def name(self):
        return self._name

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, age):
        self._age = age

    def play(self):
        if self._age <= 16:
            print('%s正在玩飞行棋.' % self._name)
        else:
            print('%s正在玩斗地主.' % self._name)


def main():
    person = Person('王大锤', 22)
    person.play()
    person._gender = '男'
    # AttributeError: 'Person' object has no attribute '_is_gay'
    person._is_gay = True

```

## metaclass & type

### type

type函数语法：type(args1,args2,args3) 其中args1是字符串类型，args2是元组类型，args3是字典类型。

type(类名，由父类名称组成的元组（针对继承的情况，可以为空），包含属性的字典（名称和值)

```python
def __new__(
    cls: type[_typeshed.Self], __name: str, __bases: tuple[type, ...], __namespace: dict[str, Any], **kwds: Any
) -> _typeshed.Self: ...
def __call__(self, *args: Any, **kwds: Any) -> Any: ...
```

Pyhton类的创建过程
1. 当 Python 见到 `class` 关键字时，会首先解析 `class ...` 中的内容。例如解析基类信息，最重要的是找到对应的元类信息（默认是 `type`)。
2. 元类找到后，Python 需要准备 namespace （也可以认为是上节中 `type` 的 `dict` 参数）。如果元类实现了 `__prepare__` 函数，则会调用它来得到默认的 namespace 。
3. 之后是调用 `exec` 来执行类的 body，包括属性和方法的定义，最后这些定义会被保存进 namespace。
4. 上述步骤结束后，就得到了创建类需要的所有信息，这时 Python 会调用元类的构造函数来真正创建类。

```python
1.使用type创建带有属性的类,添加的属性是类属性，并不是实例属性
Girl = type("Girl",(),{"country":"china","sex":"male"})
girl = Girl()
print(girl.country,girl.sex)  #使用type创建的类，调用属性时IDE不会自动提示补全
print(type(girl),type(Girl))
'''
china male
<class '__main__.Girl'> <class 'type'>
'''
 
2.使用type创建带有方法的类
# python中方法有普通方法，类方法，静态方法。
def speak(self): #要带有参数self,因为类中方法默认带self参数。
    print("这是给类添加的普通方法")
 
@classmethod
def c_run(cls):
    print("这是给类添加的类方法")
 
@staticmethod
def s_eat():
    print("这是给类添加的静态方法")
 
# 创建类，给类添加静态方法，类方法，普通方法。跟添加类属性差不多.
Boy = type("Boy",(),{"speak":speak,"c_run":c_run,"s_eat":s_eat,"sex":"female"})
boy = Boy()
boy.speak()
boy.s_eat() # 调用类中的静态方法
boy.c_run() # 调用类中类方法
print("boy.sex:",boy.sex)
print(type(boy),type(Boy))
'''
这是给类添加的普通方法
这是给类添加的静态方法
这是给类添加的类方法
boy.sex: female
<class '__main__.Boy'> <class 'type'>
'''

3.使用type定义带继承、属性和方法的类
class Person(object):
    def __init__(self,name):
        self.name = name
    def p(self):
        print("这是Person的方法")
class Animal(object):
    def run(self):
        print("animal can run ")
# 定义一个拥有继承的类，继承的效果和性质和class一样。
Worker = type("Worker",(Person,Animal),{"job":"程序员"})
w1 = Worker("tom")
w1.p()
w1.run()
print(type(w1),type(Worker))
'''
这是Person的方法
animal can run 
<class '__main__.Worker'> <class 'type'>
'''
```

### metaclass

- 类的类，元类的实例就是类（不是继承关系），其继承自type，本质是个type（ps:其它定义的类是type类的实例对象）
- 在定义类时可以指定元类来改变类的创建过程。在元类中做一些操作，保证创建的类的可行
- 使用元类的主要目的就是为了实现在创建类时，能够动态地改变类中定义的属性或者方法


```python
# 定义一个元类
class FirstMetaClass(type):
    # cls代表动态修改的类
    # name代表动态修改的类名
    # bases代表被动态修改的类的所有父类
    # attr代表被动态修改的类的所有属性、方法组成的字典 namespace
    def __new__(cls, name, bases, attrs):
        # 动态为该类添加一个name属性
        attrs['name'] = "C语言中文网"
        attrs['say'] = lambda self: print("调用 say() 实例方法")
        return super().__new__(cls,name,bases,attrs)

# 定义类时，指定元类
class CLanguage(object,metaclass=FirstMetaClass):
    pass
clangs = CLanguage() = FirstMetaClass('CLanguage', (), {})()
print(clangs.name)
clangs.say()

class UpperAttrMetaclass(type):
# 有class类A用其做元类的时候，类A得属性就会被全部大写
    def __new__(cls, cls_name, bases, attr_dict):
        uppercase_attr = {}
        for name, val in attr_dict.items():
            if name.startswith('__'):
                uppercase_attr[name] = val
            else:
                uppercase_attr[name.upper()] = val
        return super(UpperAttrMetaclass, cls).__new__(cls, cls_name, bases, uppercase_attr)
        
# 元类的 __call__ 方法可以被用于定制类的实例化过程
class Singleton(type):
    """单例, metaclass=Singleton"""

    def __init__(cls, *args, **kwargs):
        cls.__instance = None
        super().__init__(*args, **kwargs)

    # 用singleton元类来创建Logging类， 当用Logging类来创建对象是，就用的是这个call，就实现了单例
    def __call__(cls, *args, **kwargs):
        if cls.__instance is None:
            cls.__instance = super().__call__(*args, **kwargs)
        if hasattr(cls.__instance, "init"):
            cls.__instance.init(*args, **kwargs)
        return cls.__instance


class Logging(metaclass=Singleton):
    pass

```

# 上下文管理器

用于文件的输入输出、数据库的连接断开等，都是很常见的资源管理操作

任何实现了 enter() 和 exit() 方法的对象都可称之为上下文管理器，上下文管理器对象可以使用 with 关键字。文件对象就是实现了上下文管理器

## 基于类实现上下文

```python
class FileManager:
    def __init__(self, name, mode):
        print('calling __init__ method')
        self.name = name
        self.mode = mode 
        self.file = None
        
    def __enter__(self):
        print('calling __enter__ method')
        self.file = open(self.name, self.mode)
        return self.file


    def __exit__(self, exc_type, exc_val, exc_tb):
        print('calling __exit__ method')
        if self.file:
            self.file.close()
            
with FileManager('test.txt', 'w') as f:
    print('ready to write to file')
    f.write('hello world')

```

## @contextmanager + 生成器实现上下文

@contextmanager + yield

在 contextlib 模块中，提供了 @contextmanager 装饰器，将一个生成器函数当成上下文管理器使用，通过 yield 将函数分割成两部分，yield 之前的语句在 **enter** 方法中执行，yield 之后的语句在 **exit** 方法中执行。紧跟在 yield 后面的值是函数的返回值。

```python
from contextlib import contextmanager

@contextmanager
def atomic():
    print("__enter__")
    a = 1
    try:
        yield a
    finally:
        print("__exit__")


if __name__ == '__main__':
    with atomic() as at:
        print('中间')

__enter__
中间
__exit__
```

# 装饰器

经常用于一些场景，比如：插入日志、性能测试、事物处理、缓存、权限校验等场景。装饰器是解决这些问题的绝佳设计，有了装饰器，我们就可以抽离出大量与函数功能本身无关的雷同代码并继续重用

概括的讲：装饰器的作用就是为了已经存在的对象添加额外功能

- 类似AOP
- **在执行主函数之前，常常要先执行某个预函数**，进行一些校验之类的操作
- 和java的  注解+反射  类似
- 在原函数的基础上对功能进行扩充，并且使得扩充的功能能够以函数的形式返回

## 装饰器类型

- python文件导入即按顺序执行，遇到装饰器就执行，所以外层被卸掉了，就保留wrapper
- 此时原函数不是原函数了，为解决该问题，引入@functools.wrap，它会帮助保留原函数的元信息（也就是将原函数的元信息，拷贝到对应的装饰器函数里）
- 

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print('wrapper of decorator')
        func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name, message):
    ...

greet.__name__
## 输出
'wrapper'

help(greet)
# 输出
Help on function wrapper in module __main__:

wrapper(*args, **kwargs)

import functools

def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print('wrapper of decorator')
        func(*args, **kwargs)
    return wrapper
    
@my_decorator
def greet(message):
    print(message)

greet.__name__

# 输出
'greet'
```

带有自定义参数的装饰器

```python
def repeat(num):
    def my_decorator(func):
        def wrapper(*args, **kwargs):
            for i in range(num):
                print('wrapper of decorator')
                func(*args, **kwargs)
        return wrapper
    return my_decorator


@repeat(4)
def greet(message):
    print(message)

greet('hello world')
```

## 多层装饰器
  
```python
@players_api.route('/<string:activity_id>', methods=['POST'])    #到这里的时候，直接method和url绑定了
@route_param_decode
@auth_user_activity_admin()
@view_resource_wrap
def players(con_func, params, activity_id):      #route_param_decode(auth_user_activity_admin(view_resource_wrap(f)))
  return con_func(activity_id, params)  # 7

def view_resource_wrap(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        name = str2hump(func.__name__)     #5
        entity = getattr(controller, name)
        if not entity:
            raise exce.NoResponseError(msg='内部服务分发错误')

        # con_func / params
        method = request.method.lower()
        params = {}
        status_code = HTTPStatus.OK
        if method == 'get':
            params = request.args.to_dict()
        elif method in ['post', 'put']:
            status_code = HTTPStatus.CREATED
            params = request.get_json()
        elif method == 'delete':
            status_code = HTTPStatus.NO_CONTENT
            params = request.get_json()
        con_func = getattr(entity(), method)
        if not con_func:
            raise exce.NoResponseError(msg='请求的uri资源(%s)不存在' % method)

        rsp = func(con_func, params or {}, *args, **kwargs)       #6 需要f即players()  players(Players.post, ...)     8
        return make_response(jsonify(rsp), status_code)         #9
 
 def auth_user_activity_admin(
        proxy_valid=False, exhibitor_valid=False, corp_valid=False
):
    def decorate(func):
        """
        校验会务前台登录帐户活动管理权限: 1.组织者 2.系统支持
        瑶台内部人员:组织者或系统支持且corp邮箱登录
        """
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # 校验proxy
            if proxy_valid is True and check_proxy():            #3 并开始往下执行
                return func(*args, **kwargs)
            # 从rbac系统中获取token
            token = request.cookies.get(config.TOKEN_COOKIE, None)
            if not token:
                raise exce.Unauthorized(msg='登录失效')  # token不存在，抛出无权限异常
            res, _ = RbacUtils.get_user(token)
            if corp_valid and not res['email'].endswith("corp.netease.com"):
                # 不是瑶台内部用户
                raise exce.Unauthorized(msg='非瑶台内部用户')

            # 登录账号
            g.login_account = res.get('name')

            account = res.get('name')
            if account and account != request.cookies.get('RBAC_USER'):
                raise exce.PermissionDenied()
            user = User.get_valid_user_by_rbac(res)
            ret = AuthUtils.check_user_activity_permission(
                user.id, exhibitor_valid, kwargs
            )
            if not ret:
                raise exce.Unauthorized()
            return func(*args, **kwargs)        #4  需要view_resource_wrap(f)    10
        return wrapper
    return decorate
    
 def route_param_decode(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # 解析route活动id
        if 'activity_id' in kwargs:          #1
            kwargs['activity_id'] = ActivityAseBase().decrypt_id(
                unquote(kwargs['activity_id'], 'utf-8')
            )
        return func(*args, **kwargs)      #2  需要auth_user_activity_admin(view_resource_wrap(f))        11
    return wrapper

```

# 反射

python的反射机制，核心就是利用字符串去已存在的模块中找到指定的属性或方法，找到方法后自动执行——基于字符串的事件驱动。

注：getattr,hasattr,setattr,delattr对模块的修改都在内存中进行，并不会影响文件中真实内容。

```python
class Fruit:
    # 构造方法
    def __init__(self,name,color):
        self.name = name
        self.color = color
    # 类的普通方法
    def buy(self,price,num):
        print("水果的价格是：",price*num)
"""
    hasattr(object,'attrName'):判断该对象是否有指定名字的属性或方法，返回值是bool类型
    setattr(object,'attrName',value):给指定的对象添加属性以及属性值
    getattr(object,'attrName'):获取对象指定名称的属性或方法，返回值是str类型
    delattr(object,'attrName'):删除对象指定名称的属性或方法值，无返回值
"""       
apple = Fruit("苹果","红色")
print(hasattr(apple,'name')) # 判断对象是否有该属性或方法
print(hasattr(apple,'buy'))

# 获取对象指定的属性值
print(getattr(apple,'name'))
print(apple.name)

f = getattr(apple,'buy')
f(5,10)
# 设置对象对应的属性
setattr(apple,'weight',100)

# 删除对象对应的属性
delattr(apple,'name')
print(hasattr(apple,'name'))
```

