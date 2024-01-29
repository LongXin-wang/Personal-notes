- [轻量级异步任务队列](#轻量级异步任务队列)
  - [任务类型](#任务类型)
      - [当我们调用任务函数时会发生什么？](#当我们调用任务函数时会发生什么)
  - [任务优先级](#任务优先级)
  - [启动](#启动)
  - [任务导入方式](#任务导入方式)
  - [启动](#启动-1)
  - [任务导入方式](#任务导入方式-1)
  - [信号](#信号)
  - [管理共享资源](#管理共享资源)
    - [执行前和执行后挂钩](#执行前和执行后挂钩)


# 轻量级异步任务队列

https://huey.readthedocs.io/en/latest/guide.html

## 任务类型

```python
Huey.task()——装饰器来指示可执行任务
Huey.periodic_task() ——装饰器以指示以周期性间隔执行的任务
TaskResultWrapper.get() ——从任务获取返回值
crontab() ——用于定义执行周期性任务的间隔时间
```

```python
from huey import RedisHuey, crontab

huey = RedisHuey('my-app', host='redis.myapp.com')
'''
Whenever the add_numbers() function is called, the actual execution will occur when the consumer dequeues the message.
'''
@huey.task()
def add_numbers(a, b):
    return a + b

# 该任务将在失败时重试两次，每次重试之间间隔60秒。
@huey.task(retries=2, retry_delay=60)
def flaky_task(url):
    # This task might fail, in which case it will be retried up to 2 times
    # with a delay of 60s between retries.
    return this_might_fail(url)

# 该任务将在每天凌晨3点执行，因为它使用 crontab 格式的时间表达式，其中 minute='0', hour='3' 表示每天的凌晨3点
@huey.periodic_task(crontab(minute='0', hour='3'))
def nightly_backup():
    sync_all_data()
    

>>> from demo import add_numbers
>>> res = add_numbers(1, 2)
>>> res
<Result: task 6b6f36fc-da0d-4069-b46c-c0d4ccff1df6>

>>> res()
3
```

#### 当我们调用任务函数时会发生什么？

1. 当`add_numbers()`调用该函数时，表示该调用的消息将被放入队列中。
2. 该函数立即返回而不实际运行，并返回一个result句柄，一旦消费者执行了任务，该句柄可用于检索结果。
3. 消费者进程看到消息已到达，工作人员将调用该`add_numbers()`函数并将返回值放入结果存储中。
4. 我们可以使用result句柄从结果存储中读取返回值。

## 任务优先级

要在 Redis 中使用任务优先级，请使用PriorityRedisHuey代替 RedisHuey

现在待处理任务的队列将是：

- `process_payment`- 优先级 = 50
- `send_email`- 优先级 = 10
- `check_spam`- 优先级 = 1
- `make_thumbnail`- 优先级 = 0

```Python
@huey.task(priority=10)
def send_email(to, subj, body):
    return mailer.send(to, 'webmaster@myapp.com', subj, body)
```

## 启动 

```Python
huey_consumer.py my_app.huey -k process -w 4   #开启4个进程处理    -k表示工作模式 -w/--works 线程/进程/协程数量     -s,--scheduler-interval 调度频率

huey_consumer.py my_app.huey -k greenlet -w 32   #开启协程处理模式

huey_consumer.py my_app.huey（消费者） -w 4   #默认使用线程开启，4个线程  cpu密集没什么用。IO密集用协程

```

## 任务导入方式

要在 Redis 中使用任务优先级，请使用PriorityRedisHuey代替 RedisHuey

现在待处理任务的队列将是：

- `process_payment`- 优先级 = 50
- `send_email`- 优先级 = 10
- `check_spam`- 优先级 = 1
- `make_thumbnail`- 优先级 = 0

```Python
@huey.task(priority=10)
def send_email(to, subj, body):
    return mailer.send(to, 'webmaster@myapp.com', subj, body)
```

## 启动 

```Python
huey_consumer.py my_app.huey -k process -w 4   #开启4个进程处理    -k表示工作模式 -w/--works 线程/进程/协程数量     -s,--scheduler-interval 调度频率

huey_consumer.py my_app.huey -k greenlet -w 32   #开启协程处理模式

huey_consumer.py my_app.huey（消费者） -w 4   #默认使用线程开启，4个线程  cpu密集没什么用。IO密集用协程

```

## 任务导入方式

当您使用task()or 装饰函数时periodic_task()，该函数会在内存中的注册表中注册自己。当一个任务函数被调用时，一个引用被放入队列，连同函数被调用的参数等等。然后消费者读取消息，并在消费者的注册表中查找任务函数。

一般来说，我构造这样的东西，这使得很容易避免循环导入。


- `config.py`，包含该[`Huey`](https://huey.readthedocs.io/en/latest/api.html#Huey)对象的模块。

```Python
# config.py
from huey import RedisHuey

huey = RedisHuey('testing')
```

`tasks.py`，包含任何修饰函数的模块。`huey`从模块导入 对象`config.py`：

```Python
# tasks.py
from config import huey

@huey.task()
def add(a, b):
    return a + b
```

`main.py`/ `app.py`，“主”模块。导入`config.py` 模块**和**模块`tasks.py`

```Python
# main.py
from config import huey  # import the "huey" object.
from tasks import add  # import any tasks / decorated functions


if __name__ == '__main__':
    result = add(1, 2)
    print('1 + 2 = %s' % result.get(blocking=True))
```

要运行消费者，请将其指向`main.huey`，这样`huey` 实例**和**任务函数都会在集中位置导入。

```Python
huey_consumer.py main.huey
```

## 信号

https://huey.readthedocs.io/en/latest/signals.html

消费者在处理任务时会发送各种信号。回调可以注册为信号处理程序，并由消费者进程同步调用。信号处理程序由消费者在处理任务时**同步**执行。在实现信号处理程序时要小心，这一点很重要，因为一个缓慢的信号处理程序可能会影响消费者的整体响应能力。

Huey 实现了以下信号：

- `SIGNAL_CANCELED`：由于预执行挂钩引发CancelExecution异常，任务被取消。
- `SIGNAL_COMPLETE`: 任务已成功执行。
- `SIGNAL_ERROR`: 由于未处理的异常，任务失败。
- `SIGNAL_EXECUTING`: 任务即将执行。
- `SIGNAL_EXPIRED`: 任务已过期。
- `SIGNAL_LOCKED`: 获取锁失败，中止任务。
- `SIGNAL_RETRYING`: 任务失败，但会重试。
- `SIGNAL_REVOKED`: 任务被撤销，不会被执行。
- `SIGNAL_SCHEDULED`：任务尚未准备好运行，已添加到计划中以供将来执行。
- `SIGNAL_INTERRUPTED`：当消费者退出时任务被中断。

要注册信号处理程序，请使用以下Huey.signal()方法：

```Python
@huey.signal()
def all_signal_handler(signal, task, exc=None):
    # This handler will be called for every signal.
    print('%s - %s' % (signal, task.id))

@huey.signal(SIGNAL_ERROR, SIGNAL_LOCKED, SIGNAL_CANCELED, SIGNAL_REVOKED)
def task_not_executed_handler(signal, task, exc=None):
    # This handler will be called for the 4 signals listed, which
    # correspond to error conditions.
    print('[%s] %s - not executed' % (signal, task.id))

@huey.signal(SIGNAL_COMPLETE)
def task_success(signal, task):
    # This handle will be called for each task that completes successfully.
    pass
```

信号是在任务被执行的同时根据当前任务的状态执行的

```Python
>>> result = add.schedule((2, 3), delay=10)
>>> result(True)  # same as result.get(blocking=True)
将发送以下信号：
SIGNAL_SCHEDULED- 该任务尚未准备好运行，因此已将其添加到计划中。
10秒后，消费者将运行任务并发送信号SIGNAL_EXECUTING。
SIGNAL_COMPLETE。

>>> result = flaky_task()
>>> try:
...     result.get(blocking=True)
... except TaskException:
...     result.reset()
...     result.get(blocking=True)  # Try again if first time fails.
假设任务第一次失败而第二次成功，我们将看到发送以下信号：

SIGNAL_EXECUTING- 任务正在执行。
SIGNAL_ERROR- 任务引发了未处理的异常。
SIGNAL_RETRYING- 该任务将被重试。
SIGNAL_SCHEDULED- 任务已添加到计划中，以便在大约 10 秒内执行。
SIGNAL_EXECUTING- 第二次尝试运行任务。
SIGNAL_COMPLETE- 任务成功。

```

## 管理共享资源

任务可能需要使用应用程序的共享资源，例如数据库连接或 API 客户端。

装饰Huey.on_startup()器用于注册一个回调，该回调在每个工作程序开始运行时执行一次。此挂钩提供了一种便捷的方法来初始化共享资源或执行应在工作线程或进程的上下文中发生的其他初始化。

```Python
import peewee

db = PostgresqlDatabase('my_app')

@huey.on_startup()
def open_db_connection():
    # If for some reason the db connection appears to already be open,
    # close it first.
    if not db.is_closed():
        db.close()
    db.connect()

@huey.task()
def run_query(n):
    db.execute_sql('select pg_sleep(%s)', (n,))
    return n
```

### 执行前和执行后挂钩

```Python
from huey import CancelExecution

@huey.pre_execute()
def pre_execute_hook(task):
    # Pre-execute hooks are passed the task that is about to be run.

    # This pre-execute task will cancel the execution of every task if the
    # current day is Sunday.
    if datetime.datetime.now().weekday() == 6:
        raise CancelExecution('No tasks on sunday!')

@huey.post_execute()
def post_execute_hook(task, task_value, exc):
# 处理函数应该接受三个参数：已执行的任务、返回值和异常（如果发生，则为None）。
    # Post-execute hooks are passed the task, the return value (if the task
    # succeeded), and the exception (if one occurred).
    if exc is not None:
        print('Task "%s" failed with error: %s!' % (task.id, exc))
```

