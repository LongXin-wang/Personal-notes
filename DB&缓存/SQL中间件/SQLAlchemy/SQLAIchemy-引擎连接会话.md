- [SQLAIchemy 框架](#sqlaichemy-框架)
  - [使用示例](#使用示例)
- [框架核心](#框架核心)
  - [Engine](#engine)
    - [Engine和Connection的关系](#engine和connection的关系)
  - [Pool](#pool)
  - [Dialect（方言）](#dialect方言)
- [schema](#schema)
  - [DBAPI](#dbapi)
- [Session](#session)
  - [使用示例](#使用示例-1)
  - [Session类](#session类)
  - [Scoped\_session](#scoped_session)
- [执行 SQL 语句](#执行-sql-语句)
  - [ResultProxy](#resultproxy)


# SQLAIchemy 框架

SQLAlchemy提供了丰富的API去完成数据库的交互任务，其中任务在两个层级完成，Core层和ORM层。核心层包括建立会话链接所需的引擎和连接池、数据库可执行语言SQL及模式管理，这些功能均已由API的形式可供调用。构建在Core层之上的是对象关系映射器（ORM）特定库，Core和ORM两种抽象层都可以完成数据持久化的任务。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403041427044.png)

核心层和ORM层的分离一直是SQLAlchemy最典型的特征，这个特征既有优点也有缺点。核心层的显式存在导致：

（一）ORM需要将映射到数据库的类属性关联到一个叫Table的结构上，而不是直接关联到数据库中表述的字符串属性名。

（二）ORM需要使用一个叫select的结构来产生SELECT查询，而不是直接将对象属性拼接成一个字符串的语句。

（三）ORM需要从ResultProxy接受结果行（ResultProxy自动将select映射到每个结果行），而不是直接操纵数据库游标(cursor)将数据转化成用户定义的对象。

## 使用示例

```python
# 创建连接数据库的引擎
eng = create_engine(
        'mysql+pymysql://root:Deve123456@127.0.0.1:3306/user?charset=utf8&autocommit=true',
        echo=True,  #设置为True，则输出sql语句
        encoding='utf-8',
        convert_unicode=True,
        case_sensitive=True,
        pool_recycle=1800,  #重连周期
        pool_pre_ping=True,
        pool_size=5,
        max_overflow=20  #连接池最大溢出容量，该容量+初始容量=最大容量。超出会堵塞等待，等待时间为timeout参数值默认30
    )
# 将引擎绑定到session类
Session = sessionmaker(bind=eng)
# 实例化session对象
ses = Session()
# ORM基类
Base = declarative_base()

class User(Base):
    __tablename__ = "user"

    id = Column(Integer, primary_key=True)
    name = Column(String)
    age = Column(Integer)

# Base.metadata.bind = eng
# Base.metadata.create_all()
ses.add_all(
   [User(id=2, name='张三', age=18),
    User(id=3, name='李四', age=19),
    User(id=4, name='王五', age=20)]
  )
ses.commit()

rs = ses.query(User).all()
```

注：create_engine的第一个参数url`mysql+pymysql` 中 + 号前面代表的是数据库类型，后面是使用的数据库驱动（其他：mysqldb）。pymysql是python的mysql数据库驱动，sqlachemy是一个数据库框架，需要利用使用DBAPI规范实现的驱动（pymysql）来操作数据库。


# 框架核心

> https://zhuanlan.zhihu.com/p/435651476

* 使用url语法查找并动态加载数据库方言
* 创建引擎对象，方言，连接池，提供SQL的API
* 使用引擎对象获取到数据库链接connect，获取后的链接使用pool管理
* 执行SQL语句并获取执行结果

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403042028944.png)

Engine、Connection两个类的execute方法返回的结果是一个ResultProxy，它提供了一个与DBAPI的游标类似但功能更丰富的接口。Engine，Connection和ResultProxy分别对应于DBAPI模块、一个具体的DBAPI连接对象，和一个具体的DBAPI游标对象。

在底层，Engine引用了一个叫Dialect的对象。Dialect是一个有众多实现的抽象类，它的每一个实现都对应于一个具体的DBAPI和数据库。一个为Engine而创建的Connection会咨询Dialect作出选择，对于不同的目标DBAPI和数据库，Connection的行为都不一样。

Connection创建时会从一个连接池获取并维护一个DBAPI的连接，这个连接池叫Pool，也和Engine相关联。Pool负责创建新的DBAPI连接，通常在内存中维护DBAPI连接池，供频繁的重复使用。

在一个语句执行的过程中，Connection会创建一个额外的ExecutionContext对象。这个对象从开始执行的时刻，一直存在到ResultProxy消亡为止。

## Engine

Engine 是访问数据库的入口，Engine引用Connection Pool和 Dialect实现了对数据库的访问, Dialect指定了具体的数据库类型 MYSQL, SQLSERVER等, 三者关系如图所示: 

观察每个EngineStrategy子类的create方法，发现它们都会在创建Engine对象之前先创建Dialect对象和Pool对象，并将这两个对象的引用保存在Engine对象中，保证了Engine对象可以通过Dialect和Pool处理DBAPI。

![file-list](http://docs.sqlalchemy.org/en/rel_1_0/_images/sqla_engine_arch.png)

直接使用 create_engine初始化engine 时，就会创建一个带连接池的引擎，并不会立即创建连接，只有当调用connect(),execute()函数的时候，才会创建数据库的连接

session 和 connection 不是相同的东西， session 使用连接来操作数据库，一旦任务完成 session 会将数据库 connection 交还给 pool。在使用 create_engine 创建引擎时，如果默认不指定连接池设置的话，一般情况下，SQLAlchemy 会使用一个 QueuePool 绑定在新创建的引擎上。并附上合适的连接池参数。

*engine主要功能就是管理和持有connection，pool和dialect，对外提供API。*

```python
# engine/__init__.py
"""使用策略模式去创建不同的engine实现"""
from . import strategies

default_strategy = "plain"  # 默认

def create_engine(*args, **kwargs):
    strategy = kwargs.pop("strategy", default_strategy)
    strategy = strategies.strategies[strategy]
    return strategy.create(*args, **kwargs)


# engine/strategies.py
"""默认的engine策略"""
strategies = {}

class EngineStrategy(object):

    def __init__(self):
        strategies[self.name] = self

class DefaultEngineStrategy(EngineStrategy):
    
    def create(self, name_or_url, **kwargs):
        ...

class PlainEngineStrategy(DefaultEngineStrategy):
    name = "plain"
    engine_cls = base.Engine  # 引擎类

PlainEngineStrategy()


"""重点就在策略的create方法"""
def create(self, name_or_url, **kwargs):
    ...
    # get dialect class
    u = url.make_url(name_or_url)
    entrypoint = u._get_entrypoint()
    dialect_cls = entrypoint.get_dialect_cls(u)
    
    # create dialect
    dialect = dialect_cls(**dialect_args)
    
    # pool
    poolclass = dialect_cls.get_pool_class(u)
    pool = poolclass(creator, **pool_args)
    
    # engine
    engineclass = self.engine_cls
    engine = engineclass(pool, dialect, u, **engine_args)
    ...
    return engine


class Engine(Connectable, log.Identified):
    _connection_cls = Connection
    
    def __init__(
        self,
        pool,
        dialect,
        url,
        logging_name=None,
        echo=None,
        proxy=None,
        execution_options=None,
    ):
        self.pool = pool
        self.url = url
        self.dialect = dialect
        self.engine = self
        ...
    
    def connect(self, **kwargs):
        return self._connection_cls(self, **kwargs)

```

### Engine和Connection的关系

`Engine.connect()`方法用于创建`Connection`：

```python
_connection_cls = Connection

def connect(self, **kwargs):
    return self._connection_cls(self, **kwargs)
```

调用`Engine.connect()`实际上是将`Engine`对象自己作为第一个参数传入了`Connection`的构造函数，但`Connection`的构造函数还要调用`Engine.raw_connection()`方法获得数据库连接。这样做主要是为了方便`Engine`的隐式执行接口。在`Connection`没有创建的时候，`Engine`也可以自己调用`raw_connection()`获得数据库连接。

最后，`Connection`拥有`Engine`的引用，并可以通过`Engine`的访问`Pool`和`Dialect`对象。对数据库的操作通常是使用`Connection.execute()`方法进行的。

## Pool

`Pool`负责管理DBAPI连接。`Connection`对象创建时，会从连接池中取出一个DBAPI连接，而在`close`()[^2]方法调用时，会将连接归还。

[^2]: 使用scoped_session类来管理session时，使用的remove()方法实际上就是close()

`Pool`的代码定义在`pool.py` 中，即目录：`sqlalchemy/pool/impl.py`包括抽象父类`Pool`，和几个有具体功能的子类：

- `QueuePool` 限制连接个数（默认使用）

- `SingletonThreadPool` 为每个线程维护一个连接

- `AssertionPool` 任何时候都只允许一个连接，否则抛出异常

- `NullPool` 不进行任何池操作，直接打开/关闭DBAPI连接

- `StaticPool` 有且仅有一个连接


```python
class Connection(Connectable):
    
    def __init__(
        self,
        engine,
        connection=None,
        close_with_result=False,
        _branch_from=None,
        _execution_options=None,
        _dispatch=None,
        _has_events=None,
    ):
        self.engine = engine
        self.dialect = engine.dialect
        self.__connection =  engine.raw_connection()
        ...


"""connection主要使用engine.raw_connection创建了一个DBAPI连接"""
class Engine(Connectable, log.Identified):
    
    def raw_connection(self, _connection=None):
        return self._wrap_pool_connect(
            self.pool.unique_connection, _connection
        )


    def _wrap_pool_connect(self, fn, connection):
        dialect = self.dialect
        try:
            return fn()
        except dialect.dbapi.Error as e:
            ...


"""pool.unique_connection负责创建数据库连接"""
```

## Dialect（方言）

**不同方言实现需要提供一个dialect对象**

SQLAlchemy本身无法操作数据库，其必须依赖pymsql、mysqldb等第三方插件，Dialect用于和数据API进行交流，为了在上层的ORM层做了无差别调用，根据配置文件的不同调用不同的数据库API，从而实现对数据库的操作。

`Dialect`定义在engine/interfaces.py文件中，是一个抽象的接口，其中定义了三个`do_execute*()`方法，分别是`do_execute()`，`do_executemany()`和`do_execute_no_params()`。`Dialect`的子类通过实现这些接口来定义自己执行时的行为。SQLAlchemy中默认的dialect子类是`DefaultDialect`。在默认实现中，`do_execute`方法调用`Cursor.execute`，而`Cursor`是来自DBAPI的类。在这里，SQLAlchemy核心层和DBAPI层连接了起来。

Dialect是一个有众多实现的抽象类，它的每一个实现都对应于一个具体的DBAPI和数据库。一个为Engine而创建的Connection会咨询Dialect作出选择，对于不同的目标DBAPI和数据库，Connection的行为都不一样。

# schema

**模式定义与ORM分离的原理**

Table和Column模型包含在一个叫做metadata（元数据）的概念内，用一个叫MetaData的集合对象代表Table对象的集合。

Table表示了目标schema中一个实际的表的名字等属性，它所含的Column对象集合代表了每个表中字段的名字和类型信息。Table中还含有一系列完整的描述constraint, index, sequence的对象，其中一些对引擎和SQL构建系统的行为影响很大。特别的，ForeignKeyConstraint在决定两个表如何进行连接上非常关键。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202403061504001.png)



## DBAPI

注：什么是DB-API，以下内容引用自SQLAlchemy文档的术语表：

> DBAPI是“Python数据库API规范”（Python Database API Specification）的简称。这是在Python中广泛使用的规范，定义了数据库连接的第三方库的使用模式。DBAPI是一个低层的API，在一个Python应用中基本上位于最底层，和数据库直接进行交互。SQLAlchemy的方言系统按照DBAPI的操作来构建。基本上，一个方言就是DBAPI加上一个特定的数据库引擎。通过在`create_engine()`函数中提供不同的数据库URL可以将方言绑定到不同的数据库引擎上。

PEP的文档介绍比较枯燥，我们可以通过这个示例代码直观地理解DBAPI的使用模式：

```python
connection = dbapi.connect(user="root", pw="123456", host="localhost:8000")
cursor = connection.cursor()
cursor.execute("select * from user_table where name=?", ("jack",))
print "Columns in result:", [desc[0] for desc in cursor.description]
for row in cursor.fetchall():
    print "Row:", row
cursor.close()
connection.close()
```

作为对比，SQLAlchemy的使用模式是这样的：

```python
engine = create_engine("postgresql://user:pw@host/dbname")
connection = engine.connect()
result = connection.execute("select * from user_table where name=?", "jack")
print result.fetchall()
connection.close()
```
可以看到，二者的使用模式非常相似，都是直接通过SQL语句进行查询。SQLAlchemy只进行了封装，但没有进行高层次的抽象。

# Session

## 使用示例

**用法一**

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import Session
 
# an Engine, which the Session will use for connection
# resources
engine = create_engine("postgresql://scott:tiger@localhost/")
 
# create session and add objects
with Session(engine) as session:
    session.add(some_object)
    session.add(some_other_object)
    session.commit()
```

**用法二**

sessionmaker()创建了一个工厂类，在创建这个工厂类时配置了参数绑定了引擎。将其赋值给Session对象（会话）。每次实例化Session都会创建一个绑定了引擎的Session。这样这个session在访问数据库时都会通过这个绑定好的引擎来获取连接资源，当实现具体的业务逻辑时， 可以将sessionmaker 进行初步封装，视作应用程序配置的一部分。

```python
# 初始化数据库连接:
engine = create_engine('mysql+mysqlconnector://root:XXXXXhost.docker.internal:3306/yaotai_backend_pytest')
# 创建DBSession类型:
DBSession = sessionmaker(bind=engine)
# 创建session对象:
session = DBSession()
# 创建新User对象:
new_user = User(
    uid='086465653',
    phone='86+1230321',
    register=True,
    register_method=1,
    create_time=datetime.now()
)
# 添加到session:
session.add(new_user)
# 提交即保存到数据库:
session.commit()
# 关闭session:   会话结束不用的话需要关闭释放资源   事物提交会话可以不用关闭，一次会话可以有多个事物
session.close()
```

## Session类

* 数据库操作需要通过session，有query() 、execute()、add()、bulk_update_mappings()\bulk_save_objects()等,所以session可以直接执行增操作，但是查、改、删（改/删都需要先查出来）需要借助query对象
* query对象有filiter、with_entities、filter_by、all、update、delete等可以被调用
* 有enter和exit，exit中是close，可以用with...as... 但需要手动commit

```python

class Session(_SessionClassMethods):
   def __init__(
       self,
       bind=None,
       autoflush=True,
       future=False,
       expire_on_commit=True,
       autocommit=False,
       twophase=False,
       binds=None,
       enable_baked_queries=True,
       info=None,
       query_cls=None,
   ):
    
   
    def query(self, *entities, **kwargs):
       """Return a new :class:`_query.Query` object corresponding to this
       :class:`_orm.Session`.
 
       """
 
       return self._query_cls(entities, self, **kwargs)


def create_engine(url, **kwargs):
    'Create a new :class:`_engine.Engine` instance.'
 
        engine = create_engine("postgresql://scott:tiger@localhost/test")
 
# 初始化数据库连接:
engine = create_engine('mysql+mysqlconnector://root:password@localhost:3306/test')
# 创建DBSession类型:# create a configured "Session" class
#Session就是具体产品类，sessionmaker是工厂类
DBSession = sessionmaker(bind=engine)
# 创建session对象: 调用call()
session = DBSession()


class sessionmaker(_SessionClassMethods):
    def __init__(
        self,
        bind=None,
        class_=Session,   #把Session类传了进来，这个工厂类用此创建session对象
        autoflush=True,
        autocommit=False,
        expire_on_commit=True,
        info=None,
        **kw
    ): #除了class_,其他的Session类中都有；call会把这些传入到Sesion类中
   
        kw["bind"] = bind
        kw["autoflush"] = autoflush
        kw["autocommit"] = autocommit
        kw["expire_on_commit"] = expire_on_commit
        if info is not None:
            kw["info"] = info
        self.kw = kw
        # make our own subclass of the given class, so that
        # events can be associated with it specifically. class_就是Session类
        self.class_ = type(**class_**.__name__, (class_,), {})
     
    #call表示 用此类生成的对象可以被当作方法调用，调用的就是call方法,就是返回Session对象了
   def __call__(self, **local_kw):
**      """Produce a new :class:`.Session` object using the configuration
          Session = sessionmaker()
          session = Session()  # invokes sessionmaker.__call__()
 
      """
      for k, v in self.kw.items():
          if k == "info" and "info" in local_kw:
              d = v.copy()
              d.update(local_kw["info"])
              local_kw["info"] = d
          else:
              local_kw.setdefault(k, v)
      return self.class_(**local_kw)    #返回的就是Session类的对象,并把那些参数传入构造了Session对象
```

## Scoped_session

* 线程隔离的
* ScopedSessionMixin 中有call方法，调用 scoped_session(sessionmaker)() 相当于调用此call
* 而`()` 相当于调用了ThreadLocalRegistry中的call

```python
session = scoped_session(_get_session(bind_name))()
 
class scoped_session(ScopedSessionMixin):
   def __init__(self, session_factory, scopefunc=None):
     
        self.session_factory = session_factory
 
        if scopefunc:
            #按照自己的设计来生成
            self.registry = ScopedRegistry(session_factory, scopefunc)
        else:
        #
            self.registry = ThreadLocalRegistry(session_factory)
             
      #   ScopedSessionMixin 中的
      def __call__(self, **kw):
 
        if kw:
            if self.registry.has():
                raise sa_exc.InvalidRequestError(
                    "Scoped session is already present; "
                    "no new arguments may be specified."
                )
            else:
                sess = self.session_factory(**kw)
                self.registry.set(sess)
        else:
            sess = self.registry()
        if not self._support_async and sess._is_asyncio:
            warn_deprecated(
                "Using `scoped_session` with asyncio is deprecated and "
                "will raise an error in a future version. "
                "Please use `async_scoped_session` instead.",
                "1.4.23",
            )
        return sess

class ThreadLocalRegistry(ScopedRegistry):
    """A :class:`.ScopedRegistry` that uses a ``threading.local()``
    variable for storage.
 
    """
 
    def __init__(self, createfunc):
        self.createfunc = createfunc
        self.registry = threading.local()    #dict
 
    def __call__(self):
        try:
            return self.registry.value
        except AttributeError:
            val = self.registry.value = self.createfunc()
            return val
   
```

# 执行 SQL 语句

Connection的execute方法执行一个SQL语句，并返回一个ResultProxy对象。execute方法接受多种类型的参数，参数类型可以是一个字符串，也可以是ClauseElement和Executable的共同子类。

- 利用dialect创建context上下文
- 使用dialect执行sql语句(文本)
- 使用context获取执行的结果并返回

```python
def execute(self, object, *multiparams, **params):
    if isinstance(object, util.string_types[0]):
        return self._execute_text(object, multiparams, params)
    try:
        meth = object._execute_on_connection
    except AttributeError:
        raise exc.InvalidRequestError(
        "Unexecutable object type: %s" %
        type(object))
    else:
        return meth(self, multiparams, params)

def _execute_text(self, statement, multiparams, params):
        """Execute a string SQL statement."""

        dialect = self.dialect
        parameters = _distill_params(multiparams, params)
        ret = self._execute_context(
            dialect,
            dialect.execution_ctx_cls._init_statement,
            statement,
            parameters,
            statement,
            parameters,
        )
        return ret

def _execute_context(
        self, dialect, constructor, statement, parameters, *args
    ):
    conn = self.__connection
    ...
    context = constructor(dialect, self, conn, *args)
    ...
    cursor, statement, parameters = (
            context.cursor,
            context.statement,
            context.parameters,
        )
    ...
    self.dialect.do_execute(
                        cursor, statement, parameters, context
                    )
    ...
    result = context._setup_crud_result_proxy()
    return result
```

可以看到，execute方法对SQL语句对象的类型进行判断，如果是一个字符串，则调用_execute_text方法执行，否则调用对象的_execute_on_connection方法，而不同对象的_execute_on_connection方法会调用Connection._execute_*()方法，具体为：

- sql.FunctionElement对象，调用Connection._execute_function()
- schema.ColumnDefault对象，调用Connection._execute_default()
- schema.DDL对象，调用Connection._execute_ddl()
- sql.ClauseElement对象，调用Connection._execute_clauseelement()
- sql.Compiled对象，调用Connection._execute_compiled()

以上的Connection._execute_*()方法都调用了Connection._execute_context()方法。这个方法的第一个参数是Dialect对象，第二个参数是ExecutionContext对象的构造器（构造器从Dialect对象获取）。在方法中调用构造器构造了一个ExecutionContext对象，并根据context对象的状态调用dialect对象的相关方法产生结果。对不同的状态，调用的dialect对象的方法不同，总的来说，调用的是Dialect.do_execute*()方法。

从上述方法调用的过程可以看出，Connection的execute方法最终将生成结果的任务转交给了Dialect的do_execute方法。SQLAlchemy正是用这种方法应对多变的DBAPI实现的：Connection在执行SQL语句的时候，会咨询Dialect作出选择。因此对于不同的目标DBAPI和数据库，Connection的行为都不一样。

## ResultProxy

ResultProxy包装了一个DBAPI游标(cursor)对象，使一行结果中的各个字段更容易访问。在数据库术语中，结果通常称为一个行(row)。

一个字段可以通过三种方式访问：

```python
row = fetchone()
col1 = row[0] # 通过位置下标访问
col2 = row['col2'] # 通过名字访问
col3 = row[mytable.c.mycol] # 通过Column对象访问
```

