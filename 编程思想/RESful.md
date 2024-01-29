- [RESful规范](#resful规范)
  - [接口示例：](#接口示例)
    - [传统URL请求格式：](#传统url请求格式)
    - [RESTful请求格式：](#restful请求格式)
- [状态码](#状态码)
- [错误处理](#错误处理)


**RESTful:用URL定位资源**、**用HTTP动词（GET、POST、PUT、DELETE)描述操作**

RESTful API就是REST风格的API，即rest是一种架构风格，跟编程语言无关，跟平台无关，采用HTTP做传输协议

# RESful规范

在rest中会通过向服务器提交的请求类型来表示增删改查这些操作
  - GET（SELECT）：从服务器取出资源。
  - POST（CREATE）：在服务器新建一个资源。
  - PUT（UPDATE）：在服务器更新资源。
  - DELETE（DELETE）：从服务器删除资源。
  - PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
  - DELETE /zoos/ID：删除某个动物园
  - GET /zoos/ID/animals：列出某个指定动物园的所有动物
  - DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

①、应该将API的版本号放入URL。GET:[http://www.xxx.com/v1/friend/123。或者将版本号放在HTTP头信息中。我个人觉得要不要版本号取决于自己开发团队的习惯和业务的需要，不是强制的。](http://www.xxx.com/v1/friend/123。或者将版本号放在HTTP头信息中。我个人觉得要不要版本号取决于自己开发团队的习惯和业务的需要，不是强制的。)

②、URL中只能有名词而不能有动词，操作的表达是使用HTTP的动词GET,POST,PUT,DELETEL。URL只标识资源的地址，既然是资源那就是名词了。

③、如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。?limit=10：指定返回记录的数量、?page=2&per_page=100：指定第几页，以及每页的记录数。

例子

GET:[http://www.xxx.com/source/id](http://www.xxx.com/source/id) 获取指定ID的某一类资源。

GET:[http://www.xxx.com/friends/123表示获取ID为123的用户的好友列表。如果不加id就表示获取所有用户的好友列表。](http://www.xxx.com/friends/123表示获取ID为123的用户的好友列表。如果不加id就表示获取所有用户的好友列表。)

POST:[http://www.xxx.com/friends/123表示为指定ID为123的用户新增好友。其他的操作类似就不举例了。](http://www.xxx.com/friends/123表示为指定ID为123的用户新增好友。其他的操作类似就不举例了。)

## 接口示例：

### 传统URL请求格式：

> [http://127.0.0.1/user/query/1](http://127.0.0.1/user/query/1) GET 根据用户id查询用户数据

[http://127.0.0.1/user/save](http://127.0.0.1/user/save) POST 新增用户

[http://127.0.0.1/user/update](http://127.0.0.1/user/update) POST 修改用户信息

[http://127.0.0.1/user/delete](http://127.0.0.1/user/delete) GET/POST 删除用户信息

### RESTful请求格式：

> [http://127.0.0.1/user/1](http://127.0.0.1/user/1) GET 根据用户id查询用户数据

[http://127.0.0.1/user](http://127.0.0.1/user) POST 新增用户

[http://127.0.0.1/user](http://127.0.0.1/user) PUT 修改用户信息

[http://127.0.0.1/user](http://127.0.0.1/user) DELETE 删除用户信息

# 状态码

幂等：

在程序中如果相同条件下**多次请求**对资源的影响**表现一致**则称请求为幂等请求，对应的接口为幂等接口。

```Python
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
**400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
**401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是**幂等**的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```

# 错误处理

如果状态码是4xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。

```Python
{
    error: "Invalid API key"
}
```

