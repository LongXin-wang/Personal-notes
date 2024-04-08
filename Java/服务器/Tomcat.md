- [Tomcat 简介](#tomcat-简介)
  - [web工程发布目录结构](#web工程发布目录结构)
- [Tomcat 架构](#tomcat-架构)
  - [连接器](#连接器)
    - [ProtocolHandler 组件](#protocolhandler-组件)
      - [EndPoint 组件](#endpoint-组件)
      - [Processor 组件](#processor-组件)
    - [Adapter 组件](#adapter-组件)
  - [容器](#容器)
    - [请求分发Servlet过程](#请求分发servlet过程)
- [Tomcat 生命周期](#tomcat-生命周期)
  - [Catalina组件](#catalina组件)
  - [Server 组件](#server-组件)
  - [Service 组件](#service-组件)
- [部署](#部署)
  - [部署文件](#部署文件)
  - [确定请求由谁处理](#确定请求由谁处理)
  - [在一个Tomcat下配置多个服务，用不同的端口号](#在一个tomcat下配置多个服务用不同的端口号)

# Tomcat 简介

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221724804.png)

## web工程发布目录结构

- `webapp`：工程发布文件夹。其实每个 war 包都可以视为 webapp 的压缩包。
- `META-INF`：META-INF 目录用于存放工程自身相关的一些信息，元文件信息，通常由开发工具，环境自动生成。
- `WEB-INF`：Java web 应用的安全目录。所谓安全就是客户端无法访问，只有服务端可以访问的目录。
- `/WEB-INF/classes`：存放程序所需要的所有 Java class 文件。
- `/WEB-INF/lib`：存放程序所需要的所有 jar 文件。
- `/WEB-INF/web.xml`：web 应用的部署配置文件。它是工程中最重要的配置文件，它描述了 servlet 和组成应用的其它组件，以及应用初始化参数、安全管理约束等。

```Java
|-- webapp                         # 站点根目录
    |-- META-INF                   # META-INF 目录
    |   `-- MANIFEST.MF            # 配置清单文件
    |-- WEB-INF                    # WEB-INF 目录
    |   |-- classes                # class文件目录
    |   |   |-- *.class            # 程序需要的 class 文件
    |   |   `-- *.xml              # 程序需要的 xml 文件
    |   |-- lib                    # 库文件夹
    |   |   `-- *.jar              # 程序需要的 jar 包
    |   `-- web.xml                # Web应用程序的部署描述文件
    |-- <userdir>                  # 自定义的目录
    |-- <userfiles>                # 自定义的资源文件

```

# Tomcat 架构

Tomcat 要实现 2 个核心功能：

- **处理 Socket 连接**，负责网络字节流与 Request 和 Response 对象的转化。
- **加载和管理 Servlet**，以及**处理具体的 Request 请求**。

为此，Tomcat 设计了两个核心组件：

- **连接器（Connector）**：负责和外部通信
- **容器（Container）**：负责内部处理

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221733741.png)


一个 Tomcat 实例有一个或多个 Service；一个 Service 有多个 Connector 和 Container。Connector 和 Container 之间通过标准的 ServletRequest 和 ServletResponse 通信。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221734921.png)


## 连接器

连接器的主要功能是：

- 网络通信
- 应用层协议解析
- Tomcat Request/Response 与 ServletRequest/ServletResponse 的转化

Tomcat 设计了 3 个组件来实现这 3 个功能，分别是 **`EndPoint`**、**`Processor`** 和 **`Adapter`**。

组件间通过抽象接口交互。这样做还有一个好处是**封装变化。**这是面向对象设计的精髓，将系统中经常变化的部分和稳定的部分隔离，有助于增加复用性，并降低系统耦合度。网络通信的 I/O 模型是变化的，可能是非阻塞 I/O、异步 I/O 或者 APR。应用层协议也是变化的，可能是 HTTP、HTTPS、AJP。浏览器端发送的请求信息也是变化的。但是整体的处理逻辑是不变的，EndPoint 负责提供字节流给 Processor，Processor 负责提供 Tomcat Request 对象给 Adapter，Adapter 负责提供 ServletRequest 对象给容器。

### ProtocolHandler 组件

**连接器用 ProtocolHandler 接口来封装通信协议和 I/O 模型的差异**。ProtocolHandler 内部又分为 EndPoint 和 Processor 模块，EndPoint 负责底层 Socket 通信，Proccesor 负责应用层协议解析。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221736188.png)

#### EndPoint 组件

EndPoint 是通信端点，即通信监听的接口，是具体的 Socket 接收和发送处理器，是对传输层的抽象，因此 EndPoint 是用来实现 TCP/IP 协议的。

EndPoint 是一个接口，对应的抽象实现类是 AbstractEndpoint，而 AbstractEndpoint 的具体子类，比如在 NioEndpoint 和 Nio2Endpoint 中，有两个重要的子组件：Acceptor 和 SocketProcessor。

其中 Acceptor 用于监听 Socket 连接请求。SocketProcessor 用于处理接收到的 Socket 请求，它实现 Runnable 接口，在 Run 方法里调用协议处理组件 Processor 进行处理。为了提高处理能力，SocketProcessor 被提交到线程池来执行。而这个线程池叫作执行器（Executor)。

#### Processor 组件

如果说 EndPoint 是用来实现 TCP/IP 协议的，那么 Processor 用来实现 HTTP 协议，Processor 接收来自 EndPoint 的 Socket，读取字节流解析成 Tomcat Request 和 Response 对象，并通过 Adapter 将其提交到容器处理，Processor 是对应用层协议的抽象。

Processor 是一个接口，定义了请求的处理等方法。它的抽象实现类 AbstractProcessor 对一些协议共有的属性进行封装，没有对方法进行实现。具体的实现有 AJPProcessor、HTTP11Processor 等，这些具体实现类实现了特定协议的解析方法和请求处理方式。

### Adapter 组件

**连接器通过适配器 Adapter 调用容器**。

由于协议不同，客户端发过来的请求信息也不尽相同，Tomcat 定义了自己的 Request 类来适配这些请求信息。

ProtocolHandler 接口负责解析请求并生成 Tomcat Request 类。但是这个 Request 对象不是标准的 ServletRequest，也就意味着，不能用 Tomcat Request 作为参数来调用容器。Tomcat 的解决方案是引入 CoyoteAdapter，这是适配器模式的经典运用，连接器调用 CoyoteAdapter 的 Sevice 方法，传入的是 Tomcat Request 对象，CoyoteAdapter 负责将 Tomcat Request 转成 ServletRequest，再调用容器的 Service 方法。

## 容器

Tomcat 设计了 4 种容器，分别是 Engine、Host、Context 和 Wrapper。

- **Engine** - Servlet 的顶层容器，包含一 个或多个 Host 子容器；
- **Host** - 虚拟主机，负责 web 应用的部署和 Context 的创建；
- **Context** - Web 应用上下文，包含多个 Wrapper，负责 web 配置的解析、管理所有的 Web 资源；
- **Wrapper** - 最底层的容器，是对 Servlet 的封装，负责 Servlet 实例的创 建、执行和销毁。

### 请求分发Servlet过程

Tomcat 是怎么确定请求是由哪个 Wrapper 容器里的 Servlet 来处理的呢？答案是，Tomcat 是用 Mapper 组件来完成这个任务的。

举例来说，假如有一个网购系统，有面向网站管理人员的后台管理系统，还有面向终端客户的在线购物系统。这两个系统跑在同一个 Tomcat 上，为了隔离它们的访问域名，配置了两个虚拟域名：`manage.shopping.com`和`user.shopping.com`，网站管理人员通过`manage.shopping.com`域名访问 Tomcat 去管理用户和商品，而用户管理和商品管理是两个单独的 Web 应用。终端客户通过`user.shopping.com`域名去搜索商品和下订单，搜索功能和订单管理也是两个独立的 Web 应用。如下所示，演示了 url 应声 Servlet 的处理流程。


![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221741805.png)

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221742277.png)

# Tomcat 生命周期

1. Tomcat 是一个 Java 程序，它的运行从执行 `startup.sh` 脚本开始。`startup.sh` 会启动一个 JVM 来运行 Tomcat 的启动类 `Bootstrap`。
2. `Bootstrap` 会初始化 Tomcat 的类加载器并实例化 `Catalina`。
3. `Catalina` 会通过 Digester 解析 `server.xml`，根据其中的配置信息来创建相应组件，并调用 `Server` 的 `start` 方法。
4. `Server` 负责管理 `Service` 组件，它会调用 `Service` 的 `start` 方法。
5. `Service` 负责管理 `Connector` 和顶层容器 `Engine`，它会调用 `Connector` 和 `Engine` 的 `start` 方法。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221745287.png)

## Catalina组件

Catalina 的职责就是解析 server.xml 配置，并据此实例化 Server。接下来，调用 Server 组件的 init 方法和 start 方法，将 Tomcat 启动起来。

如果我们需要在 JVM 关闭时做一些清理工作，比如将缓存数据刷到磁盘上，或者清理一些临时文件，可以向 JVM 注册一个“关闭钩子”。“关闭钩子”其实就是一个线程，JVM 在停止之前会尝试执行这个线程的 `run` 方法。

```java
public void start() {
    //1. 如果持有的 Server 实例为空，就解析 server.xml 创建出来
    if (getServer() == null) {
        load();
    }

    //2. 如果创建失败，报错退出
    if (getServer() == null) {
        log.fatal(sm.getString("catalina.noServer"));
        return;
    }

    //3. 启动 Server
    try {
        getServer().start();
    } catch (LifecycleException e) {
        return;
    }

    // 创建并注册关闭钩子
    if (useShutdownHook) {
        if (shutdownHook == null) {
            shutdownHook = new CatalinaShutdownHook();
        }
        Runtime.getRuntime().addShutdownHook(shutdownHook);
    }

    // 用 await 方法监听停止请求
    if (await) {
        await();
        stop();
    }
}
```

## Server 组件

Server 组件的具体实现类是 StandardServer，Server 继承了 LifeCycleBase，它的生命周期被统一管理，并且它的子组件是 Service，因此它还需要管理 Service 的生命周期，也就是说在启动时调用 Service 组件的启动方法，在停止时调用它们的停止方法。Server 在内部维护了若干 Service 组件，它是以数组来保存的。


Server 组件还有一个重要的任务是启动一个 Socket 来监听停止端口，这就是为什么你能通过 shutdown 命令来关闭 Tomcat。不知道你留意到没有，上面 Caralina 的启动方法的最后一行代码就是调用了 Server 的 await 方法。

在 await 方法里会创建一个 Socket 监听 8005 端口，并在一个死循环里接收 Socket 上的连接请求，如果有新的连接到来就建立连接，然后从 Socket 中读取数据；如果读到的数据是停止命令“SHUTDOWN”，就退出循环，进入 stop 流程。

```java
@Override
public void addService(Service service) {

    service.setServer(this);

    synchronized (servicesLock) {
        // 创建一个长度 +1 的新数组
        Service results[] = new Service[services.length + 1];

        // 将老的数据复制过去
        System.arraycopy(services, 0, results, 0, services.length);
        results[services.length] = service;
        services = results;

        // 启动 Service 组件
        if (getState().isAvailable()) {
            try {
                service.start();
            } catch (LifecycleException e) {
                // Ignore
            }
        }

        // 触发监听事件
        support.firePropertyChange("service", null, service);
    }

}
```

## Service 组件

Service 组件的具体实现类是 StandardService。

【源码】StandardService 源码定义

```Java
public class StandardService extends LifecycleBase implements Service {
    // 名字
    private String name = null;

    //Server 实例
    private Server server = null;

    // 连接器数组
    protected Connector connectors[] = new Connector[0];
    private final Object connectorsLock = new Object();

    // 对应的 Engine 容器
    private Engine engine = null;

    // 映射器及其监听器
    protected final Mapper mapper = new Mapper();
    protected final MapperListener mapperListener = new MapperListener(this);

  // ...
}

```

StandardService 维护了一个 MapperListener 用于支持 Tomcat 热部署。当 Web 应用的部署发生变化时，Mapper 中的映射信息也要跟着变化，MapperListener 就是一个监听器，它监听容器的变化，并把信息更新到 Mapper 中，这是典型的观察者模式。

作为“管理”角色的组件，最重要的是维护其他组件的生命周期。此外在启动各种组件时，要注意它们的依赖关系，也就是说，要注意启动的顺序。

```Java
protected void startInternal() throws LifecycleException {

    //1. 触发启动监听器
    setState(LifecycleState.STARTING);

    //2. 先启动 Engine，Engine 会启动它子容器
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }

    //3. 再启动 Mapper 监听器
    mapperListener.start();

    //4. 最后启动连接器，连接器会启动它子组件，比如 Endpoint
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            if (connector.getState() != LifecycleState.FAILED) {
                connector.start();
            }
        }
    }
}

```

从启动方法可以看到，Service 先启动了 Engine 组件，再启动 Mapper 监听器，最后才是启动连接器。这很好理解，因为内层组件启动好了才能对外提供服务，才能启动外层的连接器组件。而 Mapper 也依赖容器组件，容器组件启动好了才能监听它们的变化，因此 Mapper 和 MapperListener 在容器组件之后启动。组件停止的顺序跟启动顺序正好相反的，也是基于它们的依赖关系。


# 部署

## 部署文件

- Server元素在最顶层，代表整个Tomcat容器，因此它必须是server.xml中唯一一个最外层的元素。一个Server元素中可以有一个或多个Service元素。Server的主要任务，就是提供一个接口让客户端能够访问到这个Service集合，同时维护它所包含的所有的Service的声明周期，包括如何初始化、如何结束服务、如何找到客户端要访问的Service。
- Service的作用，是在Connector和Engine外面包了一层，把它们组装在一起，对外提供服务。**一个**Service**可以包含多个Connector，但是只能包含一个Engine；**其中Connector的作用是从客户端接收请求，Engine的作用是处理接收进来的请求。Tomcat可以提供多个Service，不同的Service监听不同的端口
- Connector的主要功能，是接收连接请求，创建Request和Response对象用于和请求端交换数据；然后分配线程让Engine来处理这个请求，并把产生的Request和Response对象传给Engine。
- Engine组件从一个或多个Connector中接收请求并处理，并将完成的响应返回给Connector，最终传递给客户端。
- Host是Engine的子容器。Engine组件中可以内嵌1个或多个Host组件，**每个Host**组件代表Engine中的一个虚拟主机。Host组件至少有一个，且其中一个的name必须与Engine组件的defaultHost属性相匹配。Host虚拟主机的作用，是运行多个Web应用（一个Context代表一个Web应用），并负责安装、展开、启动和结束每个Web应用。
- Context**元素代表在特定虚拟主机上运行的一个Web**应用。**在后文中，提到Context、应用或Web应用，它们指代的都是Web应用。每个Web应用基于WAR文件，或WAR文件解压后对应的目录（这里称为应用目录）

```XML
<Server>
    <Service>
       <Connector />
       <Connector />
       <Engine>
            <Host>
                <Context /><!-- 现在常常使用自动部署，不推荐配置Context元素，Context小节有详细说明 -->
            </Host>
       </Engine>
    </Service>
</Server>
```

## 确定请求由谁处理

（1）根据协议和端口号选定Service和Engine

Service中的Connector组件可以接收特定端口的请求，因此，当Tomcat启动时，Service组件就会监听特定的端口。在第一部分的例子中，Catalina这个Service监听了8080端口（基于HTTP协议）和8009端口（基于AJP协议）。当请求进来时，Tomcat便可以根据协议和端口号选定处理请求的Service；Service一旦选定，Engine也就确定。

**通过在Server中配置多个Service，可以实现通过不同的端口号来访问同一台机器上部署的不同应用。**

（2）根据域名或IP地址选定Host

Service确定后，Tomcat在Service中寻找名称与域名/IP地址匹配的Host处理该请求。如果没有找到，则使用Engine中指定的defaultHost来处理该请求。在第一部分的例子中，由于只有一个Host（name属性为localhost），因此该Service/Engine的所有请求都交给该Host处理。

（3）根据URI选定Context/Web应用

这一点在Context一节有详细的说明：Tomcat根据应用的 path属性与URI的匹配程度来选择Web应用处理相应请求，这里不再赘述。

（4）举例

以请求http://localhost:8080/app1/index.html为例，首先通过协议和端口号（http和8080）选定Service；然后通过主机名（localhost）选定Host；然后通过uri（/app1/index.html）选定Web应用。

当客户端发送请求时，请求是通过目标主机的 IP 地址和端口号来定位服务的。当 Tomcat 作为 Web 服务器运行时，默认情况下监听 HTTP 请求的端口是 8080。这意味着，如果 Tomcat 正在运行，并监听在端口 8080 上，它将能够接收来自客户端的 HTTP 请求。

一旦请求到达服务器，Tomcat 将根据请求中的 Host 头信息来确定应该将请求交给哪个虚拟主机（不同的域名对应不同的虚拟主机）。然后，Tomcat 将请求路由到相应的虚拟主机Host，也就是配置中对应的 Host 元素，从而处理该请求。

为两个不同的域名（www.xxxx.com 和 www.yyyy.com dns解析到同一个IP即可）添加了不同的 Host 元素，并分别指定了相应的 appBase 和 name 属性。这样配置后，Tomcat 将会为每个域名服务请求

```xml
<Service name="Catalina">
    <!-- 其他配置 -->
    <Engine defaultHost="localhost" name="Catalina">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
        <Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
        </Host>
        <Host appBase="webapps1" autoDeploy="true" name="www.xxxx.com" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
        </Host>
        <Host appBase="webapps2" autoDeploy="true" name="www.yyyy.com" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
        </Host>
    </Engine>
</Service>
```

## 在一个Tomcat下配置多个服务，用不同的端口号

```Bash
http://localhost:8013/项目名称1
http://localhost:8014/项目名称2
http://localhost:8015/项目名称3
```

```xml
<Service name="Catalina">
    <Connector connectionTimeout="20000" port="8013" protocol="HTTP/1.1" redirectPort="8443"/>
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
    <Engine defaultHost="localhost" name="Catalina">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
        <Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
        </Host>
    </Engine>
</Service>

<Service name="Catalina1">
    <Connector connectionTimeout="20000" port="8014" protocol="HTTP/1.1" redirectPort="8443"/>
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
    <Engine defaultHost="localhost" name="Catalina1">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
        <Host appBase="webapps1" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
    </Host>
    </Engine>
</Service>

<Service name="Catalina2">
    <Connector connectionTimeout="20000" port="8015" protocol="HTTP/1.1" redirectPort="8443"/>
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
    <Engine defaultHost="localhost" name="Catalina2">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
        <Host appBase="webapps2" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
        </Host>
    </Engine>
</Service>
```
```XML
配置 Nginx 和 Tomcat 联合使用时，需要对 Nginx 和 Tomcat 的配置文件进行设置。

Nginx 配置文件示例（/etc/nginx/nginx.conf）：
http {
    server {
        listen 80;
        server_name your_domain.com;

        location / {
            proxy_pass http://localhost:8080;  # 将请求代理到 Tomcat
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
Tomcat 配置文件示例（server.xml）：
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```