- [概念](#概念)
  - [连接查询](#连接查询)
  - [flush](#flush)
  - [数据库迁移](#数据库迁移)
    - [Alembic](#alembic)
  - [update\_time 和create\_time](#update_time-和create_time)
- [查询](#查询)
  - [ORM查询](#orm查询)
    - [query](#query)
    - [with\_entities](#with_entities)
    - [load](#load)
  - [row对象和model对象的处理](#row对象和model对象的处理)
  - [model对象](#model对象)
    - [row对象](#row对象)
  - [直接sql查询](#直接sql查询)

# 概念

## 连接查询

> 连接—创建会话—执行—关闭会话

```python
from sqlalchemy.orm import load_only, sessionmaker
from sqlalchemy import Column, String, create_engine, func
from sqlalchemy.orm import sessionmaker

# 初始化数据库连接:
engine = create_engine('mysql+mysqlconnector://root:password@localhost:3306/test')
# 创建DBSession类型:# create a configured "Session" class
DBSession = sessionmaker(bind=engine)
# 创建session对象:
session = DBSession()
# 创建新User对象:
new_user = User(id='5', name='Bob')
# 添加到session:
session.add(new_user)
# 提交即保存到数据库:
session.commit()
# 关闭session: 
session.close()
```

## flush

- Commit操作比较好理解，就是提交一次事务`Transaction`操作。既然是提交一次事务操作，就包含了增删改的SQL操作。所以必然是Session通过Connection进行写磁盘I/O的操作。
- Flush不同的是，它并没有真正的执行事务`Transaction`的操作，而是更新了数据库的事务缓存。**所以Flush是会和数据库进行通信的。Flush操作告知数据库把事务操作缓存在数据库，直到数据库收到了Commit操作之后才会真正将操作更新到磁盘中。**
- flush阶段执行sql语句（id生成），不持久化到磁盘，commit才持久化数据库文件，它 的主要动作就是向数据库发送一系列的sql语句（insert的sql），并执行这些sql语句，但是不会向数据库提交。
- **update和delete不需要手动flush，这两个是将对象先查到缓存中的，对象的修改也在 缓存中，同一事物内可见**
- update、delete之前都会先查出来封装成obj/row保存在缓存中，是个完整的数据库record对象，但是add到内存中的还不是（还缺字段、也没有联系上数据库，那么from table 就也没用），所以无法select
- update和delete的sql执行在buffer缓存中，形成脏页；insert的因为没有先查到buffer中，直接插入数据库中的，执行sql在flush阶段
  
## 数据库迁移

### Alembic

Alembic 是一款轻量型的数据库迁移工具，它与 SQLAlchemy 一起共同为 Python 提供数据库管理与迁移支持。

```python
#会自动在项目目录下创建一个 migrations 目录.  migrations 里面有一个 versions 文件夹，这个文件夹用于存放迁移脚本，
python flask_migrate_db.py db init 

#生成迁移文件   文件中有upgrade  downgrade(用于回退) 还有当前版本和所依赖的上一个版本，执行迁移的时候如果有跳过文件，则可能是依赖不是连续的
python flask_migrate_db.py db migrate -m "create table"

#执行迁移，从数据库中alembic_version所指向的开始执行
#执行 upgrade 命令相当于执行迁移脚本中的 upgrade() 函数，执行里面创建表的代码。
python flask_migrate_db.py db upgrade

# 回退到指定版本
python flask_migrate_db.py db downgrade 指定版本

#找到历史的版本号和
python flask_migrate_db.py db history 跟着执行的migrate走的，
```

## update_time 和create_time

```python
class TicketConf(Base, ModelBase):

    create_time = Column(
        DateTime,
        server_default=func.now(),    # server_default=sa.text('now()') 但在数据库那边的表现也是CURRENT_TIMESTAMP
        comment='创建时间'
    )
    update_time = Column(
        DateTime,
        server_default=func.now(),
        server_onupdate=func.now(),
        comment='更新时间'
    )
    
    
 或者：
 create_time = Column(
    DateTime,
    server_default=text('CURRENT_TIMESTAMP')   # server_default=sa.text('CURRENT_TIMESTAMP')
  )
  update_time = Column(
    DateTime,
    server_default=text('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP')
  )
```

# 查询

## ORM查询

**经过了ORM，所以查询出来的字段都是Model层的字段类型**

### query

- 返回结果是[('title1','20'),('title2','22')，row对象]   row对象  first返回的就是一个row，没有list;
- 返回结果是[model对象, model对象]   query(User)

```python
db.session.query(表名.字段名, 表名.字段名).all().   #<class 'sqlalchemy.engine.row.Row'>
```

### with_entities

- 结果是rows [('title1','20'),('title2','22'),('title3','30')    [row, row])

```python
db.session.query(表名).with_entities(表名.title, 表名.page).all()
```

### load

- 返回结果是 [model对象1，对象2].   Model对象     <class 'conference.models.model.Player'>

```python
with atomic() as db_session:
    query = db_session.query(Danmaku).options(load_only('id', 'fast_danmaku')).filter(Danmaku.id == 21).first()
    print(query)        #<conference.models.model.Danmaku object at 0x400abccf60>
    print(query.id)   #21
    print(type(query))  #<class 'conference.models.model.Danmaku'>
```


## row对象和model对象的处理

## model对象

```python
def as_dict_get_value__(self, attr):
    return getattr(self, attr, None)

def value_field_output_convert(self, columns, attr, value):
    """
    将资源对象转换为json的格式输出 
    查出来的某个字段的值进行转换
    """
    if value is None:
        return
    try:
        #获取列名即字段 的类型，是Model的字段类型，非数据库的字段类型
        ins_type = columns[attr].columns[0].type
        print(columns[attr])   #Activity.id  Activity.name
        print(columns[attr].columns[0]). #activity.id    activity.name   activity.start_time
        if isinstance(ins_type, DateTime):
            #time.strftime(format[, t])
            return value.strftime('%Y-%m-%d %H:%M:%S')
        if isinstance(ins_type, (Date, Numeric)):
            return str(value)
        if isinstance(ins_type, Enum):
            #emun就返回其value   数据库中是枚举，可处理；数据库中是tinyint，经过model层转为了枚举，也可处理
            if hasattr(value, 'value'):
                return value.value
            return value
    except Exception as e:
        logger.warning("exception e:%r", e)
    return value

def as_dict(self, depth=0):
    """
    转换db对象为dict， 是转换所有字段的，即便你只查两字段，也是转换所有字段
    传入的是db对象，即model对象；row不可
    """
    #SQLAIchemy的inspect 用于查看内存中对象的配置和构造，获取运行对象的状态
    ins = inspect(self).mapper #映射关系     mapped class Activity->activity   映射了所有的字段，哪怕你查出来的只有2字段
    #获取列   Activity.id .....所有Model字段
    columns = ins.column_attrs
    dict_data = {}
    for attr in columns.keys(): #字段名
        value = self.as_dict_get_value__(attr)
        value = self.value_field_output_convert(columns, attr, value)
        dict_data[attr] = value
    if not depth:
        return dict_data
    # 查询关联表对象
    relationships = ins.relationships
    for attr in relationships.keys():
        value = self.as_dict_get_value__(attr)
        rel = relationships[attr]
        if not rel.uselist:
            if value is None:
                dict_data[attr] = None
            else:
                dict_data[attr] = value.as_dict(depth=depth-1)
        else:
            dict_data[attr] = [x.as_dict(depth=depth - 1) for x in value]
    return dict_data
```

### row对象

```python
@classmethod
def row_to_dict(cls, row):
    """
    数据库查询结果row(通过orm查的row),返回字典  查出来的row直接全转
    转成model字段类型并进一步处理
    return:
        {"id": id, "name": name, ...}
    """
    if row is None:
        return row
    columns = inspect(cls).mapper.column_attrs
    _inst = cls()
    fields_ = row._fields
    return {
        field: _inst.value_field_output_convert(
            columns, field, getattr(row, field)
        ) for field in fields_
    }
```

## 直接sql查询

**不经过ORM，查询出来的字段类型和数据库一致**

```python
@classmethod
def filter_one_by_sql_execute(
    cls,
    db_session,
    sql_tpl,
    sql_params=None,
    to_dict=False,
    to_json=None,
    to_bool=None,
    dt_to_st=None,
    **kwargs
):
    """
    execute sql，防sql注入
    Params:
        sql_tpl: sql模板，需要替换的字段用":xxx"定义
        sql_params: sql参数字段，需与sql_tpl中定义保持一致。
            【注意！】枚举类传参时必须使用 .value 获取值
    execute params:
        https://github.com/sqlalchemy/sqlalchemy/blob/rel_1_4_6/lib/sqlalchemy/orm/session.py
    """
    query = db_session.execute(
        text(sql_tpl), params=sql_params
    ).fetchone()
    if to_dict and query:
        return cls._format_dict_data(query, to_json, to_bool, dt_to_st)

    return query

@classmethod
def filter_all_by_sql_execute(
        cls,
        db_session,
        sql_tpl,
        sql_params=None,
        to_dict=False,
        to_json=None,
        to_bool=None,
        dt_to_st=None,
        **kwargs
):
    """
    execute sql，防sql注入
    Params:
        sql_tpl: sql模板，需要替换的字段用":xxx"定义
        sql_params: sql参数字段，需与sql_tpl中定义保持一致。
            【注意！】枚举类传参时必须使用 .value 获取值
    execute params:
        https://github.com/sqlalchemy/sqlalchemy/blob/rel_1_4_6/lib/sqlalchemy/orm/session.py
    """
    query = db_session.execute(
        text(sql_tpl), params=sql_params
    ).fetchall()
    if to_dict and query:
        return [
            cls._format_dict_data(row_data, to_json, to_bool, dt_to_st) for row_data in query
        ]

    return query

@classmethod
def _format_dict_data(
    cls,
    query,
    to_json,
    to_bool,
    dt_to_st
):
    '''数据库的类型 tinyint转bool；str转json；datetime转str'''
    data = dict(query)
    if to_json:
        for key in to_json:
            if key not in data or data[key] is None:
                continue
            data[key] = json.loads(data[key])
    if to_bool:
        for key in to_bool:
            if key not in data or data[key] is None:
                continue
            data[key] = bool(data[key])
    if dt_to_st:
        for key in dt_to_st:
            if key not in data or data[key] is None:
                continue
            data[key] = trans_dt_to_ts(data[key])
    return data
```


