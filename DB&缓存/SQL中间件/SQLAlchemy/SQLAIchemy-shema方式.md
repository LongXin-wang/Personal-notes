- [schema方式 \& orm方式](#schema方式--orm方式)
  - [使用](#使用)
    - [orm 方式](#orm-方式)
    - [schema 方式](#schema-方式)
- [Schema方式 解读](#schema方式-解读)
  - [DDL（data definition language）创建 table](#ddldata-definition-language创建-table)
  - [DML](#dml)
  - [DQL](#dql)
- [ORM - Model核心功能](#orm---model核心功能)

# schema方式 & orm方式

**sqlalchemy可以在低层次上提供了sql语句的方式使用；在次层次上提供定义schema方式使用；在高层次上提供orm的实现，让应用可以根据项目的特点自主选择不同层级的API**

SQLAlchemy在核心层上提供了SQL语句执行的方式以供使用；也提供定义schema方式使用；在ORM层上提供orm的实现，让应用可以根据项目的特点能自由选择不同层级的API。

使用schema时候，主要使用Metadata，Table和Column等对象去定义Schema数据结构，使用编译器自动将schema转换成合法的SQL语句，使用connecttion操作数据。

使用orm的时候，则是创建特定的数据模型，**模型对象会隐式创建schema**，通过session方式进行数据访问。

*orm方式和schema方式都是向MeatData中添加table和column*

使用SQLAlchemy核心来定义schema，定义好的schema仍然需要经过创建engine以及如果需要的话创建连接池的过程。使用SQLAlchemy核心定义的schema仍然需要与engine和连接池相关联，以便执行针对数据库的SQL查询。虽然它跳过了ORM层，但仍然需要这些组件进行数据库交互。

|             | schema方式                   | orm方式                                    |
| ----------- | ---------------------------- | ------------------------------------------ |
| 创建engine  | 创建metadata，用于管理schema | 用于数据库连接                             |
| 创建model   | 创建users表的Table           | 创建User模型将metadata提交到engine(创建表) |
| 创建session | 使用execute插入数据dict      | 使用session插入数据Use                     |

## 使用

### orm 方式

```python
from sqlalchemyimport create_engine
from sqlalchemy.ext.declarativeimport declarative_base
from sqlalchemyimport Column, Integer, String
from sqlalchemy.ormimport *

engine = create_engine(
    'mysql+pymysql://root:Deve123456@127.0.0.1:3306/user?charset=utf8&autocommit=true',
    echo=True,  # 设置为True，则输出sql语句 
    encoding='utf-8')
# 将引擎绑定到session类
Session = sessionmaker(bind=engine)
# 实例化session对象
session = Session()
# orm的方式
Base = declarative_base()  
# ORM基类
class User(Base):
    __tablename__ = "user"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    image_id = Column(String)
    age = Column(Integer)

session.add_all([User(name='张三', age=18), User(name='李四', age=19), User(name='王五', age=20)])session.commit()
# 添加
six_user = User(name='小六', age=21)
session.add(six_user)session.commit()
```

### schema 方式

定义 schema 结构、创建引擎、获取连接、执行数据库操作、获取结果

```python
metadata = MetaData()
user = Table('user', metadata, 
    Column('id', Integer, primary_key=True),             
    Column('name', String(16), nullable=False),             
    Column('age', String(60))             
)
Model.metadata.create_all(engine)  
# 创建表格，数据库里已存在对应表格有就不用
for tin metadata.sorted_tables:    
    print(t.name)
# 添加
connection = engine.connect()
schema_user = future.insert()
six_user = dict(name='小六', age=21)
result = connection.execute(schema_user, six_user)
session.commit()
print(six_user.id)
# 查询
result = connection.execute(select([user])).where(users.c.name == '小六')
for rowin result:    
print(row)
```

# Schema方式 解读

> https://zhuanlan.zhihu.com/p/435652219

## DDL（data definition language）创建 table

*metadata，table和column的数据结构：metadata持有table集合，table持有column集合*

```python
"""schema的基础实现visitable
Visitable约定子类必须提供 visit_name 的类属性，用来绑定编译方法。参与sql的类都继承自Visitable"""
class VisitableType(type):

    def __init__(cls, clsname, bases, clsdict):
        if clsname != "Visitable" and hasattr(cls, "__visit_name__"):
            _generate_dispatch(cls)
        super(VisitableType, cls).__init__(clsname, bases, clsdict)

def _generate_dispatch(cls):
    if "__visit_name__" in cls.__dict__:
        visit_name = cls.__visit_name__
        if isinstance(visit_name, str):
            getter = operator.attrgetter("visit_%s" % visit_name)

            def _compiler_dispatch(self, visitor, **kw):
                try:
                    meth = getter(visitor)
                except AttributeError:
                    raise exc.UnsupportedCompilationError(visitor, cls)
                else:
                    return meth(self, **kw)
        
        cls._compiler_dispatch = _compiler_dispatch

class Visitable(util.with_metaclass(VisitableType, object)):
    pass


class SchemaItem(SchemaEventTarget, visitors.Visitable):
    __visit_name__ = "schema_item"
    
class MetaData(SchemaItem):
    __visit_name__ = "metadata"
    
class Table(DialectKWArgs, SchemaItem, TableClause):
    __visit_name__ = "table"

class Column(DialectKWArgs, SchemaItem, ColumnClause):
    __visit_name__ = "column"

class TypeEngine(Visitable):
    ...

class Integer(_LookupExpressionAdapter, TypeEngine):
    __visit_name__ = "integer"


"""MetaData是schema的集合，记录了所有的Table定义, 通过 _add_table 函数用来添加表"""
class MetaData(SchemaItem):
    def __init__(
        self,
        bind=None,
        reflect=False,
        schema=None,
        quote_schema=None,
        naming_convention=None,
        info=None,
    ):
        # table集合
        self.tables = util.immutabledict()
        
        self.schema = quoted_name(schema, quote_schema)
        self._schemas = set()
    
    def _add_table(self, name, schema, table):
        key = _get_table_key(name, schema)
        dict.__setitem__(self.tables, key, table)
        if schema:
            self._schemas.add(schema)


"""Table是column的集合，在创建table对象的时候，把自己添加到metadata中"""
class Table(DialectKWArgs, SchemaItem, TableClause):
    
    def __new__(cls, *args, **kw):
        name, metadata, args = args[0], args[1], args[2:]
        schema = metadata.schema
        table = object.__new__(cls)
        # 添加到metadata
        metadata._add_table(name, schema, table)
        table._init(name, metadata, *args, **kw)
        return table
    
    def _init(self, name, metadata, *args, **kwargs):
        super(Table, self).__init__(
            quoted_name(name, kwargs.pop("quote", None))
        )
        self.metadata = metadata
        self.schema = metadata.schema
        # column集合
        self._columns = ColumnCollection()
        self._init_items(*args)
    
    def _init_items(self, *args):
        # column 
        for item in args:
            if item is not None:
                item._set_parent_with_dispatch(self)


"""Column是通过下面的方法将column添加到table的colummns中"""
class Column(DialectKWArgs, SchemaItem, ColumnClause):
    
    def __init__(self, *args, **kwargs):
        pass
        
    def _set_parent(self, table):
        table._columns.replace(self)

class ColumnCollection(util.OrderedProperties):
    def replace(self, column):
        ...
        self._data[column.key] = column
        ...

```

*数据结构如何转换成sql语句，API是通过 MetaData.create_all*

```python
class MetaData(SchemaItem):
    def create_all(self, bind=None, tables=None, checkfirst=True):
        # bing = engine
        bind._run_visitor(
            ddl.SchemaGenerator, self, checkfirst=checkfirst, tables=tables
        )

class Engine(Connectable, log.Identified):
    def _run_visitor(
        self, visitorcallable, element, connection=None, **kwargs
    ):
        # 上下文管理器获取连接conn
        with self._optional_conn_ctx_manager(connection) as conn:
            # visitorcallable = ddl.SchemaGenerator
            conn._run_visitor(visitorcallable, element, **kwargs)
            
class Connection(Connectable):
    def _run_visitor(self, visitorcallable, element, **kwargs):
        visitorcallable(self.dialect, self, **kwargs).traverse_single(element)


"""create-table的sql编译主要由ddl中的SchemaGenerator实现, 下面是SchemaGenerator的继承关系和核心的traverse_single函数"""
class ClauseVisitor(object):

    def traverse_single(self, obj, **kw):
        # 遍历所有的visit实现 
        for v in self.visitor_iterator:
            meth = getattr(v, "visit_%s" % obj.__visit_name__, None)
            if meth:
                return meth(obj, **kw)
    
    @property
    def visitor_iterator(self):
        v = self
        while v:
            yield v
            v = getattr(v, "_next", None)
            
class SchemaVisitor(ClauseVisitor):
    ...
    
class DDLBase(SchemaVisitor):
    ...

class SchemaGenerator(DDLBase):
    ...

"""创建meta，table和columun的过程:"""
class SchemaGenerator(DDLBase):
    def visit_metadata(self, metadata):
        tables = list(metadata.tables.values())
        collection = sort_tables_and_constraints(
            [t for t in tables if self._can_create_table(t)]
        )
        for table, fkcs in collection:
            if table is not None:
                # 创建表 
                self.traverse_single(
                    table,
                    create_ok=True,
                    include_foreign_key_constraints=fkcs,
                    _is_metadata_operation=True,
                )
            
    def visit_table(
        self,
        table,
        create_ok=False,
        include_foreign_key_constraints=None,
        _is_metadata_operation=False,
    ):
        for column in table.columns:
            if column.default is not None:
                # 创建column-DDLElement
                self.traverse_single(column.default)
    
        self.connection.execute(
            # fmt: off
            # 创建create-table-DDLElement
            CreateTable(
                table,
                include_foreign_key_constraints=  # noqa
                    include_foreign_key_constraints,
            )
            # fmt: on
        )

class _DDLCompiles(ClauseElement):
    def _compiler(self, dialect, **kw):
        return dialect.ddl_compiler(dialect, self, **kw)
        
class DDLElement(Executable, _DDLCompiles):
    ...

class _CreateDropBase(DDLElement):
    ...
    
class CreateTable(_CreateDropBase):

    __visit_name__ = "create_table"
    
    def __init__(
        self, element, on=None, bind=None, include_foreign_key_constraints=None
    ):
        super(CreateTable, self).__init__(element, on=on, bind=bind)
        self.columns = [CreateColumn(column) for column in element.columns]

class CreateColumn(_DDLCompiles):
    __visit_name__ = "create_column"

    def __init__(self, element):
        self.element = element

"""最终这些DDLElement在compiler中被DDLCompiler编译成sql语句, CREATE TABLE是这样被编译的"""
def visit_create_table(self, create):
    """
    根据传入的 create 对象生成创建表的 SQL 语句。它首先获取要创建的表 table 和一个预处理器 preparer。然后，它构建创建表的 SQL 语句，包括表名、前缀、表后缀等。接下来，它通过遍历 create 对象的列（columns）来处理每个列，生成列的 SQL 语句，并在必要时添加主键等约束。最后，它构建并返回创建表的完整 SQL 语句。
    """
    table = create.element
    preparer = self.preparer

    text = "\nCREATE "
    if table._prefixes:
        text += " ".join(table._prefixes) + " "
    text += "TABLE " + preparer.format_table(table) + " "

    create_table_suffix = self.create_table_suffix(table)
    if create_table_suffix:
        text += create_table_suffix + " "

    text += "("

    separator = "\n"

    # if only one primary key, specify it along with the column
    first_pk = False
    for create_column in create.columns:
        column = create_column.element
        try:
            processed = self.process(
                create_column, first_pk=column.primary_key and not first_pk
            )
            if processed is not None:
                text += separator
                separator = ", \n"
                text += "\t" + processed
            if column.primary_key:
                first_pk = True
        except exc.CompileError as ce:
            ...

    const = self.create_table_constraints(
        table,
        _include_foreign_key_constraints=create.include_foreign_key_constraints,  # noqa
    )
    if const:
        text += separator + "\t" + const

    text += "\n)%s\n\n" % self.post_create_table(table)
    return text

def visit_create_column(self, create, first_pk=False):
    """
     create 对象生成创建列的 SQL 语句。它首先获取要创建的列 column。然后，它调用 get_column_specification 方法生成列的规范，并在必要时添加约束。最后，它返回创建列的 SQL 语句    
    """
    column = create.element

    text = self.get_column_specification(column, first_pk=first_pk)
    const = " ".join(
        self.process(constraint) for constraint in column.constraints
    )
    if const:
        text += " " + const

    return text
```

## DML

*数据插入的API由TableClause提供的insert函数*

```python
class TableClause(Immutable, FromClause):
    
    @util.dependencies("sqlalchemy.sql.dml")
    def insert(self, dml, values=None, inline=False, **kwargs):
        return dml.Insert(self, values=values, inline=inline, **kwargs)

class UpdateBase(
    HasCTE, DialectKWArgs, HasPrefixes, Executable, ClauseElement
):
    ...
    
class ValuesBase(UpdateBase):
    ...
    
class Insert(ValuesBase):
    __visit_name__ = "insert"
    ...

class SQLCompiler(Compiled):
    
    def visit_insert(self, insert_stmt, asfrom=False, **kw):

        crud_params = crud._setup_crud_params(
            self, insert_stmt, crud.ISINSERT, **kw
        )

        if insert_stmt._has_multi_parameters:
            crud_params_single = crud_params[0]
        else:
            crud_params_single = crud_params

        preparer = self.preparer
        supports_default_values = self.dialect.supports_default_values

        text = "INSERT "

        text += "INTO "
        table_text = preparer.format_table(insert_stmt.table)

        if crud_params_single or not supports_default_values:
            text += " (%s)" % ", ".join(
                [preparer.format_column(c[0]) for c in crud_params_single]
            )
        ...
        if insert_stmt.select is not None:
            select_text = self.process(self._insert_from_select, **kw)

            if self.ctes and toplevel and self.dialect.cte_follows_insert:
                text += " %s%s" % (self._render_cte_clause(), select_text)
            else:
                text += " %s" % select_text
        elif not crud_params and supports_default_values:
            text += " DEFAULT VALUES"
        elif insert_stmt._has_multi_parameters:
            text += " VALUES %s" % (
                ", ".join(
                    "(%s)" % (", ".join(c[1] for c in crud_param_set))
                    for crud_param_set in crud_params
                )
            )
        else:
            text += " VALUES (%s)" % ", ".join([c[1] for c in crud_params])

        return text
```

## DQL

```python
class SelectBase(HasCTE, Executable, FromClause):
    ...
    
class GenerativeSelect(SelectBase):
    ...
    
class Select(HasPrefixes, HasSuffixes, GenerativeSelect):
    __visit_name__ = "select"
    
    def __init__(
        self,
        columns=None,
        whereclause=None,
        from_obj=None,
        distinct=False,
        having=None,
        correlate=True,
        prefixes=None,
        suffixes=None,
        **kwargs
    ):
        GenerativeSelect.__init__(self, **kwargs)
        ...

class SQLCompiler(Compiled):
    
    def visit_select(
        self,
        select,
        asfrom=False,
        parens=True,
        fromhints=None,
        compound_index=0,
        nested_join_translation=False,
        select_wraps_for=None,
        lateral=False,
        **kwargs
    ):
        ...
        
        froms = self._setup_select_stack(select, entry, asfrom, lateral)

        column_clause_args = kwargs.copy()
        column_clause_args.update(
            {"within_label_clause": False, "within_columns_clause": False}
        )

        text = "SELECT "  # we're off to a good start !

        text += self.get_select_precolumns(select, **kwargs)
        # the actual list of columns to print in the SELECT column list.
        inner_columns = [
            c
            for c in [
                self._label_select_column(
                    select,
                    column,
                    populate_result_map,
                    asfrom,
                    column_clause_args,
                    name=name,
                )
                for name, column in select._columns_plus_names
            ]
            if c is not None
        ]
        
        ...
        
        text = self._compose_select_body(
            text, select, inner_columns, froms, byfrom, kwargs
        )

        if select._statement_hints:
            per_dialect = [
                ht
                for (dialect_name, ht) in select._statement_hints
                if dialect_name in ("*", self.dialect.name)
            ]
            if per_dialect:
                text += " " + self.get_statement_hint_text(per_dialect)

        if self.ctes and toplevel:
            text = self._render_cte_clause() + text

        if select._suffixes:
            text += " " + self._generate_prefixes(
                select, select._suffixes, **kwargs
            )

        self.stack.pop(-1)

        if (asfrom or lateral) and parens:
            return "(" + text + ")"
        else:
            return text
```

# ORM - Model核心功能

Model类使用declarative_base动态创建:

- Model类的构造函数__init__使用_declarative_constructor
- Model类的子类在构造的时候会调用_as_declarative
- model对象会使用_add_attribute进行赋值

```python
class DeclarativeMeta(type):
    def __init__(cls, classname, bases, dict_):
        if "_decl_class_registry" not in cls.__dict__:
            _as_declarative(cls, classname, cls.__dict__)
        type.__init__(cls, classname, bases, dict_)

    def __setattr__(cls, key, value):
        _add_attribute(cls, key, value)

    def __delattr__(cls, key):
        _del_attribute(cls, key)

def declarative_base(
    bind=None,
    metadata=None,
    mapper=None,
    cls=object,
    name="Base",
    constructor=_declarative_constructor,
    class_registry=None,
    metaclass=DeclarativeMeta,
):
    # 创建metadata
    lcl_metadata = metadata or MetaData()

    if class_registry is None:
        class_registry = weakref.WeakValueDictionary()

    bases = not isinstance(cls, tuple) and (cls,) or cls
    class_dict = dict(
        _decl_class_registry=class_registry, metadata=lcl_metadata
    )

    # 构造函数
    if constructor:
        class_dict["__init__"] = constructor
    if mapper:
        class_dict["__mapper_cls__"] = mapper
    
    # class-meta
    return metaclass(name, bases, class_dict)

"""Model的另外一个功能是隐式创建Table对象，在_as_declarative函数中通过_MapperConfig实现"""
class _MapperConfig(object):
    def setup_mapping(cls, cls_, classname, dict_):
        cfg_cls = _MapperConfig
        cfg_cls(cls_, classname, dict_)
    
    def __init__(self, cls_, classname, dict_):
        ...
        self._setup_table()
        ...
    
    def _setup_table(self):
        ...
        table_cls = Table
        args, table_kw = (), {}
        if table_args:
            if isinstance(table_args, dict):
                table_kw = table_args
            elif isinstance(table_args, tuple):
                if isinstance(table_args[-1], dict):
                    args, table_kw = table_args[0:-1], table_args[-1]
                else:
                    args = table_args

        autoload = dict_.get("__autoload__")
        if autoload:
            table_kw["autoload"] = True

        cls.__table__ = table = table_cls(
            tablename,
            cls.metadata,
            *(tuple(declared_columns) + tuple(args)),
            **table_kw
        )
        ...

"""而Column是通过下面的函数实现"""
def _add_attribute(cls, key, value):

    if "__mapper__" in cls.__dict__:
        if isinstance(value, Column):
            _undefer_column_name(key, value)
            cls.__table__.append_column(value)
            cls.__mapper__.add_property(key, value)
        ...
    else:
        type.__setattr__(cls, key, value)
```