- [Servlet简介](#servlet简介)
  - [Servlet任务](#servlet任务)
  - [Servlet生命周期](#servlet生命周期)
  - [Servlet接口](#servlet接口)
    - [init()方法](#init方法)
    - [service() 方法](#service-方法)
    - [doGet() 方法](#doget-方法)
    - [doPost() 方法](#dopost-方法)
    - [destroy() 方法](#destroy-方法)

# Servlet简介

## Servlet任务

Servlet 执行以下主要任务：

- 读取客户端（浏览器）发送的显式的数据。这包括网页上的 HTML 表单，或者也可以是来自 applet 或自定义的 HTTP 客户端程序的表单。
- 读取客户端（浏览器）发送的隐式的 HTTP 请求数据。这包括 cookies、媒体类型和浏览器能理解的压缩格式等等。
- 处理数据并生成结果。这个过程可能需要访问数据库，执行 RMI 或 CORBA 调用，调用 Web 服务，或者直接计算得出对应的响应。
- 发送显式的数据（即文档）到客户端（浏览器）。该文档的格式可以是多种多样的，包括文本文件（HTML 或 XML）、二进制文件（GIF 图像）、Excel 等。
- 发送隐式的 HTTP 响应到客户端（浏览器）。这包括告诉浏览器或其他客户端被返回的文档类型（例如 HTML），设置 cookies 和缓存参数，以及其他类似的任务。

## Servlet生命周期

Servlet 生命周期如下：

1. **加载** - 第一个到达服务器的 HTTP 请求被委派到 Servlet 容器。容器通过类加载器使用 Servlet 类对应的文件加载 servlet；
2. **初始化** - Servlet 通过调用 **init ()** 方法进行初始化。
3. **服务** - Servlet 调用 **service()** 方法来处理客户端的请求。
4. **销毁** - Servlet 通过调用 **destroy()** 方法终止（结束）。
5. **卸载** - Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

![](https://secure2.wostatic.cn/static/3SAg7xf6wRg48MHXKpU7o/image.png?auth_key=1716371575-tET27fj7DEvEz9Rhk5Dug5-0-60e5bf07142f7f4a17b493d1cdf6ed11)

## Servlet接口

```Java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}

```

### init()方法

init 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用。因此，它是用于一次性初始化，就像 Applet 的 init 方法一样。

Servlet 创建于用户第一次调用对应于该 Servlet 的 URL 时，但是您也可以指定 Servlet 在服务器第一次启动时被加载。

当用户调用一个 Servlet 时，就会创建一个 Servlet 实例，每一个用户请求都会产生一个新的线程，适当的时候移交给 doGet 或 doPost 方法。init() 方法简单地创建或加载一些数据，这些数据将被用于 Servlet 的整个生命周期。

```Java
public void init() throws ServletException {
  // 初始化代码...
}
```

### service() 方法

**`service()`**** 方法是执行实际任务的核心方法**。Servlet 容器（即 Web 服务器）调用 `service()` 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。

`service()` 方法有两个参数：`ServletRequest` 和 `ServletResponse`。`ServletRequest` 用来封装请求信息，`ServletResponse` 用来封装响应信息，因此**本质上这两个类是对通信协议的封装。**

每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。`service()` 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 `doGet`、`doPost`、`doPut`，`doDelete` 等方法。

下面是该方法的特征：

```Java
public void service(ServletRequest request,
                    ServletResponse response)
      throws ServletException, IOException{
}

```

service() 方法由容器调用，service 方法在适当的时候调用 doGet、doPost、doPut、doDelete 等方法。所以，您不用对 service() 方法做任何动作，您只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可。

doGet() 和 doPost() 方法是每次服务请求中最常用的方法。下面是这两种方法的特征。

### doGet() 方法

GET 请求来自于一个 URL 的正常请求，或者来自于一个未指定 METHOD 的 HTML 表单，它由 doGet() 方法处理。

```Java
public void doGet(HttpServletRequest request,
                  HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}

```

### doPost() 方法

POST 请求来自于一个特别指定了 METHOD 为 POST 的 HTML 表单，它由 doPost() 方法处理。

```Java
public void doPost(HttpServletRequest request,
                   HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}

```

### destroy() 方法

destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法可以让您的 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。

在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。destroy 方法定义如下所示：

```Java
  public void destroy() {
    // 终止化代码...
  }

```