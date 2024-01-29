- [服务器](#服务器)
- [高性能web服务器Ngnix](#高性能web服务器ngnix)
  - [进程池+单线程](#进程池单线程)
  - [IO多路复用](#io多路复用)
  - [多阶段处理](#多阶段处理)
- [Nginx配置](#nginx配置)
  - [简单配置](#简单配置)
  - [配置文件构成](#配置文件构成)
    - [location配置](#location配置)
    - [请求转发和重定向](#请求转发和重定向)
    - [nginx静态文件配置](#nginx静态文件配置)
      - [静态文件缓存](#静态文件缓存)
      - [静态文件压缩](#静态文件压缩)
      - [文件下载器](#文件下载器)
    - [配置https SSL](#配置https-ssl)

# 服务器

web服务器（http服务器）： nginx、Apache HTTP Server    处理http请求

应用服务器：tomcat、uwsgi、gunicorn.... 应用服务器都包含了web服务器的功能     运行web应用

WSGI是Python Web应用程序与uWSGI服务器之间的通信协议

Java Servlet API和JSP技术是Java Web应用程序和Tomcat应用服务器之间的通信协议；Tomcat应用服务器提供了Java Servlet容器和JSP容器，可以运行和管理Java Web应用程序。当Tomcat接收到HTTP请求时，它会将请求转发给Java Servlet容器或JSP容器，然后由容器处理请求，生成响应并返回给Tomcat，最终由Tomcat将响应发送给客户端。

**多了一层反向代理的好处？（如上篇所述）**

1. 提高web服务器性能（uWSGI处理静态资源优于nginx，nginx会在收到一个完整的http请求之后转发给wWSGI） 动静分离
2. nginx可以做负载均衡
3. 保护了实际的Web服务器（客户端是和nginx交互而不是直接与uWSGI交互） 反向代理

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202402191910375.png)

# 高性能web服务器Ngnix

## 进程池+单线程

Nginx 则完全不同，“一反惯例”地没有使用多线程，而是使用了“**进程池 + 单线程**”的工作模式。

Nginx 在启动的时候会预先创建好固定数量的 worker 进程，在之后的运行过程中不会再 fork 出新进程，这就是进程池，而且可以自动把进程“绑定”到独立的 CPU 上，这样就完全消除了进程创建和切换的成本，能够充分利用多核 CPU 的计算能力。

## IO多路复用

Web 服务器从根本上来说是“I/O 密集型”而不是“CPU 密集型”，处理能力的关键在于网络收发而不是 CPU 计算（这里暂时不考虑 HTTPS 的加解密），而网络 I/O 会因为各式各样的原因不得不等待，比如数据还没到达、对端没有响应、缓冲区满发不出去等等。

这种情形就有点像是 HTTP 里的“队头阻塞”。对于一般的单线程来说 CPU 就会“停下来”，造成浪费。而多线程的解决思路有点类似“并发连接”，虽然有的线程可能阻塞，但由于多个线程并行，总体上看阻塞的情况就不会太严重了。

Nginx 里使用的 epoll，就好像是 HTTP/2 里的“多路复用”技术，它把多个 HTTP 请求处理打散成碎片，都“复用”到一个单线程里，不按照先来后到的顺序处理，而是只当连接上真正可读、可写的时候才处理，如果可能发生阻塞就立刻切换出去，处理其他的请求。

## 多阶段处理

Nginx 的 HTTP 处理有四大类模块：

1. handler 模块：直接处理 HTTP 请求；
2. filter 模块：不直接处理请求，而是加工过滤响应报文；
3. upstream 模块：实现反向代理功能，转发请求到其他服务器；
4. balance 模块：实现反向代理时的负载均衡算法。

# Nginx配置

## 简单配置

server{ } 其实是包含在 http{ } 内部的。每一个 server{ } 是一个虚拟主机（站点）。

当一个请求叫做`localhost:8080`请求nginx服务器时，该请求就会被匹配进该代码块的 server{ } 中执行

```bash
# 负载均衡：设置domain
upstream domain {
    server localhost:8000;
    server localhost:8001;
}
server {  
        listen       8080;        
        server_name  localhost;

        location / {
            # root   html; # Nginx默认值
            # index  index.html index.htm;
            
            proxy_pass http://domain; # 负载均衡配置，请求会被平均分配到8000和8001端口
            proxy_set_header Host $host:$server_port;
        }
        
        # 静态化配置，所有静态请求都转发给 nginx 处理，存放目录为 my-project
        location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
            root /usr/local/var/www/my-project; # 静态请求所代理到的根目录
        }
}
```

## 配置文件构成

https://zhuanlan.zhihu.com/p/372610935

- 全局块：比如工作进程数，定义日志路径；
- Events块：设置处理轮询事件模型，每个工作进程最大连接数及http层的keep-alive超时时间；
- http块：路由匹配、静态文件服务器、反向代理、负载均衡等。

```bash
# 全局块
 user www-data;
 worker_processes  2;  ## 默认1，一般建议设成CPU核数1-2倍
 error_log  logs/error.log; ## 错误日志路径
 pid  logs/nginx.pid; ## 进程id
 
 # Events块
 events {
   # 使用epoll的I/O 模型处理轮询事件。
   # 可以不设置，nginx会根据操作系统选择合适的模型
   use epoll;
   
   # 工作进程的最大连接数量, 默认1024个
   worker_connections  2048;
   
   # http层面的keep-alive超时时间
   keepalive_timeout 60;
   
   # 客户端请求头部的缓冲区大小
   client_header_buffer_size 2k;
 }
 
 # http块
 http { 
   # httpp全局块
   include mime.types;  # 导入文件扩展名与文件类型映射表
   default_type application/octet-stream;  # 默认文件类型
   
   # 日志格式及access日志路径
   # 在全局块中，我们介绍过errer_log指令，其用于配置Nginx进程运行时的日志存放和级别，此处所指的日志与常规的不同，它是指记录Nginx服务器提供服务过程应答前端请求的日志
   log_format   main '$remote_addr - $remote_user [$time_local]  $status '
     '"$request" $body_bytes_sent "$http_referer" '
     '"$http_user_agent" "$http_x_forwarded_for"';
   access_log   logs/access.log  main;
   # 如果你要关闭access_log,你可以使用下面的命令
   # access_log off;
   
   # 允许sendfile方式传输文件，默认为off。
   sendfile     on;
   tcp_nopush   on; # sendfile开启时才开启。
 
   # http server块  虚拟主机
   # 简单反向代理
   server {
     # listen 127.0.0.1:8000;  #只监听来自127.0.0.1这个IP，请求8000端口的请求
     # listen 127.0.0.1; #只监听来自127.0.0.1这个IP，请求80端口的请求（不指定端口，默认80）
     # listen 8000; #监听来自所有IP，请求8000端口的请求
     listen       80;
     # 虚拟主机
     server_name  domain2.com www.domain2.com;
     access_log   logs/domain2.access.log  main;
    
     # 转发动态请求到web应用服务器
     location / {
       proxy_pass      http://127.0.0.1:8000;
       deny 192.24.40.8;  # 拒绝的ip
       allow 192.24.40.6; # 允许的ip   
     }
     
     # 错误页面
     error_page   500 502 503 504  /50x.html;
         location = /50x.html {
             root   html;
         }
   }
 
   # 负载均衡
   upstream backend_server {
     server 192.168.0.1:8000 weight=5; # weight越高，权重越大
     server 192.168.0.2:8000 weight=1;
     server 192.168.0.3:8000;
     server 192.168.0.4:8001 backup; # 热备
   }
 
   server {
     listen          80;
     server_name     big.server.com;
     access_log      logs/big.server.access.log main;
     
     charset utf-8;
     client_max_body_size 10M; # 限制用户上传文件大小，默认1M
 
     location / {
       # 使用proxy_pass转发请求到通过upstream定义的一组应用服务器
       proxy_pass      http://backend_server;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header Host $http_host;
       proxy_redirect off;
       proxy_set_header X-Real-IP  $remote_addr;
     }
     
   }
 }​

```

### location配置

匹配符 匹配规则 优先级逐级降低

- = 精确匹配 
- ^~ 以某个字符串开头 
- ~ 区分大小写的正则匹配 
- ~* 不区分大小写的正则匹配 
- !~ 区分大小写的不匹配正则 
- !~* 不区分大小写的不匹配正则 
- / 通用匹配，任何请求都会匹配到 

alias和root的区别

- root对路径的处理：root路径 ＋ location路径
- alias对路径的处理：使用alias路径替换location路径

```bash
 # 规则1：通用匹配
 location / {
 }
 
 # 规则2：处理以/static/开头的url
 location ^~ /static {                         
     alias /usr/share/nginx/html/static; # 静态资源路径
 }
 
  # 规则2：处理以/static/开头的url
 location ^~ /static {                         
     root /usr/share/nginx/html; # 静态资源路径
 }
 
 # 拒绝访问所有图片格式文件
 location ~* .*\.(jpg|gif|png|jpeg)$ {
         deny all;
 }
```

### 请求转发和重定向

```bash
# 转发动态请求
 server {  
     listen 80;                                                         
     server_name  localhost;                                               
     client_max_body_size 1024M;
 
     location / {
         proxy_pass http://localhost:8080;   
         proxy_set_header Host $host:$server_port;
     }
 }
 
 # http请求重定向到https请求
 server {
     listen 80;
     server_name Domain.com;
     return 301 https://$server_name$request_uri;
 }

```



### nginx静态文件配置

Nginx可直接作为强大的静态文件服务器使用，支持对静态文件进行缓存还可以直接将Nginx作为文件下载服务器使用。

#### 静态文件缓存

缓存可以加快下次静态文件加载速度。我们很多与网站样式相关的文件比如css和js文件一般不怎么变化，缓存有效器可以通过expires选项设置得长一些。

```bash
# 使用expires选项开启静态文件缓存，10天有效
 location ~ ^/(images|javascript|js|css|flash|media|static)/  {
   root    /var/www/big.server.com/static_files;
   expires 10d;
 }
```

#### 静态文件压缩

Nginx可以对网站的css、js 、xml、html 文件在传输前进行压缩，大幅提高页面加载速度。经过Gzip压缩后页面大小可以变为原来的30%甚至更小。使用时仅需开启Gzip压缩功能即可。你可以在http全局块或server块增加这个配置。

```bash
http {
     
     # 开启gzip压缩功能
     gzip on;
     
     # 设置允许压缩的页面最小字节数; 这里表示如果文件小于10k，压缩没有意义.
     gzip_min_length 10k; 
     
     # 设置压缩比率，最小为1，处理速度快，传输速度慢；
     # 9为最大压缩比，处理速度慢，传输速度快; 推荐6
     gzip_comp_level 6; 
     
     # 设置压缩缓冲区大小，此处设置为16个8K内存作为压缩结果缓冲
     gzip_buffers 16 8k; 
     
     # 设置哪些文件需要压缩,一般文本，css和js建议压缩。图片视需要要锁。
     gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript; 
     
 }
```

#### 文件下载器

```bash
 server {
 
     listen 80 default_server;
     listen [::]:80 default_server;
     server_name  _;
     
     location /download {    
         # 下载文件所在目录
         root /usr/share/nginx/html;
         
         # 开启索引功能
         autoindex on;  
         
         # 关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
         autoindex_exact_size off; 
         
         #显示本机时间而非 GMT 时间
         autoindex_localtime on;   
                 
         # 对于txt和jpg文件，强制以附件形式下载，不要浏览器直接打开
         if ($request_filename ~* ^.*?\.(txt|jpg|png)$) {
             add_header Content-Disposition 'attachment';
         }
     }
 }
```



### 配置https SSL

1. 获取SSL证书：您需要获取SSL证书，可以从第三方提供商购买，也可以使用免费的证书颁发机构（CA）如Let's Encrypt获取。
2. 安装SSL证书：您需要将SSL证书文件和私钥文件复制到Nginx服务器上，并确保只有具有读取权限的用户可以访问它们。
3. 配置Nginx：在Nginx的配置文件中，您需要添加以下部分来启用HTTPS并将SSL证书与Nginx配置关联：

```bash
server {
  listen 443 ssl;
  server_name example.com;

  ssl_certificate /path/to/cert.pem;
  ssl_certificate_key /path/to/key.pem;

  # Other HTTPS-related configurations
}
```