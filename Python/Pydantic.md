- [创建数据模型](#创建数据模型)
- [数据校验](#数据校验)
  - [模型字段定义校验](#模型字段定义校验)
  - [自定义校验函数方法](#自定义校验函数方法)
    - [执行循序](#执行循序)
- [收集错误信息](#收集错误信息)
- [数据导出](#数据导出)

# 创建数据模型

* 继承 BaseModel 【常见】

```python
from datetime import datetime
from typing import List, Optional
from pydantic import BaseModel


class User(BaseModel):
    id: int
    name = 'John Doe'
    signup_ts: Optional[datetime] = None
    friends: List[int] = []


external_data = {
    'id': '123',
    'name': '222',
    'signup_ts': '2019-06-01 12:22',
    'friends': [1, 2, '3'],
}
user = User(**external_data)
print(user.id)

print(repr(user.signup_ts))

print(user.friends)

print(user.dict())
123
datetime.datetime(2019, 6, 1, 12, 22)
[1, 2, 3]
{'id': 123, 'signup_ts': datetime.datetime(2019, 6, 1, 12, 22), 'friends': [1, 2, 3], 'name': '222'}
```

* 通过工厂函数 create_model、create_model_from_namedtuple、create_model_from_typeddict 动态生成

```python
from datetime import datetime
from typing import List, Optional
from pydantic import create_model

DynamicUserModel = create_model(
    'DynamicUserModel', 
    id=(int, ...), 
    name='John Doe',
    signup_ts=(Optional[datetime], None),
    friends=(List[int], [])
)

external_data = {
    'id': '123',
    'name': '222',
    'signup_ts': '2019-06-01 12:22',
    'friends': [1, 2, '3'],
}


user = DynamicUserModel(**external_data)
print(user.id)

print(repr(user.signup_ts))

print(user.friends)

print(user.dict())
123
datetime.datetime(2019, 6, 1, 12, 22)
[1, 2, 3]
{'id': 123, 'signup_ts': datetime.datetime(2019, 6, 1, 12, 22), 'friends': [1, 2, 3], 'name': '222'}
```

* dataclasses 装饰器【用于取代python内置的dataclasses】
  生成的实例对象，并没有继承完整的 BaseModel 方法，有一部分部分接口是用不了的

```python
from datetime import datetime
from pydantic.dataclasses import dataclass


@dataclass
class User:
    id: int
    name = 'John Doe'
    signup_ts: Optional[datetime] = None

external_data = {
    'id': '123',
    'signup_ts': '2019-06-01 12:22',
}


user = User(**external_data)
print(user.id)

print(repr(user.signup_ts))

print(user.dict())
123
datetime.datetime(2019, 6, 1, 12, 22)
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
/root/project/flask_demo/pydantic_demo.ipynb Cell 5 in <cell line: 22>()
     <a href='vscode-notebook-cell://wsl%2Bubuntu-20.04/root/project/flask_demo/pydantic_demo.ipynb#W4sdnNjb2RlLXJlbW90ZQ%3D%3D?line=17'>18</a> print(user.id)
     <a href='vscode-notebook-cell://wsl%2Bubuntu-20.04/root/project/flask_demo/pydantic_demo.ipynb#W4sdnNjb2RlLXJlbW90ZQ%3D%3D?line=19'>20</a> print(repr(user.signup_ts))
---> <a href='vscode-notebook-cell://wsl%2Bubuntu-20.04/root/project/flask_demo/pydantic_demo.ipynb#W4sdnNjb2RlLXJlbW90ZQ%3D%3D?line=21'>22</a> print(user.dict())

AttributeError: 'User' object has no attribute 'dict'
```

# 数据校验

## 模型字段定义校验
- typing 校验
- con* 系校验
- Pydantic 内置的校验字段
- 自定义校验字段

## 自定义校验函数方法
- validator
- root_validator

### 执行循序

root_validator(pre=True)
validator(, pre=True) -> Field_check ->validator() #按定义时候的顺序执行
validator(, pre=True) -> Field_check ->validator()
....
root_validator()

```python
class BaseModel(Representation, metaclass=ModelMetaclass):
    def __init__(__pydantic_self__, **data: Any) -> None:
        """
        Create a new model by parsing and validating input data from keyword arguments.

        Raises ValidationError if the input data cannot be parsed to form a valid model.
        """
        # Uses something other than `self` the first arg to allow "self" as a settable attribute
        values, fields_set, validation_error = validate_model(__pydantic_self__.__class__, data)
        if validation_error:
            raise validation_error
        object_setattr(__pydantic_self__, '__dict__', values)
        object_setattr(__pydantic_self__, '__fields_set__', fields_set)
        __pydantic_self__._init_private_attributes()


def validate_model(  # noqa: C901 (ignore complexity)
    model: Type[BaseModel], input_data: 'DictStrAny', cls: 'ModelOrDc' = None
) -> Tuple['DictStrAny', 'SetStr', Optional[ValidationError]]:
    """
    validate data against a model.
    """
    # 初始化了values和errors 容器
    values = {}
    errors = []
    # input_data names, possibly alias
    names_used = set()
    # field names, never aliases
    fields_set = set()
    config = model.__config__
    check_extra = config.extra is not Extra.ignore
    cls_ = cls or model

    # 先执行了 __pre_root_validators__
    for validator in model.__pre_root_validators__:
        try:
            input_data = validator(cls_, input_data)
        except (ValueError, TypeError, AssertionError) as exc:
            return {}, set(), ValidationError([ErrorWrapper(exc, loc=ROOT_KEY)], cls_)
    # 然后安装 model.__fields__ 顺序进行执行
    for name, field in model.__fields__.items():
        if field.type_.__class__ == ForwardRef:
            raise ConfigError(
                f'field "{field.name}" not yet prepared so type is still a ForwardRef, '
                f'you might need to call {cls_.__name__}.update_forward_refs().'
            )
        # 支持字段定义别名
        value = input_data.get(field.alias, _missing)
        using_name = False
        if value is _missing and config.allow_population_by_field_name and field.alt_alias:
            value = input_data.get(field.name, _missing)
            using_name = True
        # 空值值字段处理逻辑
        if value is _missing:
            # 必填字段
            if field.required:
                errors.append(ErrorWrapper(MissingError(), loc=field.alias))
                continue
            # 调用默认值工厂函数
            value = field.get_default()
            if not config.validate_all and not field.validate_always:
                values[name] = value
                continue
        else:
            fields_set.add(name)
            if check_extra:
                names_used.add(field.name if using_name else field.alias)
        # 执行字段的校验，并收集异常
        v_, errors_ = field.validate(value, values, loc=field.alias, cls=cls_)
        if isinstance(errors_, ErrorWrapper):
            errors.append(errors_)
        elif isinstance(errors_, list):
            errors.extend(errors_)
        else:
            values[name] = v_
    # 处理传入的多余字段
    if check_extra:
        if isinstance(input_data, GetterDict):
            extra = input_data.extra_keys() - names_used
        else:
            extra = input_data.keys() - names_used
        if extra:
            fields_set |= extra
            if config.extra is Extra.allow:
                for f in extra:
                    values[f] = input_data[f]
            else:
                for f in sorted(extra):
                    errors.append(ErrorWrapper(ExtraError(), loc=f))
    # 输出校验 root_validators 
    for skip_on_failure, validator in model.__post_root_validators__:
        if skip_on_failure and errors:
            continue
        try:
            values = validator(cls_, values)
        except (ValueError, TypeError, AssertionError) as exc:
            errors.append(ErrorWrapper(exc, loc=ROOT_KEY))

    if errors:
        return values, fields_set, ValidationError(errors, cls_)
    else:
        return values, fields_set, None


class ModelField(Representation):
    
    def validate(
        self, v: Any, values: Dict[str, Any], *, loc: 'LocStr', cls: Optional['ModelOrDc'] = None
    ) -> 'ValidateReturn':

        errors: Optional['ErrorList']
        if self.pre_validators:
            v, errors = self._apply_validators(v, values, loc, cls, self.pre_validators)
            if errors:
                return v, errors

        if v is None:
            if self.allow_none:
                if self.post_validators:
                    return self._apply_validators(v, values, loc, cls, self.post_validators)
                else:
                    return None, None
            else:
                return v, ErrorWrapper(NoneIsNotAllowedError(), loc)
```

```python
from pydantic import BaseModel,ValidationError, validator, constr, root_validator


class MyField:
    
    @classmethod
    def __get_validators__(cls):
        print('run get_validators')
        yield cls.validate
        
    @classmethod
    def validate(cls, v):
        print('run field validate')
        return v

print('new a UserModel')

class UserModel(BaseModel):
    name: constr(strip_whitespace=True)
    username: MyField
    password1: str
    password2: str

    @root_validator(pre=True)
    def check_input(cls, values):
        print(f'run int root_validator pre', values)
        values['password4'] = '444'
        return values
    
    @root_validator()
    def check_output(cls, values):
        print(f'run int root_validator', values)
        return values

    @validator('name')
    def name_must_contain_space(cls, v):
        if ' ' not in v:
            raise ValueError('must contain a space')
        return v.title()

    @validator('password2')
    def passwords_match(cls, v, values, **kwargs):
        print("passwords_values", values)
        print("kwargs", kwargs)
        if 'password1' in values and v != values['password1']:
            raise ValueError('passwords do not match')
        return v

    @validator('username')
    def username_(cls, v, values, **kwargs):
        print("username_values", values)
        print("-name-" * 10)
        assert v.isalnum(), 'must be alphanumeric'
        return v
    
    @validator('username', pre=True)
    def username_pre(cls, v):
        print("-name-" * 10)
        print("username_pre",)
        return v

print(f'{"**" * 10} ')

user = UserModel(
    name='samuel colvin',
    username='scolvin',
    password1='zxcvbn',
    password2='zxcvbn',
    password3='zxcvbn',
)

print('user info:', user)

print(f'error:\n {"*****" * 10}')
new a UserModel
run get_validators
******************** 
run int root_validator pre {'name': 'samuel colvin', 'username': 'scolvin', 'password1': 'zxcvbn', 'password2': 'zxcvbn', 'password3': 'zxcvbn'}
-name--name--name--name--name--name--name--name--name--name-
username_pre
run field validate
username_values {'name': 'Samuel Colvin'}
-name--name--name--name--name--name--name--name--name--name-
passwords_values {'name': 'Samuel Colvin', 'username': 'scolvin', 'password1': 'zxcvbn'}
kwargs {'field': ModelField(name='password2', type=str, required=True), 'config': <class '__main__.Config'>}
run int root_validator {'name': 'Samuel Colvin', 'username': 'scolvin', 'password1': 'zxcvbn', 'password2': 'zxcvbn'}
user info: name='Samuel Colvin' username='scolvin' password1='zxcvbn' password2='zxcvbn'
error:
 **************************************************
```

# 收集错误信息
- loc:错误的位置列表。列表中的第一项将是发生错误的字段，如果该字段是子模型，则将出现后续项以指示错误的嵌套位置。
- type:计算机可读的错误类型的标识符。
- msg:人类可读的错误解释。
- ctx:包含呈现错误消息所需的值的可选对象。【error_msg_templates 模板中可以调用】

```python
from typing import List
from pydantic import BaseModel, ValidationError, conint, Field


class Location(BaseModel):
    lat = 0.1
    lng = 10.1


class Model(BaseModel):
    class Config:
        error_msg_templates = {
            "value_error.missing": "字段值缺失",
            "value_error.number.not_gt": '字段值小于阈值{limit_value}'
        }
        
    is_required: float = Field(error_name='必传字段')
    gt_int: conint(gt=42)
    list_of_ints: List[int] = Field(title='整数列表')
    a_float: float = None
    recursive_model: Location = None


data = dict(
    list_of_ints=['1', 2, 'bad'],
    a_float='not a float',
    recursive_model={'lat': 4.2, 'lng': 'New York'},
    gt_int=21,
)

try:
    Model(**data)
except ValidationError as e:
    print(e)

5 validation errors for Model
is_required
  字段值缺失 (type=value_error.missing)
gt_int
  字段值小于阈值42 (type=value_error.number.not_gt; limit_value=42)
list_of_ints -> 2
  value is not a valid integer (type=type_error.integer)
a_float
  value is not a valid float (type=type_error.float)
recursive_model -> lng
  value is not a valid float (type=type_error.float)

try:
    Model(**data)
except ValidationError as e:
    print(e.json())

[
  {
    "loc": [
      "is_required"
    ],
    "msg": "\u5b57\u6bb5\u503c\u7f3a\u5931",
    "type": "value_error.missing"
  },
  {
    "loc": [
      "gt_int"
    ],
    "msg": "\u5b57\u6bb5\u503c\u5c0f\u4e8e\u9608\u503c42",
    "type": "value_error.number.not_gt",
    "ctx": {
      "limit_value": 42
    }
  },
  {
    "loc": [
      "list_of_ints",
      2
    ],
    "msg": "value is not a valid integer",
    "type": "type_error.integer"
  },
  {
    "loc": [
      "a_float"
    ],
    "msg": "value is not a valid float",
    "type": "type_error.float"
  },
  {
    "loc": [
      "recursive_model",
      "lng"
    ],
    "msg": "value is not a valid float",
    "type": "type_error.float"
  }
]
```

# 数据导出

model.dict(...)
model.json(...)

```python
from typing import List, Optional, Any
from pydantic import BaseModel


class BarModel(BaseModel):
    whatever: int


class FooBarModel(BaseModel):
    banana: Any = 1
    foo: str
    bar: BarModel
    f1: Optional[int] = 0
    f2: Optional[float] = 2
    f3: Optional[float]


m = FooBarModel(banana=None, foo='hello', bar={'whatever': 123}, f2=2.1)

# returns a dictionary:
print(m.dict())
print('include', m.dict(include={'foo', 'bar'}))
print('exclude', m.dict(exclude={'foo', 'bar'}))
print('exclude_unset', m.dict(exclude_unset=True))
# 注意如果在没有显示设置的话此时默认值None，这时如果输入了None，会导致字段缺失，这种不在预料中的情况。
print('exclude_defaults', m.dict(exclude_defaults=True))
print('exclude_none', m.dict(exclude_none=True))

{'banana': None, 'foo': 'hello', 'bar': {'whatever': 123}, 'f1': 0, 'f2': 2.1, 'f3': None}
include {'foo': 'hello', 'bar': {'whatever': 123}}
exclude {'banana': None, 'f1': 0, 'f2': 2.1, 'f3': None}
exclude_unset {'banana': None, 'foo': 'hello', 'bar': {'whatever': 123}, 'f2': 2.1}
exclude_defaults {'banana': None, 'foo': 'hello', 'bar': {'whatever': 123}, 'f2': 2.1}
exclude_none {'foo': 'hello', 'bar': {'whatever': 123}, 'f1': 0, 'f2': 2.1}
```