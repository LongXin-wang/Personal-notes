- [Mock](#mock)
- [MagicMock](#magicmock)
- [Patch](#patch)
- [MonkeyPatch](#monkeypatch)
- [mock原理](#mock原理)

# Mock

测试和关注的是自身组件代码的稳定性，而非外界依赖\服务的交互（这块的测试放到了集成测试的部分），也因此产生了伪造调用依赖/服务的需求。

通过mock，我们可以假装调用了xxx服务，或者xxx sdk的方法，甚至于可以借此测试一些复杂的场景，比如重试部分的代码是否符合预期等等。

单元测试的条件有限，在测试过程中，有时会遇到难以准备的环境。 比如，与服务器的网络交互、对数据库的读写等。****

传统思路，是利用fixture进行测试环境准备。 这种做法的优点是，与真实环境非常相似，测试效果好。 但缺点是，测试代码开发时间长，测试执行时间也很长。****

另一种思路是，准备一个虚假的沙箱，对代码的执行效果进行模拟。 这样虽然不能测试真正的最终效果，但是却更容易保证100%测试覆盖率，并且避免重复测试，降低测试执行时间。

```python
from unittest.mock import MagicMock, patch
```

# MagicMock

```python
# a simple module with a simple class
class AClass(object):
    def method(self, an_argument):
        print "I typed {}".format(an_argument)

>>> from unittest.mock import MagicMock
>>> import amodule
>>> thing = amodule.AClass()

# method方法替换成MagicMock实例
# 这里传入参数指定了return_value，因此调用MagicMock伪造的method方法，将会返回3！ ，MagicMock即替代了thing的method方法，它会假装接受传入的参数，然后返回值设置为3
>>> thing.method = MagicMock(return_value=3)
>>> print thing.method('an_arg')
3

# assert_called_with 即验证，我们最近一次调用method方法，是传入了'others'来调用的
# 但是，显然不是:( 因此这行会报错
>>> print thing.method.assert_called_with('others') # ssert_called_with方法，那么这个方法的作用是什么呢？正如注释里面所说的，它会去验证最近一次对mock代替的方法的调用，传入的参数是预期的那样，否则会报错AssertionError。
...
AssertionError: Expected call: mock('others')
Actual call: mock('an_arg')
```

# Patch


```python
from unittest import mock
import requests
import json
import unittest


def get_request():
    req = requests.get("http://www.baidu.com")
    # do sth...
    return req.status_code == 200


class TestClient(unittest.TestCase):
    @mock.patch('requests.get')
    def test_get_request(self, mock_request):
        mock_request.return_value = mock.MagicMock(
            status_code=200, response=json.dumps({'key': 'value'})
        )
        self.assertEqual(get_request(), True)


if __name__ == '__main__':
    unittest.main()
```

```python
# coding=utf-8

from unittest import mock
import unittest
import retrying
import requests
import json


class RequestError(Exception):
    def __init__(self, server):
        self.message = "Request failed to {}.".format(server)


class Blog(object):
    '''
    Blog类包含一些博客文章相关的操作封装
    类在初始化后，post方法提供对指定文章链接内容的获取并返回JSON的格式
    '''
    def __init__(self, name):
        self.name = name

    @retrying.retry(stop_max_attempt_number=3, wait_fixed=100)
    def post(self, article_path):
        '''
        根据获取文章的内容
        '''
        response = requests.get("http://{name}/{path}".format(
            name=self.name,
            path=article_path))
        if response.status_code >= 200 and response.status_code < 300:
            return response.text
        else:
            raise RequestError(self.name)

    def __repr__(self):
        return '<Blog: {}>'.format(self.name)


class TestBlog(unittest.TestCase):
    '''
    我们需要测试哪些内容？根据K大的http://docs.python-guide.org/en/latest/writing/tests/
    我们关注本身的功能的覆盖，那么主要需要测试：
    1. 正常请求时是否能获得正确的JSON数据
    2. 异常返回时是否抛出RequestError异常
    3. 遇到错误是否会重试
    '''
    @mock.patch('requests.get')
    def test_blog_post(self, mock_get):
        mocked_content = json.dumps({
            'status': True, 'content': 'This is the post content'
        })
        mocked_success = mock.MagicMock(
                status_code=200,
                headers={'content-type': "application/json"},
                text=mocked_content)
        mock_get.return_value = mocked_success

        # new blog instance and call `post`.
        # confirm return value on succcess.
        b = Blog(name="devopstarter.info")
        content = b.post("three-years-later-test-mock")
        self.assertEqual(content, mocked_content)

        mock_get.reset_mock()  # reset mock, include called count etc.
        mocked_error = mock.MagicMock(
                status_code=500,
                headers={'content-type': "application/json"},
                text="Internal server error")
        # confirm raise exception on failure.
        # due to mock object return_value is always error, retry is also error.
        mock_get.return_value = mocked_error
        with self.assertRaises(RequestError):
            b.post("three-years-later-test-mock")
        # confirm the retry works in failure case, call_count exceeded 3!
        self.assertEqual(mock_get.call_count, 3)

        # confirm the retry works while ONLY one time failed.
        # `side_effect` helped us complete this goal!
        mock_get.reset_mock()  # reset mock, include called count etc.
        mock_get.side_effect = [    # side_effect属性来指定mock对象的行为。在这个例子中，我们指定了两个不同的行为：第一次调用mock_get时，它会引发一个requests.ConnectionError异常；第二次调用mock_get时，它会返回一个名为mocked_success的mock响应对象
            requests.ConnectionError('Test error'),
            mocked_success
        ]
        content = b.post("three-years-later-test-mock")
        self.assertEqual(content, mocked_content)


if __name__ == '__main__':
    unittest.main()
```

# MonkeyPatch

monkey patch 指的是对于一个类或者模块所进行的动态修改。在Python语言中，我们其实可以在运行时修改代码的行为。

```ptyhon

#mock一些装饰器 用装饰器_auth_login mock掉装饰器auth_login
@pytest.fixture(autouse=True)
def mock_auth_login(monkeypatch):
    from conference.common import auth_decorator
 
    def _auth_login(proxy_valid=False, user_args=False):
        def decorate(func):
            @functools.wraps(func)
            def wrapper(*args, **kwargs):
                g.login_account = 'test_user'
                if user_args:
                   kwargs['user'] = User.filter([User.id == 1]).first()
                return func(*args, **kwargs)
            return wrapper
        return decorate
 
    monkeypatch.setattr(
        auth_decorator,
        'auth_login',
        _auth_login
        )

```

# mock原理

mockey pacthing mock掉函数、装饰器、模块等是将sys.moudule中加载的函数、装饰器等进行mock，其实就是指向新的地址，

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405171124857.png)