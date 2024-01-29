- [类型注解](#类型注解)
  - [类型别名](#类型别名)
  - [Callable](#callable)
  - [TypeVar \& Generic](#typevar--generic)
    - [泛型函数](#泛型函数)
    - [泛型类](#泛型类)
  - [NewType](#newtype)

# 类型注解

## 类型别名

类型别名通过将类型分配给别名来定义。在这个例子中， Vector 和 List[float] 将被视为可互换的同义词

```python
from typing import List
Vector = List[float]

def scale(scalar: float, vector: Vector) -> Vector:
    return [scalar * num for num in vector]

# typechecks; a list of floats qualifies as a Vector.
new_vector = scale(2.0, [1.0, -4.2, 5.4])

from typing import Dict, Tuple, Sequence

#不是实例化，进去看源码就知道了；不是括号，是中括号呀
ConnectionOptions = Dict[str, str]
Address = Tuple[str, int]
Server = Tuple[Address, ConnectionOptions]

def broadcast_message(message: str, servers: Sequence[Server]) -> None:
    ...

# The static type checker will treat the previous type signature as
# being exactly equivalent to this one.
def broadcast_message(
        message: str,
        servers: Sequence[Tuple[Tuple[str, int], Dict[str, str]]]) -> None:
    ...
```


## Callable

只要可以在一个对象的后面使用小括号来执行代码，那么这个对象就是callable对象：以下都是callable对象

1. 函数   可以被调用，是callable对象
2. 类     可以被调用，是callable对象
3. 类里的函数      可以被调用，是callable对象
4. **实现了__call__方法的实例对象**       实例对象可以被调用，是callable对象

Callable 作为函数参数使用，其实只是做一个类型检查的作用，检查传入的参数值 get_func 是否为可调用对象

Callable  作为函数返回值使用，其实只是做一个类型检查的作用，看看返回值是否为可调用对象

```python
def date(year: int, month: int, day: int) -> str:
 return f'{year}-{month}-{day}'
 
def get_date_fn() -> Callable[[int, int, int], str]:     返回值date是否是可调用对象，date就是上面那个发那个方法
 return date
 
Callable type; Callable[[int], str] is a function of (int) -> str.
第一个类型(int)代表参数类型
第二个类型(str)代表返回值类型

# Callable 作为函数参数使用，其实只是做一个类型检查的作用，检查传入的参数值 get_func 是否为可调用对象
def get_name(get_func: Callable[[str], None]):
    return get_func

# Callable  作为函数返回值使用，其实只是做一个类型检查的作用，看看返回值是否为可调用对象
def get_name_return() -> Callable[[str], None]:
    return print_name
 
 
vars = get_name_return()
vars("test")
```

## TypeVar & Generic

TypeVar T 是一个泛型类型变量，它在函数签名中起到了占位符的作用，表示实际传入的类型。

Generic 是一个泛型基类，用于创建泛型类。通过继承 Generic 类，可以在类定义中使用类型变量。它通常与 TypeVar 一起使用，用于指定类的参数化类型。

```python
from typing import TypeVar

T = TypeVar('T')  # 创建一个泛型类型变量

def first_item(items: list[T]) -> T:  # 使用泛型类型变量
    return items[0]


from typing import TypeVar, Generic

T = TypeVar('T')

class Box(Generic[T]):
    def __init__(self, content: T):
        self.content = content

box = Box[int](10)  # 创建一个包含整数的 Box 实例
```

### 泛型函数

```python
from typing import TypeVar, List, Union

# 通过TypeVar限定为整数型的列表和浮点数的列表
T = TypeVar("T", bound=Union[List[int], List[float]])
# 也可以写成如下形式
T = TypeVar("T", List[int], List[float])

def printList(l: T):
    for e in l:
        print(e)

printList([1, 2, 3])           # 打印整数型列表
printList([1.1, 2.2, 3.3])     # 打印浮点数列表
printList(["a", "b", "c"])     # 字符串型列表，mypy会报错
```

```java
public class MyClass {
   
   // 以下就是一个泛型函数
   public static < T > void printArray(T[] inputArray) {
         for ( T element: inputArray ){        
            System.out.printf( "%s ", element );
         }
    }

    public static void main(String args[]) {
        // 创建两种不同类型的数组： Integer, Double
        Integer[] intArray = { 1, 2, 3, 4, 5 }; // 注意这里要用Integer，不能用int
        Double[] doubleArray = { 1.1, 2.2, 3.3, 4.4 }; // 注意这里要用Double，不能用double
 
        printArray( intArray  ); // 传递一个int型数组，可以！没问题！
        printArray( doubleArray ); // 传递一个double型数组，也可以！不会报错！
    }
}
```

### 泛型类

```python
from typing import TypeVar, Union, Generic, List

# 通过TypeVar限定为整数型的列表和浮点数的列表
T = TypeVar("T", bound=Union[int, float])

class MyList(Generic[T]):

    def __init__(self, size: int) -> None:
        self.size = size
        self.list: List[T] = []
    
    def append(self, e: T):
        self.list.append(e)
        print(self.list)

print(type(MyList))   # <class 'type'>

int_list = MyList[int]

print(type(int_list))   # <class 'typing._GenericAlias'>
print(isinstance(int_list, MyList)) # False 表明不是MyList的实例
print(isinstance(int_list, type)) # False 表明不是type的实例

# 适用于整数型 
# Mylis[int] 表示一个泛型类Mylist 的特定版本，其中泛型类型参数 T 被指定为 int。这意味着 Mylist[int] 是一个能够容纳整数类型对象的版本。 Mylist[int] 不是一个类，是一个泛型类的具体化，是一个泛型别名，不是一个传统的类， 

intList = MyList[int](3)    # 通过[int]进行类型提示！
intList.append(101)

# 也适用于浮点数
floatList = MyList[float](3)
floatList.append(1.1)

# 但不适用于字符串，以下代码通过mypy检查会报错！
strList = MyList[str](3)
strList.append("test")


from typing import TypeVar, Generic

T = TypeVar('T')

class Box(Generic[T]):
    def __init__(self, content: T):
        self.content = content

class IntBox(Box[int]):
    def double_content(self):
        return self.content * 2

int_box = IntBox(5)
print(int_box.double_content())  # 输出: 10
```

```java
public class MyArray<T> {
    int size;
    T[] array;

    public MyArray (Integer size) {
        this.size = size;
        this.array = (T[]) new Object[size];
        // 注意这里不能直接写 this.array = T[size]
    }

    public void add(T e, Integer i) {
        this.array[i] = e;
        System.out.printf( "%s ", this.array );
    }

    public static void main(String args[]) {
        // 适用于Integer型
        MyArray intArray = new MyArray<Integer>(3);
        intArray.add(101, 0);

        // 同样也适用于Double类型
        MyArray doubleArray = new MyArray<Double>(3);
        doubleArray.add(1.1, 0);
    }
}
```


## NewType

使用 NewType()辅助函数创建不同的类型:

- NewType(name, tp) 返回一个函数，这个函数返回其原本的值; 创建一个新类型，其类型和tp一样
- 静态类型检查器会将新类型看作是原始类型的一个子类
- 本质就是利用type创建类

```python
from typing import NewType

UserId = NewType('UserId', int)
some_id = UserId(524313)

UserId('user')  # Fails type check
num = UserId(5)  # type: int

print(type(UserId(5)))

```