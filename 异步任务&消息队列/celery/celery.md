- [基础知识](#基础知识)
  - [使用场景](#使用场景)
  - [使用](#使用)
    - [任务启动](#任务启动)
- [celery原理](#celery原理)
  - [redis中的异步任务序列化信息](#redis中的异步任务序列化信息)
  - [交换机/路由](#交换机路由)
  - [调用任务](#调用任务)
  - [任务重试](#任务重试)
  - [周期任务](#周期任务)

# 基础知识

- 使用异步任务框架实现异步任务的处理  生产者—消费者模式
- 区别异步编程，协程还是在主线程中并发运行
- 协程数量太多，消耗太多资源，可考虑使用异步框架处理

各部分介绍：

1. celery应用： 用户编写的代码脚本，用来定义要执行的任务，然后通过 broker 将任务发送到消息队列中
2. broker(经纪人)：Celery本身不提供消息服务，但是可以方便的和第三方提供的消息中间件集成。包括，RabbitMQ, Redis等等
3. backend：任务返回结果存储
4. worker： 任务执行单元，worker并发的运行在分布式的系统节点中

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402261101446.png)

1）可以不依赖任何服务器，通过自身命令，启动服务(内部支持socket)
2）celery服务为为其他项目服务提供异步解决任务需求的
注：会有两个服务同时运行，一个是项目服务，一个是celery服务，项目服务将需要异步处理的任务交给celery服务，celery就会在需要时异步完成项目的需求

## 使用场景

异步执行：解决耗时任务

延迟执行：解决延迟任务

定时执行：解决周期(周期)任务

## 使用

如果 Celery对象:Celery(...) 是放在一个包下的 

1）必须在这个包下建一个celery.py的文件，将Celery(...)产生对象的语句放在该文件中 

2）执行启动worker的命令：celery worker -A 包名 -l info 

```text
project
    ├── celery_task    # celery包
    │   ├── __init__.py # 包文件
    │   ├── celery.py   # celery连接和配置相关文件，且名字必须交celery.py
    │   └── tasks.py    # 所有任务函数
    ├── add_task.py    # 添加任务
    └── get_result.py   # 获取结果
```

```Python
# 安装执行
pip install celery
pip install -U "celery[redis]"
```

### 任务启动

任务启动分为worker启动和定时任务beat启动

- 定时任务beat启动只是启动定时任务去将任务发送到broker，还是需要worker来消费的

```Python
celery -A celery_task worker -l info

celery -A celery_task beat -l info

```

# celery原理

- 客户端调用异步任务，异步任务发送至redis，redis模拟队列，celery-worker消耗队列中的tasks
- 执行结果可以放到redis等broker中，也可以打日志等
- 开启多个worker处理不同的队列，例如，异步任务一个worker，延迟任务一worker，定时任务一个worker；
- 每个worker可处理多个队列，每个队列中的任务还可以分优先级
- 任务-路由-队列：通过路由键和tasks 将任务路由到对应的队列，然后队列由worker执行
-  队列和交换机的映射（交换机名+交换类型+路由键）

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402261108940.png)

## redis中的异步任务序列化信息

- 指定需要调用的为celery_task.tasks.add，参数为"argsrepr":"(10, 20)",
- 指定访问地址"origin":"gen28563@wanglongxindeMacBook-Pro.local",
- 指定路由键，路由键是celery
- 然后就根据这个指定的路由键去celery应用中找到相对应的队列

```Python
{
    "body":"W1sxMCwgMjBdLCB7fSwgeyJjYWxsYmFja3MiOiBudWxsLCAiZXJyYmFja3MiOiBudWxsLCAiY2hhaW4iOiBudWxsLCAiY2hvcmQiOiBudWxsfV0=",
    "content-encoding":"utf-8",
    "content-type":"application/json",
    "headers":{
        "lang":"py",
        "task":"celery_task.tasks.add",
        "id":"adc9f7ea-eafe-4c12-b5d8-a19ef4d26583",
        "shadow":null,
        "eta":null,
        "expires":null,
        "group":null,
        "group_index":null,
        "retries":0,
        "timelimit":[
            null,
            null
        ],
        "root_id":"adc9f7ea-eafe-4c12-b5d8-a19ef4d26583",
        "parent_id":null,
        "argsrepr":"(10, 20)",
        "kwargsrepr":"{}",
        "origin":"gen28563@wanglongxindeMacBook-Pro.local",
        "ignore_result":false,
        "stamped_headers":null,
        "stamps":{

        }
    },
    "properties":{
        "correlation_id":"adc9f7ea-eafe-4c12-b5d8-a19ef4d26583",
        "reply_to":"30348596-17d8-370f-83e5-5b135bd94400",
        "delivery_mode":2,
        "delivery_info":{
            "exchange":"",
            "routing_key":"celery"
        },
        "priority":0,
        "body_encoding":"base64",
        "delivery_tag":"79db3684-1bd3-48b3-af4f-e1f218d559c8"
    }
}
```

## 交换机/路由

```Python
Celery支持以下几种交换机类型：

direct（默认类型）：直接匹配，把消息路由到那些binding key与routing key完全匹配的Queue中。
fanout：广播形式，它没有参数绑定，就是不需要指定routing_key之类的东西，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中去。
topic：主题匹配，它与direct类型的Exchage相似，但是有不同之处。它引入了两个通配符#和*，前者匹配多个单词（可以为0），后者匹配一个单词。在绑定topic类型的Exchange与Queue时，需要指定一个binding key，它是一个带有通配符的字符串。
```

```Python
# 设置默认的队列名称，如果一个消息不符合其他的队列就会放在默认队列里面，如果什么都不设置的话，数据都会发送到默认的队列中
task_default_queue = "default"
# 设置详细的队列
task_queues = {
    Queue('asyn', Exchange('asyn', type='direct'), routing_key='asyn'),
    Queue('perioc', Exchange('perioc', type='direct'), routing_key='perioc'),
    Queue('delay', Exchange('delay', type='direct'), routing_key='delay')
}

# #设置任务-路由-队列
task_routes = {
    # 这个的路径取决于运行task的路径
    'celery_task.tasks.add':{
        'queue': 'asyn',
        'routing_key': 'asyn' #redis没有强匹配
     },

    'celery_task.tasks.multi': {
        'queue': 'delay',
        'routing_key': 'delay'
     }

}


```

## 调用任务

```Python
apply_async(args[, kwargs[, …]])
Sends a task message.

delay(*args, **kwargs)
Shortcut to send a task message, but doesn’t support execution options.

calling (__call__)
Applying an object supporting the calling API (e.g., add(2, 2)) means that the task will not be executed by a worker, but in the current process instead (a message won’t be sent).



T.delay(arg, kwarg=value)
Star arguments shortcut to .apply_async. (.delay(*args, **kwargs) calls .apply_async(args, kwargs)).

T.apply_async((arg,), {'kwarg': value})

T.apply_async(countdown=10)
executes in 10 seconds from now.

T.apply_async(eta=now + timedelta(seconds=10))
executes in 10 seconds from now, specified using eta

T.apply_async(countdown=60, expires=120)
executes in one minute from now, but expires after 2 minutes.

T.apply_async(expires=now + timedelta(days=2))
expires in 2 days, set using datetime.

```

## 任务重试

```Python
from kombu.exceptions import TimeoutError

# 重试超时任务
add.apply_async((2, 2), retry=True, retry_policy={
    'max_retries': 3,
    'retry_errors': (TimeoutError, ),
})
```

## 周期任务

```Python
from celery import Celery
from celery.schedules import crontab

app = Celery()

@app.on_after_configure.connect
def setup_periodic_tasks(sender, **kwargs):
    # Calls test('hello') every 10 seconds.
    sender.add_periodic_task(10.0, test.s('hello'), name='add every 10')

    # Calls test('hello') every 30 seconds.
    # It uses the same signature of previous task, an explicit name is
    # defined to avoid this task replacing the previous one defined.
    sender.add_periodic_task(30.0, test.s('hello'), name='add every 30')

    # Calls test('world') every 30 seconds
    sender.add_periodic_task(30.0, test.s('world'), expires=10)

    # Executes every Monday morning at 7:30 a.m.
    sender.add_periodic_task(
        crontab(hour=7, minute=30, day_of_week=1),
        test.s('Happy Mondays!'),
    )

@app.task
def test(arg):
    print(arg)

@app.task
def add(x, y):
    z = x + y
    print(z)
```

该[`add_periodic_task()`](https://docs.celeryq.dev/en/stable/reference/celery.html#celery.Celery.add_periodic_task)函数会将条目添加到 [`beat_schedule`](https://docs.celeryq.dev/en/stable/userguide/configuration.html#std-setting-beat_schedule)后台设置中，同样的设置也可以用于手动设置周期性任务：

```Python
app.conf.beat_schedule = {
    'add-every-30-seconds': {
        'task': 'tasks.add',
        'schedule': 30.0,
        'args': (16, 16)
    },
}
app.conf.timezone = 'UTC'
```