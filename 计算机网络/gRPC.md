- [简述](#简述)
- [Protocol Buffers (协议缓冲区)](#protocol-buffers-协议缓冲区)
  - [示例](#示例)
- [gRPC的四种服务提供方法](#grpc的四种服务提供方法)
  - [Unary RPC](#unary-rpc)
  - [Server streaming RPC](#server-streaming-rpc)
  - [Client streaming RPC](#client-streaming-rpc)
  - [Bidirectional streaming RPC](#bidirectional-streaming-rpc)
- [RPC 原理](#rpc-原理)
  - [远程调用实现](#远程调用实现)
    - [传递值参数](#传递值参数)
    - [传递引用参数](#传递引用参数)

# 简述

1、客户端（gRPC Stub）调用 A 方法，发起 RPC 调用。
2、对请求信息使用 Protobuf 进行对象序列化压缩（IDL）。
3、服务端（gRPC Server Stub）接收到请求后，解码请求体，进行业务逻辑处理并返回。
4、对响应结果使用 Protobuf 进行对象序列化压缩（IDL）。
5、客户端接受到服务端响应，解码请求体。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402201045647.png)

# Protocol Buffers (协议缓冲区)

Protocol Buffers是一个带有.proto扩展名的普通文本文件。

```python
syntax = "proto3";


// service 定义了名为Detect的gPRC服务，有名为detect的方法，接受DetectRequest，返回DetectResponse
service Detect {
  rpc detect (DetectRequest) returns (DetectResponse) {}
}

// request
message DetectRequest {
  string content = 1;
}

// response
message DetectResponse {
  string data = 1;
}

## protos文件说明
+ protos文件夹中为grpc服务相关文件
  + detect.proto为grpcde proto文件，该文件内定义了客户端及服务端的接口格式
  + protobuf文件夹内的为proto文件执行编译生成python的proto序列化协议源代码
    # 编译生成的命令为：
    > python -m grpc_tools.protoc -I ./protos/antispam --python_out=./protos/antispam/protobuf/ --grpc_python_out=./protos/antispam/protobuf/ ./protos/antispam/*.proto
```

## 示例

```python
"""客户端Stub"""
def grpc_request(cls, url, need_log, params):
    time_start = time.time()
    try:
        service_config_json = json.dumps({
            "methodConfig": [{
                "name": [{}],
                "retryPolicy": {
                    "maxAttempts": 5,
                    "initialBackoff": "0.1s",
                    "maxBackoff": "1s",
                    "backoffMultiplier": 2,
                    "retryableStatusCodes": ["UNAVAILABLE"],
                },
            }]
        })
        with grpc.insecure_channel(
            url,
            options=[
                ("grpc.enable_retries", 1),
                ("grpc.service_config", service_config_json)
            ]
        ) as channel:
            # 在proto协议生成的文件中
            stub = detect_pb2_grpc.DetectStub(channel)
            response = stub.detect(detect_pb2.DetectRequest(
                content=json.dumps(params)
            ))
            rsp = json.loads(response.data)
    except:
        pass


"""服务端Stub"""
class Detect(detect_pb2_grpc.DetectServicer):
    """
    detect rpc服务
    """

    async def detect(
        self,
        detect_request: detect_pb2.DetectRequest,
        context: grpc.aio.ServicerContext,
    ) -> detect_pb2.DetectResponse:
        """
        反垃圾检测
        Args:
            detect_request:
            context:
        """
        req = DetectRequest(**json.loads(detect_request.content))
        data = None
        if req.request_type == RequestType.TEXT:
            # 单一文本检测
            data = await text_detect(req, client=client)
        elif req.request_type == RequestType.TEXT_BATCH:
            # 批量文本检测
            data = await text_batch_detect(req, client=client)
        elif req.request_type == RequestType.IMAGE:
            # 图片检测
            data = await image_detect(req, client=client)
        elif req.request_type == RequestType.CRAWLER:
            data = await crawler_detect(req, client=client)
        return detect_pb2.DetectResponse(
            data=json.dumps(Response(data=data).dict())
        )


async def start_server():
    # start rpc service
    server = aio.server(
        futures.ThreadPoolExecutor(),
        options=[
            ('grpc.so_reuseport', 0),
            ('grpc.max_send_message_length', 100 * 1024 * 1024),
            ('grpc.max_receive_message_length', 100 * 1024 * 1024),
            ('grpc.enable_retries', 1),
        ]
    )
    # 在proto协议生成的文件中
    detect_pb2_grpc.add_DetectServicer_to_server(Detect(), server)
    server.add_insecure_port('[::]:8011')
    await server.start()

    async def server_graceful_shutdown():
        logging.info("Starting graceful shutdown...")
        await server.stop(5)

    _cleanup_coroutines.append(server_graceful_shutdown())
    await server.wait_for_termination()
    try:
        await server.wait_for_termination()
    except KeyboardInterrupt:
        await server.stop(None)


async def create_client():
    return aiohttp.ClientSession()

client = None

if __name__ == '__main__':
    asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
    loop = asyncio.get_event_loop()
    try:
        client = loop.run_until_complete(create_client())
        loop.run_until_complete(start_server())
    finally:
        loop.run_until_complete(*_cleanup_coroutines)
        loop.close()
```

#  gRPC的四种服务提供方法

## Unary RPC

一元 RPC，其中客户端向服务器发送单个请求并获得单个响应，就像正常的函数调用一样。
```python
rpc SayHello(HelloRequest) returns (HelloResponse);
```

## Server streaming RPC

服务器流式 RPC，其中客户端向服务器发送请求并获取流以读回一系列消息。客户端从返回的流中读取，直到没有更多消息为止。gRPC 保证单个 RPC 调用中的消息顺序。

```python
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
```

## Client streaming RPC

客户端流式 RPC，其中客户端写入一系列消息并将它们发送到服务器，再次使用提供的流。一旦客户端完成了消息的写入，它就会等待服务器读取它们并返回它的响应。gRPC 再次保证了单个 RPC 调用中的消息顺序。

```python
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);

```

## Bidirectional streaming RPC

双向流式 RPC，双方使用读写流发送一系列消息。这两个流独立运行，因此客户端和服务器可以按照他们喜欢的任何顺序读取和写入：例如，服务器可以在写入响应之前等待接收所有客户端消息，或者它可以交替读取消息然后写入消息，或其他一些读取和写入的组合。保留每个流中消息的顺序。

```python
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse)

```

# RPC 原理

## 远程调用实现

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202404291422250.png)

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202404291415528.png)

### 传递值参数

传递值参数比较简单

### 传递引用参数

RPC中的引用参数，一种常见的做法是将参数的副本发送到远程系统，并在那里使用这些副本进行操作，以避免直接传递引用和指针。这可以通过以下方式实现：

序列化和反序列化：将参数对象序列化为一种可以在网络上传输的格式，然后在远程系统中进行反序列化，以还原成对象的副本。这确保了在远程系统中使用的是独立的副本，而不是原始对象的引用。

无指针表示：对于包含指针的复杂数据结构（比如树和链表），需要将其转换为一种无指针的表示形式，以便在远程系统中进行重建。这可以通过将数据结构展平为可以轻松传输和重建的形式来实现。

远程系统操作：在远程系统中，使用接收到的参数副本进行操作，并在必要时将更新后的对象发送回到调用方，以保持调用方和远程系统中对象的同步。


![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202404291423443.png)