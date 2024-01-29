- [async 创建协程](#async-创建协程)
  - [asyncio 几个概念](#asyncio-几个概念)
  - [绑定回调函数](#绑定回调函数)
- [协程并发](#协程并发)
  - [gevent 协程并发](#gevent-协程并发)
  - [asyncio 协程并发](#asyncio-协程并发)
- [动态添加协程](#动态添加协程)
  - [动态添加](#动态添加)
  - [利用redis实现动态添加任务](#利用redis实现动态添加任务)

# async 创建协程

**使用async可以定义协程，协程用于耗时的io操作；嵌套协程，即一个协程中await了另外一个协程，如此连接起来**

```Python
'''async关键字'''
from collections.abc import Coroutine

async def hello(name):
    print('Hello,', name)

if __name__ == '__main__':
    # 生成协程对象，并不会运行函数内的代码
    coroutine = hello("World")

    # 检查是否是协程 Coroutine 类型
    print(isinstance(coroutine, Coroutine))  # True

'''@asyncio.coroutine 装饰器标记为协程'''
import asyncio
from collections.abc import Generator, Coroutine

@asyncio.coroutine
def hello():
    # 异步调用asyncio.sleep(1):
    yield from asyncio.sleep(1)


if __name__ == '__main__':
    coroutine = hello()
    print(isinstance(coroutine, Generator))  # True
    print(isinstance(coroutine, Coroutine))  # False
```

## asyncio 几个概念

* event_loop 事件循环：程序开启一个无限的循环，程序员会把一些函数（协程）注册到事件循环上。当满足事件发生的时候，调用相应的协程函数。

* coroutine 协程：协程对象，指一个使用async关键字定义的函数，它的调用不会立即执行函数，而是会返回一个协程对象。协程对象需要注册到事件循环，由事件循环调用。

* future 对象： 代表将来执行或没有执行的任务的结果。它和task没有本质的区别

* task 任务：一个协程对象就是一个原生可以挂起的函数，任务则是对协程进一步封装，其中包含任务的各种状态。Task 对象是 Future 的子类，它将 coroutine 和 Future 联系在一起，将 coroutine 封装成一个 Future 对象。

* async/await 关键字：python3.5 用于定义协程的关键字，async定义一个协程，await用于挂起阻塞的异步调用接口。其作用在一定程度上类似于yield。

```python
import asyncio

async def hello(name):
    print('Hello,', name)

# 定义协程对象
coroutine = hello("World")

# 将协程转为task任务(future对象)
task = asyncio.ensure_future(coroutine)

# 定义事件循环对象容器
loop = asyncio.get_event_loop()
# task = loop.create_task(coroutine)

# 将task任务扔进事件循环对象中并触发
loop.run_until_complete(task)


# 定义协程对象
coroutine = hello("World")

# 定义事件循环对象容器
loop = asyncio.get_event_loop()
# 利用loop创建task
task = loop.create_task(coroutine)

# 将task任务扔进事件循环对象中并触发
loop.run_until_complete(task)
```

## 绑定回调函数

**取得协程的await的返回值**
```python
import asyncio
import time

async def _sleep(x):
    time.sleep(2)
    return '暂停了{}秒！'.format(x)


coroutine = _sleep(2)
loop = asyncio.get_event_loop()

task = asyncio.ensure_future(coroutine)
loop.run_until_complete(task)

# task.result() 可以取得返回结果
print('返回结果：{}'.format(task.result()))
```

**asyncio自带的添加回调函数功能**
```python
import time
import asyncio


async def _sleep(x):
    time.sleep(2)
    return '暂停了{}秒！'.format(x)

def callback(future):
    print('这里是回调函数，获取返回结果是：', future.result())

coroutine = _sleep(2)
loop = asyncio.get_event_loop()
task = asyncio.ensure_future(coroutine)

# 添加回调函数
task.add_done_callback(callback)

loop.run_until_complete(task)
```

# 协程并发

## gevent 协程并发

```python
jobs = []
server_list = server_list if server_list is not None else self.get_server_list()
for game_server_base in server_list:
    jobs.append(
        gevent.spawn(
            self._request_post,
            f"{game_server_base}{path}",
            params, timeout, err_msg))
gevent.joinall(jobs, timeout=timeout)
for job in jobs:
    if job.successful() is False:
        # 任务失败, 若出现异常, 则抛出异常
        if job.exception:
            raise job.exception
        else:
            raise GameServerServiceError(msg='游戏服务端通知失败！')
```

## asyncio 协程并发

- asyncio.wait接受list对象，list对象可以是转换后的task任务，也可以不转task任务，是协程即可（此时无法获取task的result）
- asyncio.wait 返回dones和pendings - dones：表示已经完成的任务 - pendings：表示未完成的任务

```python
loop = asyncio.get_event_loop()

tasks=[
       factorial("A", 2),
       factorial("B", 3),
       factorial("C", 4)
]

loop.run_until_complete(asyncio.wait(tasks))


tasks=[
       asyncio.ensure_future(factorial("A", 2)),
       asyncio.ensure_future(factorial("B", 3)),
       asyncio.ensure_future(factorial("C", 4))
]

loop = asyncio.get_event_loop()

loop.run_until_complete(asyncio.gather(*tasks))
```

- asyncio.gather接受比较广泛
- asyncio.gather 它会把值直接返回给我们，不需要手工去收集

```python
loop = asyncio.get_event_loop()

loop.run_until_complete(asyncio.gather(
    factorial("A", 2),
    factorial("B", 3),
    factorial("C", 4),
))
```

```python
import asyncio

# 协程函数
async def do_some_work(x):
    print('Waiting: ', x)
    await asyncio.sleep(x)
    return 'Done after {}s'.format(x)

# 协程对象
coroutine1 = do_some_work(1)
coroutine2 = do_some_work(2)
coroutine3 = do_some_work(4)

# 将协程转成task，并组成list
tasks = [
    asyncio.ensure_future(coroutine1),
    asyncio.ensure_future(coroutine2),
    asyncio.ensure_future(coroutine3)
]

loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait(tasks))

for task in tasks:
    print('Task ret: ', task.result())
```

```python
import asyncio

# 用于内部的协程函数
async def do_some_work(x):
    print('Waiting: ', x)
    await asyncio.sleep(x)
    return 'Done after {}s'.format(x)

# 外部的协程函数
async def main():
    # 创建三个协程对象
    coroutine1 = do_some_work(1)
    coroutine2 = do_some_work(2)
    coroutine3 = do_some_work(4)

    # 将协程转为task，并组成list
    tasks = [
        asyncio.ensure_future(coroutine1),
        asyncio.ensure_future(coroutine2),
        asyncio.ensure_future(coroutine3)
    ]

    # 【重点】：await 一个task列表（协程）
    # dones：表示已经完成的任务
    # pendings：表示未完成的任务
    dones, pendings = await asyncio.wait(tasks)

    for task in dones:
        print('Task ret: ', task.result())

    # results = await asyncio.gather(*tasks)
    # for result in results:
    #     print('Task ret: ', result)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

# 动态添加协程

## 动态添加

**主线程是同步的**

```python
import time
import asyncio
from queue import Queue
from threading import Thread

def start_loop(loop):
    # 一个在后台永远运行的事件循环
    asyncio.set_event_loop(loop)
    loop.run_forever()

def do_sleep(x, queue, msg=""):
    time.sleep(x)
    queue.put(msg)

queue = Queue()

new_loop = asyncio.new_event_loop()

# 定义一个线程，并传入一个事件循环对象
t = Thread(target=start_loop, args=(new_loop,))
t.start()

print(time.ctime())

# 动态添加两个协程
# 这种方法，在主线程是同步的
new_loop.call_soon_threadsafe(do_sleep, 6, queue, "第一个")
new_loop.call_soon_threadsafe(do_sleep, 3, queue, "第二个")

while True:
    msg = queue.get()
    print("{} 协程运行完..".format(msg))
    print(time.ctime())

由于是同步的，所以总共耗时6+3=9秒.

Thu May 31 22:11:16 2018
第一个 协程运行完..
Thu May 31 22:11:22 2018
第二个 协程运行完..
Thu May 31 22:11:25 2018
```

**主线程是异步的**

```python
import time
import asyncio
from queue import Queue
from threading import Thread

def start_loop(loop):
    # 一个在后台永远运行的事件循环
    asyncio.set_event_loop(loop)
    loop.run_forever()

async def do_sleep(x, queue, msg=""):
    await asyncio.sleep(x)
    queue.put(msg)

queue = Queue()

new_loop = asyncio.new_event_loop()

# 定义一个线程，并传入一个事件循环对象
t = Thread(target=start_loop, args=(new_loop,))
t.start()

print(time.ctime())

# 动态添加两个协程
# 这种方法，在主线程是异步的
asyncio.run_coroutine_threadsafe(do_sleep(6, queue, "第一个"), new_loop)
asyncio.run_coroutine_threadsafe(do_sleep(3, queue, "第二个"), new_loop)

while True:
    msg = queue.get()
    print("{} 协程运行完..".format(msg))
    print(time.ctime())

由于是异步的，所以总共耗时max(6, 3)=6秒

Thu May 31 22:23:35 2018
第二个 协程运行完..
Thu May 31 22:23:38 2018
第一个 协程运行完..
Thu May 31 22:23:41 2018
```

## 利用redis实现动态添加任务

```python
import time
import redis
import asyncio
from queue import Queue
from threading import Thread

def start_loop(loop):
    # 一个在后台永远运行的事件循环
    asyncio.set_event_loop(loop)
    loop.run_forever()

async def do_sleep(x, queue):
    await asyncio.sleep(x)
    queue.put("ok")

def get_redis():
    connection_pool = redis.ConnectionPool(host='127.0.0.1', db=0)
    return redis.Redis(connection_pool=connection_pool)

def consumer():
    while True:
        task = rcon.rpop("queue")
        if not task:
            time.sleep(1)
            continue
        asyncio.run_coroutine_threadsafe(do_sleep(int(task), queue), new_loop)


if __name__ == '__main__':
    print(time.ctime())
    new_loop = asyncio.new_event_loop()

    # 定义一个线程，运行一个事件循环对象，用于实时接收新任务
    loop_thread = Thread(target=start_loop, args=(new_loop,))
    loop_thread.setDaemon(True)
    loop_thread.start()
    # 创建redis连接
    rcon = get_redis()

    queue = Queue()

    # 子线程：用于消费队列消息，并实时往事件对象容器中添加新任务
    consumer_thread = Thread(target=consumer)
    consumer_thread.setDaemon(True)
    consumer_thread.start()

    while True:
        msg = queue.get()
        print("协程运行完..")
        print("当前时间：", time.ctime())
```