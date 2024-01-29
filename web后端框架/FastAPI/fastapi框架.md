- [FastAPI VS Flask](#fastapi-vs-flask)

# FastAPI VS Flask

**性能比较**

- flask是同步框架，fastapi是异步框架
- flask + gevent 处理请求的协程是异步的（其靠的是全局的monkey- patch实现异步库的替换），若是异步库不支持，则受限；而fastapi原生支持异步
- gevent的异步库像个盲盒，说实话，不知道哪些被替换了，但是fastapi却可以在开发端去控制

**使用便捷**

- 自动swagger，生成api文档
- 依赖注入：类似装饰器的玩法，很多校验、类型检查、出口入口都用它来标准化了，不需要写到函数具体实现当中，只需要在类型注解中

```python
@api_router.get(
    "/user/info",
    response_model=List[str],
)
def get_user(
    user_info: tuple = Depends(partial(auth_announcement_manage, is_raise=False))
):
    """
    获取当前登录用户的信息
    """
    return user_info[0]
```
