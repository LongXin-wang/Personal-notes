- [fixture](#fixture)
  - [fixture的scope](#fixture的scope)
    - [fixture直接调用](#fixture直接调用)
    - [fixture 实现前后置工作](#fixture-实现前后置工作)
    - [fixture的addfinalizer](#fixture的addfinalizer)
  - [fixture 执行顺序](#fixture-执行顺序)
    - [单独fixture情况（在各个py文件的）](#单独fixture情况在各个py文件的)
    - [fixture+conftest情况](#fixtureconftest情况)

# fixture

fixture主要的目的是为了提供一种可靠和可重复性的手段去运行那些最基本的测试内容，可以理解为一种资源。比如在测试网站的功能时，每个测试用例都要登录和退出，利用fixture就可以只做一次，否则每个测试用例都要做这两步也是冗余。

fixture是pytest里的一个特性，目的是对上面传统的setup/teardown函数的改进，它有以下几个特点：

- 每个fixture都拥有姓名，可以被显示调用（setup/teardown函数都是隐式调用）
- fixture本质是一个函数，fixture之间可以互相调用
- fixture可以在函数、方法、模块间复用（在哪里调用都可以）
- fixture的主要工作也是准备数据、设置环境、清理等，但不局限于此，fixtures也可以用来执行（Act）

```python
def fixture(scope="function", params=None, autouse=False, ids=None, name=None)
```

## fixture的scope

- function，表示 fixture 函数在测试方法执行前和执行后执行一次。
- class，表示 fixture 函数在测试类执行前和执行后执行一次。
- module，表示 fixture 函数在测试脚本执行前和执行后执行一次。
- package，表示 fixture 函数在测试包（文件夹）中第一个测试用例执行前和最后一个测试用例执行后执行一次。
- session，表示所有测试的最开始和测试结束后执行一次。

在使用fixture的scope时，需要注意以下问题：

- 当出现多个范围装饰的时候，优先实例化范围优先级高的。
- 作用域大的不可依赖作用域小的，否则报错ScopeMismatch。
- fixture如果存在依赖项，那么它所依赖的fixture函数会先被执行；若该fixture存在依赖项，则对于调用了- - fixture的测试函数来说，这些依赖项也可以看做是自动使用的
默认时function级别


### fixture直接调用

```python

#test_fix1.py
import pytest

@pytest.fixture(scope="function")
def go(request):
    print("\n----开始执行function----")
     
def test_1(go):
    print("---用例1执行---")
     
class Test_aaa():
    def test_2(self,go):
        print("-----用例2----")
    def test_3(self,go):
        print("---用例3---")

if __name__=="__main__":
    pytest.main(["-s","test_fix1.py"])
     
     
 test_001.py
----开始执行function----
.---用例1执行---
 
----开始执行function----
.-----用例2----
 
----开始执行function----
.---用例3---
```

### fixture 实现前后置工作

```python
@pytest.fixture
def mail_admin():
    return MailAdminClient()
 
@pytest.fixture
def sending_user(mail_admin):
    # Arrange
    user = mail_admin.create_user()
    yield user
    # Cleanup
    mail_admin.delete_user(user)
 
@pytest.fixture
def receiving_user(mail_admin):
    # Arrange
    user = mail_admin.create_user()
    yield user
    # Cleanup
    mail_admin.delete_user(user)
 
def test_email_received(receiving_user, sending_user):
    email = Email(subject="Hey!", body="How's it going?")
    sending_user.send_email(email, receiving_user)
    assert email in receiving_user.inbox
     
     
pytest的执行过程如下：可以看到，前置和后置动作都在test执行前后得以执行
admin = main_admin()
send_user = admin.create_user()    # in sending_user
receive_user = admin.create_user() # in receiving_user
test_email_received(receiving_user=receive_user,
                    sending_user=send_user)
admin.delete_user(receive_user)     # in receiving_user
admin.delete_user(send_user)
```

### fixture的addfinalizer

```python
@pytest.fixture()
def demo_fixture(request):
    print("\n这个fixture在每个case前执行一次")
    def demo_finalizer():
        print("\n在每个case完成后执行的teardown")
 
    #注册demo_finalizer为终结函数   
    request.addfinalizer(demo_finalizer)
 
def test_01(demo_fixture):
    print("\n===执行了case: test_01===")
 
def test_02(demo_fixture):
    print("\n===执行了case: test_02===")
 
执行结果：
test_module.py
这个fixture在每个case前执行一次
.
===执行了case: test_01===
 
在每个case完成后执行的teardown
 
这个fixture在每个case前执行一次
.
===执行了case: test_02===
 
在每个case完成后执行的teardown
```

## fixture 执行顺序

经过上面的介绍，我们了解到了fixture可以在各个单测用例的py文件中，也可以被统一放置到conftest中，具体的放置则根据实际情况来定，一般而言，共用的fixture和scope为session的fixture放置在conftest中

### 单独fixture情况（在各个py文件的）
没有autouse=True， 按照scope顺序执行，有依赖的先执行依赖
autouse = True，fixture 不用主动调用，它会在相应的作用域中自动执行，相同scope下autouse=True先执行
例如，3个fixture带autouse=True, 2个test用例，那么若fixture是function级别，则，每个test执行前执行一次，若是不带autouse，则调用的时候执行

### fixture+conftest情况
pytest用例自动去conftest下查找，不需要在其他py中导入conftest，直接用你想用的fixture即可
先按照scope优先级执行，scope相同的情况下，autouse=True先执行；

例如：conftest有一个module的，不是autouse的，每个py的test用例执行之前会先执行，并缓存，这个py文件则只执行一次，若是function的就每个test用例之前执行，执行多次

例如：conftest中有一个session的fixture，一个function的fixture，然后另外一个py文件有3个test，那么此时，session级别的执行一次，function级别的执行三次(前提是设置了autouse=true)

又比如：conftest中有一个session级别和一个module级别的fixture，然后另外两个py文件中有3个test，那么session的执行一次，module的执行两次(前提是设置了autouse=true)