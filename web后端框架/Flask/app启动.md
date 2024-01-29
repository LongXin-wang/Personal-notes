- [app创建](#app创建)
  - [创建app对象](#创建app对象)
  - [加载配置文件](#加载配置文件)
    - [设置配置](#设置配置)
  - [加载请求钩子](#加载请求钩子)
  - [加载错误处理方法](#加载错误处理方法)
  - [加载蓝图](#加载蓝图)

# app创建

```python
from flask import Flask
import os

# 创建app对象
app = Flask(__name__,template_folder='static/html')
# 加载配置文件
app.config['SECRET_KEY'] = '123'

# 加载请求钩子
@app.before_request
def rest_test():
    print('this is a test')
    pass

@定义错误处理逻辑
@app.errorhandler(400)
def handle_errer(errer):
    print(errer)
    return errer

# 加载蓝图
app.register_blueprint(rest, url_prefix='')

# 定义api处理视图
@app.route('/test1')
def test():
    return 'OK'


if __name__ == '__main__':
    app.run(host='127.0.0.1', port=80, debug=True)

```

## 创建app对象

```python
'''Flask'''
class Flask(Scaffold):
    request_class = Request # 指定请求对象
    response_class = Response # 指定响应对象
    jinja_environment = Environment # 指定前端模板语言环境
    app_ctx_globals_class = _AppCtxGlobals # 设置app全局对象，本质是一个字典
    request_globals_class = property(_get_request_globals_class，_set_request_globals_class) # 设置请求上下文全局对象，其等于app_ctx_globals_class
                                       
    config_class = Config # 设置参数配置对象
    debug = ConfigAttribute('DEBUG') # 配置文件中如果DEBUG=True就启用debug模式
    testing = ConfigAttribute('TESTING') # 配置文件中如果TESTING=True就启用TESTING模式

    # 通过LOGGER_NAME指定日志对象的名称
    logger_name = ConfigAttribute('LOGGER_NAME')

    # 常用默认的配置集合
    default_config = ImmutableDict({
        'DEBUG': get_debug_flag(default=False), # 默认不开启DEBUG模式
        'TESTING': False, # 默认不开启TESTING模式
        'SECRET_KEY': None, # 默认没有密匙
        'PERMANENT_SESSION_LIFETIME': timedelta(days=31), # 默认session失效时间一个月
        'USE_X_SENDFILE': False, # 不启用X_SENDFILE功能
        'LOGGER_NAME': None, # 不指定logger名字，使用的是__name__
        'LOGGER_HANDLER_POLICY': 'always',
        'SERVER_NAME': None, # 设置服务器的名字，默认None
        'SESSION_COOKIE_NAME': 'session', # 设置cookie中的session名字
        'MAX_CONTENT_LENGTH': None, # 限制提交请求的最大字节数
        'SEND_FILE_MAX_AGE_DEFAULT': timedelta(hours=12), # 设置文件最大缓存时间
        'PREFERRED_URL_SCHEME': 'http', # 默认的通讯协议
        'JSONIFY_MIMETYPE': 'application/json', # 当json数据交互时，设置响应头的mimetype参数
    })

    url_rule_class = Rule # 指定路由对象管理路由
    def __init__(self,
                    import_name, # 指定app的名字
                    static_path=None,  # 会赋值给static_url_path参数，所以一般设置static_url_path而不是这个
                    static_url_path=None, # 设置静态文件路由的前缀，默认为“/static”，即static_folder的路径
                    static_folder='static', # 指定静态文件是哪个目录
                    template_folder='templates',# 模板文件的存放目录，默认值为"templates"
                    instance_path=None, # 设置配置文件的路径，在instance_relative_config=True情况下生效
                    instance_relative_config=False, # 设置为True表示使用instance_path
                    root_path=None): # app所在的根路径，默认指的是创建app这个代码的文件的目录。

        # 所有定义的视图函数存放字典，以蓝图.视图函数的标识符为键，视图函数对象为字典； endpoint：view function的字典
        self.view_functions = {}
        # 所有的Rule对象保存的字典 储存 <Rule '/admin/' (OPTIONS, HEAD, GET) -> adminbp.index>
        self.url_map = self.url_map_class()

       # 所有的http请求处理前的请求钩子方法存放字典
        self.before_request_funcs = {}
        # 第一次请求处理前的请求钩子方法存放字典
        self.before_first_request_funcs = []
        # 所有的http请求处理后的请求钩子方法存放字典
        self.after_request_funcs = {}

        # 所有自定义的错误处理方法的存放字典，以蓝图名字为键
        self.error_handler_spec = {None: self._error_handlers}

        # 所有的与url错误处理方法存放列表
        self.url_build_error_handlers = []

        # 在处理即使存在异常的情况下的请求处理后的请求钩子方法存放字典
        self.teardown_request_funcs = {}

        # 应用上下文被弹出之前的处理函数存放点
        self.teardown_appcontext_funcs = []
        
        #配置文件路径
        self.instance_path = instance_path
        
        #class:`Config`. 配置信息       
        self.config = self.make_config(instance_relative_config)

        self.blueprints = {} # 所有的蓝图的保存字典

        # request.
        self._got_first_request = False
        self._before_request_lock = Lock()

        # 添加访问静态文件的Rule对象
        if self.has_static_folder:
            assert (
                bool(static_host) == host_matching
            ), "Invalid static_host/host_matching combination"
            self.add_url_rule(
                self.static_url_path + "/<path:filename>",
                endpoint="static",  #也设置了
                host=static_host,
                view_func=self.send_static_file,
            )

```



## 加载配置文件

app的config对象其实是Config实例，是一个dict类型的子类，也就是说app的配置是通过一个字典来保存的,新加载的配置如果键和原来默认的配置的键相同将更新，否则添加

```python
self.config = self.make_config(instance_relative_config)

#Flask类中init被调用
def make_config(self, instance_relative=False):
    root_path = self.root_path
    #instance_relative就是instance_relative_config 为false时是直接使用根路径，为True使用传入的instance_path
    if instance_relative:
        root_path = self.instance_path
    defaults = dict(self.default_config)
    #获取默认的是production
    defaults["ENV"] = get_env()
    #DEBUG=True...开发模式可调式，修改之后不用重启服务器，只需要reload；
    defaults["DEBUG"] = get_debug_flag() 
    # appd的类实例config_class
    return self.config_class(root_path, defaults)
```

### 设置配置

**项目中采用的是直接操作app.config/current_app.config(设置类属性)，还有就是run的时候传入debug=config.DEBUG**

- 通过加载文件设置参数

```Python
app.config.from_pyfile("./config.cfg") # 指定参数的路径，内容按行书写,配置文件放置在与app的同目录下

def from_pyfile(self, filename, silent=False):
    filename = os.path.join(self.root_path, filename)
    pass
```
- 通过类设置参数

```Python
class Config(object):  # 该类可以定义在一个py文件中然后导入py文件
    """配置参数"""
    DEBUG = True
app.config.from_object(Config)
```
- 通过json格式的文件配置

```Python
# config.json
{
    'DEBUG' = True
}
app.config.from_json('config.json') # 配置文件放置在与app的同目录下
```
- 直接操作app.config对象进行设置

```Python
app.config["DEBUG"] = True
或者
app.config.update({
    "DEBUG":True,
})
或者app.run(debug=True)
            
            
 if __name__ == '__main__':
    # 可以在环境变量中设定启动的host和端口；默认是 0.0.0.0:2333
    host = config.FLASK_HOST
    port = config.FLASK_PORT
    #所配置文件或者环境中的debug值就是这时候被传进去的
    app.run(host, port, debug=config.DEBUG)

```

## 加载请求钩子

请求钩子常用的有五种，它们通过装饰器的方式添加到app相应的存储字典(实例属性)中。

before_first_request：在处理第一个请求前运行。

before_request：在每次请求前运行。

after_request：如果没有未处理的异常抛出，在每次请求后运行。

teardown_request：在每次请求后运行，即使有未处理的异常抛出。

teardown_appcontext：在每次请求结束后应用上下文被弹出时执行，即appcontext调用pop方法；

```python
@app.before_request
def test():
    pass

@setupmethod
def before_request(self, f):
    self.before_request_funcs.setdefault(None, []).append(f)
    return f
```

## 加载错误处理方法

```python
@setupmethod
def errorhandler(
    self, code_or_exception: t.Union[t.Type[Exception], int]
) -> t.Callable[[T_error_handler], T_error_handler]:

    def decorator(f: T_error_handler) -> T_error_handler:
        self.register_error_handler(code_or_exception, f)
        #上面这句的核心 self.error_handler_spec[None][code][exc_class] = f 就是在给app实例属性添加东西了
        return f

    return decorator

# 其主要源码
def _register_error_handler(self, key, code_or_exception, f):
    # 可以传入状态码或我们自定义的异常
    exc_class, code = self._get_exc_class_and_code(code_or_exception)
    handlers = self.error_handler_spec.setdefault(key, {}).setdefault(code, {})
    handlers[exc_class] = f
```

## 加载蓝图

```python
players_api = Blueprint('conference-players', __name__)
flask_app.register_blueprint(players_api, url_prefix="/api/v1/players")

#flask init中的
self.blueprints: t.Dict[str, "Blueprint"] = {}
#Flask类中函数
@setupmethod
def register_blueprint(self, blueprint: "Blueprint", **options: t.Any) -> None:
    """Register a :class:`~flask.Blueprint` on the application. Keyword
    arguments passed to this method will override the defaults set on the
    blueprint.

    Calls the blueprint's :meth:`~flask.Blueprint.register` method after
    recording the blueprint in the application's :attr:`blueprints`.
    """
    blueprint.register(self, options)
```

