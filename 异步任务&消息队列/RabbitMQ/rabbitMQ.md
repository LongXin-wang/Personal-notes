- [RabbitMQ 模型](#rabbitmq-模型)
- [常用模式](#常用模式)
  - [简单模式](#简单模式)
    - [简单模式](#简单模式-1)
    - [Work 模式](#work-模式)
  - [exchange 模式](#exchange-模式)
    - [fanout 模式](#fanout-模式)
    - [direct 模式](#direct-模式)
    - [topic 模式](#topic-模式)

# RabbitMQ 模型 

生产者

产生数据发送消息的程序是生产者。

交换机

交换机是 RabbitMQ 非常重要的一个部件，一方面它接收来自生产者的消息，另一方面它将消息 推送到队列中。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推 送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定 。

队列

队列是RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ 和应用程序，但它们只能存 储在队列中。队列仅受主机的内存和磁盘限制的约束，本质上是一个大的消息缓冲区。许多生产者可 以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式 。

消费者

消费与接收具有相似的含义。消费者大多时候是一个等待接收消息的程序。请注意生产者，消费 者和消息中间件很多时候并不在同一机器上。同一个应用程序既可以是生产者又是可以是消费者。


![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402261611181.png)

# 常用模式

## 简单模式

缺点就是生产者的所有消息全堆积到同一个队列中，没有做消息分类

### 简单模式

一对一，一个生产者，一个消费者。一边塞，一边取。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402271408565.png)

```python
'''生产者'''
import pika

# 1、连接rabbitmq服务器
with pika.BlockingConnection(pika.ConnectionParameters('x.x.x.x')) as connection:
    channel = connection.channel()

    # 2、创建一个名为hello的队列
    # channel.queue_declare(queue='hello', durable=True) # 持久化队列
    channel.queue_declare(queue='hello')

    # 3、如果exchange为空，即简单模式:向名为hello队列中插入字符串Hello World!
    channel.basic_publish(exchange='',
                          routing_key='hello',
                          body="Hello World",
                          # 持久化队列配置
                          # properties=pika.BasicProperties(
                          #     delivery_mode=2,
                          # )
                          )

    print("发送 ‘{}’ 成功".format("Hello World!"))


'''消费者'''
import pika

# 1、连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(host='x.x.x.x'))
channel = connection.channel()

# 2、两边谁先启动谁创建队列
# channel.queue_declare(queue='hello',durable=True) # 持久化队列
channel.queue_declare(queue='hello')

# 一旦有消息就执行该回调函数(比如减库存操作就在这里面)
def callback(ch,method,properties,body):
    print("消费者端收到来自消息队列中的{}成功".format(body))
    # 数据处理完成，MQ收到这个应答就会删除消息
    ch.basic_ack(delivery_tag=method.delivery_tag)

# 消费者这边监听的队列是hello,一旦有值出现,则触发回调函数：callback
channel.basic_consume(queue='hello',
                      auto_ack=False, # 默认就是False,可以直接不写
                      on_message_callback=callback,
                      )


print('当前MQ简单模式正在等待生产者往消息队列塞消息.......要退出请按 CTRL+C.......')
channel.start_consuming()
```

### Work 模式

一个生产者P，对应多个消费者，多个消费者同时监听这一个队列

谁处理完谁来拿消息处理，就不分你我，大家共同目的就是早处理完消息

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402271417406.png)

```python
'''生产者'''
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='work_queue', durable=True)

for i in range(10):
    message = f"Message {i}"
    channel.basic_publish(exchange='',
                          routing_key='work_queue',
                          body=message,
                          properties=pika.BasicProperties(
                              delivery_mode=2,  # 使消息持久化
                          ))
    print("Sent %r" % message)

connection.close()

'''消费者'''
# 在不同的终端中运行 work_consumer.py 
import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='work_queue', durable=True)

def callback(ch, method, properties, body):
    print("Received %r" % body)
    time.sleep(body.count(b'.'))
    print("Done")
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='work_queue', on_message_callback=callback)

print('Waiting for messages. To exit press CTRL+C')
channel.start_consuming()
```

## exchange 模式

fanout，direct和topic统称为exchange模式或交换机模式，该模式下的每个消费者都有自己创建的队列，采用三种方式中的任意一种来绑定交换机，再由交换机分配消息给这些队列

### fanout 模式

将交换器获取的消息，直接全部转发给所有绑定的队列

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402271429317.png)

```python
'''生产者'''
import pika

# 1、连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(host='x.x.x.x'))
channel = connection.channel()

# 2、创建一个名为logs的交换机(用于分发日志),模式是发布订阅模式
channel.exchange_declare(exchange='logs',
                         exchange_type='fanout')

message = "I am producer this is my message"
# 生产者向交换机logs塞消息message
channel.basic_publish(exchange='logs',
                      routing_key='',
                      body=message,
                      )

print("发送 {} 成功".format(message))
# 关闭连接
connection.close()

'''消费者'''
**多个消费者运行都获取到了**
import pika

# 1、连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(host='x.x.x.x'))
channel = connection.channel()

# 2、创建一个名为logs的交换机(用于分发日志),模式是发布订阅模式
channel.exchange_declare(exchange='logs',
                         exchange_type='fanout')

# 3、创建一个随机队列（exclusive=True）
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue  # 获取随机队列名称
print('随机队列名: {}'.format(queue_name))

# 4、为名为queue_name的随机队列绑定名为logs的交换机
channel.queue_bind(exchange='logs',
                   queue=queue_name,
                   )

print('当前MQ发布订阅模式正在等待交换机往消息队列塞消息.......要退出请按 CTRL+C.......')

# 创建回调函数(收到监听队列的消息后执行该回调)
def callback(ch,method,properties,body):
    print("接收到 {} 成功.......".format(body))

    # 给mq发送应答信号，表明数据已经处理完成，可以删除
    ch.basic_ack(delivery_tag=method.delivery_tag)

# 监听随机队列，一旦有消息出现，则触发回调函数：callback
channel.basic_consume(queue=queue_name,
                      auto_ack=False, # 默认就是False,可以直接不写
                      on_message_callback=callback,
                      )

# 哪个消费者先处理完谁就去消息队列取
channel.basic_qos(prefetch_count=1)
channel.start_consuming()
```

### direct 模式

消费者端创建随机队列后，给队列绑定交换机时，传一个routing_key给交换机，然后生产者端发送消息给交换机时，也给交换机一个routing_key，只有双方给交换机的关键字暗号对的上，交换机才会转发消息给队列

交换器转发消息可以进行路由过滤，路由只支持精确匹配

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402271440950.png)

```python
'''生产者'''
import pika

# 1、连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(host='x.x.x.x'))
channel = connection.channel()

# 2、声明一个名为direct_logs的交换机,类型为关键字模式
channel.exchange_declare(exchange='direct_logs',
                         exchange_type='direct',
                         )

message = "I am producer this is my message Hello!"

# 3、向交换机发送消息,并告诉交换机只发给绑定了lan或yue关键字的消费者队列
channel.basic_publish(exchange='direct_logs',
                      routing_key='lan', # 可以用for循环，不用像这样一个一个加
                      body=message,
                      )
channel.basic_publish(exchange='direct_logs',
                      routing_key='yue',
                      body=message,
                      )


print("Sent {} 成功".format(message))
# 关闭连接
connection.close()


'''消费者'''
**一个消费者的随机队列绑定了lan和yue，另一个消费者绑定yue 类此设计**
import pika

# 1、连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(host='x.x.x.x'))
channel = connection.channel()

# 跟队列同理,因为不确定生产者和消费者谁先跑起来,消费者端也要创建交换机
# 2、创建一个名为direct_logs的交换机，类型为关键字模式。
channel.exchange_declare(exchange='direct_logs',
                         exchange_type='direct',
                         )

# 3、创建一个随机队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue


# 4、为随机队列绑定名为direct_logs的交换机,关键字为lan和yue （这一个随机队列绑定了两个）
channel.queue_bind(exchange='direct_logs',
                   queue=queue_name,
                   routing_key='yue') # 也推荐用for循环

channel.queue_bind(exchange='direct_logs',
                   queue=queue_name,
                   routing_key='chuan')


print('当前MQ关键字模式正在等待交换机往消息队列塞消息.......要退出请按 CTRL+C.......')

# 构建回调函数
def callback(ch,method,properties,body):
    print("Received {} 成功.......".format(body))

    # 给mq发送应答信号，表明数据已经处理完成，可以删除
    ch.basic_ack(delivery_tag=method.delivery_tag)

# 监听随机队列，一旦有值出现，则触发回调函数：callback
channel.basic_consume(queue=queue_name,
                      auto_ack=False, # 默认就是False,可以直接不写
                      on_message_callback=callback,
                      )

# 消费者不止这一个时，谁先处理完谁就去消息队列取
channel.basic_qos(prefetch_count=1)
channel.start_consuming()
```

### topic 模式

路由模式，支持路由匹配，且支持路由的模糊匹配

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402271443694.png)

#匹配0个或多个单词，*只匹配一个单词，a和abc都叫做一个单词

```python
'''生产者'''
import pika

# 1、连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(host='x.x.x.x'))
channel = connection.channel()

# 2、创建一个名为topic_logs的交换机
channel.exchange_declare(exchange='topic_logs',
                         exchange_type='topic',
                         )

message = "welcome to rabbitmq  lyc"

# 3、向交换机发送数据,让交换机只给能匹配lan.adasd.*的队列发消息
channel.basic_publish(exchange='topic_logs',
                      routing_key='lan.adasd.*',
                      body=message,
                      )

print("Sent {} 成功".format(message))
# 关闭连接
connection.close()

'''消费者'''
import pika

# 1、连接rabbitmq服务器
connection = pika.BlockingConnection(pika.ConnectionParameters(host='x.x.x.x'))
channel = connection.channel()

# 2、创建一个名为topic_logs的交换机
channel.exchange_declare(exchange='topic_logs',
                         exchange_type='topic',
                         )

# 3、创建一个随机队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue


# 4、为名为queue_name的随机队列绑定名为logs的交换机
channel.queue_bind(exchange='topic_logs',
                   queue=queue_name,
                   routing_key='lan.*.#') # 以routing_key作为关键字


print('当前MQ模糊匹配模式正在等待交换机往消息队列塞消息.......要退出请按 CTRL+C.......')

# 构建回调函数
def callback(ch,method,properties,body):
    body = body.decode('utf8')
    print("接收 {} 成功.......".format(body))

    # 给mq发送应答信号，表明数据已经处理完成，可以删除
    ch.basic_ack(delivery_tag=method.delivery_tag)

# 监听随机队列，一旦有值出现，则触发回调函数：callback
channel.basic_consume(queue=queue_name,
                      auto_ack=False, # 默认就是False,可以直接不写
                      on_message_callback=callback,
                      )

# 消费者不止这一个时，谁先处理完谁就去消息队列取
channel.basic_qos(prefetch_count=1)
channel.start_consuming()

```