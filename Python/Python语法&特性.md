- [标准数据类型](#标准数据类型)
  - [列表\&元组](#列表元组)
  - [具名元组](#具名元组)
  - [字典和集合](#字典和集合)
    - [集合](#集合)
    - [字典](#字典)
  - [字符串](#字符串)
    - [字符串常用操作](#字符串常用操作)
    - [字符串格式化](#字符串格式化)
- [函数](#函数)
  - [值传递vs引用传递](#值传递vs引用传递)
  - [传参](#传参)
  - [自定义函数](#自定义函数)
    - [嵌套函数](#嵌套函数)
    - [闭包](#闭包)
  - [匿名函数](#匿名函数)
- [JSON](#json)
- [迭代器\&生成器](#迭代器生成器)
  - [迭代器](#迭代器)
  - [生成器](#生成器)
    - [列表解析](#列表解析)
    - [生成器](#生成器-1)
- [类](#类)
  - [init new call](#init-new-call)
  - [静态方法、类方法、实例方法](#静态方法类方法实例方法)
  - [\_\_slots\_\_魔法](#__slots__魔法)
- [上下文管理器](#上下文管理器)
  - [基于类实现上下文](#基于类实现上下文)
  - [@contextmanager + 生成器实现上下文](#contextmanager--生成器实现上下文)

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
for key,value in dict.items () 
```

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
def nth_power(exponent):
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

