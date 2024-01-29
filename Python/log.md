- [日志级别](#日志级别)
- [logging和print的区别](#logging和print的区别)
- [logging 提供的函数处理](#logging-提供的函数处理)
  - [输出到文件](#输出到文件)
  - [输出到控制台](#输出到控制台)
- [logging 提供的四大组件](#logging-提供的四大组件)
  - [py文件形式](#py文件形式)
    - [文件大小 \& 事件间隔 滚动](#文件大小--事件间隔-滚动)
  - [配置文件形式](#配置文件形式)
- [异常详解](#异常详解)
  - [LogRecord](#logrecord)
  - [logger 打印异常](#logger-打印异常)

# 日志级别

日志功能应以所追踪事件级别或严重性而定。各级别适用性如下（以严重性递增）：

| 级别       | 何时使用                                                             |
| ---------- | -------------------------------------------------------------------- |
| `DEBUG`    | 细节信息，仅当诊断问题时适用。                                       |
| `INFO`     | 确认程序按预期运行。                                                 |
| `WARNING`  | 表明有已经或即将发生的意外（例如：磁盘空间不足）。程序仍按预期进行。 |
| `ERROR`    | 由于严重的问题，程序的某些功能已经不能正常执行                       |
| `CRITICAL` | 严重的错误，表明程序已不能继续执行                                   |


默认的级别是 `WARNING`，意味着只会追踪该级别及以上的事件，除非更改日志配置。

所追踪事件可以以不同形式处理。最简单的方式是输出到控制台。另一种常用的方式是写入磁盘文件。

# logging和print的区别

print ：会输出到标准输出流中，格式为字符串格式

logging 模块：更加灵活

- 可以设置输出到任意位置，如写入文件、写入远程服务器等

- 设置日志等级，在不同的版本(如开发环境、生产环境)上通过设置不同的输出等级来记录对应的日志

- 具有灵活的配置和格式化功能，如配置输出当前模块信息、运行时间等，相比 print 的字符串格式化更加方便易用。可以在 logging 模块中非常灵活。

用`sys.stdout`（控制台）将`print`行导向到你定义的日志文件中

```Python
import sys

# 先把标准输出保存起来
stdout_backup = sys.stdout
# define the log file that receives your log info
log_file = open("message.log", "w")
# redirect print output to log file
sys.stdout = log_file

# 输出到文件了，因为这改变了stdout
print("Now all print info will be written to message.log")

log_file.close()
# 再把标准输出恢复
sys.stdout = stdout_backup

print("Now this will be presented on screen")


'''直接输出到文件'''
# 打开文件来写入输出
with open('output.txt', 'w') as f:
    print('Hello, this will be written to the file', file=f)

```

> print 伪代码

```C
# 打开文件来写入输出
void PyPrint(PyObject *object, FILE *file, int end, int flush) {
    // 将对象转换为字符串
    PyObject *string = PyObject_Str(object);
    if (string == NULL) {
        // 处理转换失败的情况
        PyErr_SetString(PyExc_RuntimeError, "Error converting object to string");
        return;
    }

    // 将字符串打印到指定文件流 指定了输出到哪
    if (file == NULL) {
        file = PySys_GetObject("stdout");
        if (file == NULL) {
            PyErr_SetString(PyExc_RuntimeError, "Error getting sys.stdout");
            return;
        }
    }
    fprintf(file, "%s", PyUnicode_AsUTF8(string));

    // 处理结尾和刷新逻辑
    if (end) {
        fprintf(file, "\n");
    }
    if (flush) {
        fflush(file);
    }
}
```

# logging 提供的函数处理

## 输出到文件

```python
import logging

# asctime filename lineno levelname 等从Python的执行环境中动态获取的，logging模块会在记录日志时自动填充这些信息
logging.basicConfig(level=logging.WARNING,
                    filename='./log.txt',
                    filemode='w',
                    format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')
# use logging
logging.info('这是 loggging info message')
logging.debug('这是 loggging debug message')
logging.warning('这是 loggging a warning message')
logging.error('这是 an loggging error message')
logging.critical('这是 loggging critical message')
```

## 输出到控制台

```python
import logging
logging.basicConfig(level=logging.WARNING,
                    format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')

# 开始使用log功能
logging.info('这是 loggging info message')
logging.debug('这是 loggging debug message')
logging.warning('这是 loggging a warning message')
logging.error('这是 an loggging error message')
logging.critical('这是 loggging critical message')

2024-04-03 14:48:17,005 - test.py[line:8] - WARNING: 这是 loggging a warning message
2024-04-03 14:48:17,005 - test.py[line:9] - ERROR: 这是 an loggging error message
2024-04-03 14:48:17,005 - test.py[line:10] - CRITICAL: 这是 loggging critical message
```

# logging 提供的四大组件

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202404031609610.png)

## py文件形式

**同时输出到文件和stdout**

```python
import logging

# 创建一个logger
logger = logging.getLogger('my_logger')
logger.setLevel(logging.DEBUG)

# 创建一个文件处理器并设置级别为DEBUG
file_handler = logging.FileHandler('example.log')
file_handler.setLevel(logging.DEBUG)

# 创建一个控制台处理器并设置级别为DEBUG
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.DEBUG)

# 创建一个格式化器
formatter = logging.Formatter('%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')

# 为处理器添加格式化器
file_handler.setFormatter(formatter)
console_handler.setFormatter(formatter)

# 将处理器添加到logger
logger.addHandler(file_handler)
logger.addHandler(console_handler)

# 开始使用log功能
logger.info('这是 loggging info message')
logger.debug('这是 loggging debug message')
logger.warning('这是 loggging a warning message')
logger.error('这是 an loggging error message')
logger.critical('这是 loggging critical message')
```

### 文件大小 & 事件间隔 滚动

```python
# 创建RotatingFileHandler对象，设置日志文件名、文件大小和保留文件数量
# 系统将通过为原文件名添加扩展名 '.1', '.2' 等来保存旧日志文件。 例如，当 backupCount 为 5 而基本文件名为 app.log 时，你将得到 app.log, app.log.1, app.log.2 直至 app.log.5
handler = RotatingFileHandler(filename='my_log.log', maxBytes=200*1024*1024, backupCount=7)

# 创建TimedRotatingFileHandler对象，设置日志文件名、时间间隔和保留文件数量
# 每天滚动1次日志 就是7天保留了7个，第8天就开始滚动掉第一天的了
handler = TimedRotatingFileHandler(filename='my_log.log', when='D', interval=1, backupCount=7)
# 切分后日志文件名的时间格式 默认 filename+"." + suffix 如果需要更改需要改logging 源码
handler.suffix = "%Y-%m-%d.log"  # 不配置默认用"%Y%m%d%H%M%S.log

```

## 配置文件形式

```python
[loggers]   # 定义了应用程序将使用的日志器。在这个例子中，定义了一个日志器：root
keys=root

[handlers]  # 定义了日志处理器，它们决定如何处理（例如，显示在控制台，写入文件等）日志消息。在这个例子中，定义了两个处理器：fileHandler和consoleHandler
keys=fileHandler,consoleHandler

[formatters]    # 定义了格式化器，它们决定如何格式化日志消息。在这个例子中，定义了一个格式化器：simpleFormatter
keys=simpleFormatter

[logger_root]   # 定义了root日志器的配置 它设置了日志级别为DEBUG，并使用fileHandler和consoleHandler处理器处理日志消息
level=DEBUG
handlers=fileHandler,consoleHandler

[handler_consoleHandler]    # 定义了consoleHandler处理器的配置。它的类设置为StreamHandler，级别设置为DEBUG，格式化器设置为simpleFormatter，参数设置为(sys.stdout,)，表示将日志消息输出到标准输出
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)

[handler_fileHandler]   # 定义了fileHandler处理器的配置。它的类设置为FileHandler，级别设置为DEBUG，格式化器设置为simpleFormatter，参数设置为('example.log',)，表示将日志消息写入到名为example.log的文件中
class=FileHandler
level=DEBUG
formatter=simpleFormatter
args=('example.log',)

[formatter_simpleFormatter] # 定义了格式
format=%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s

import logging
import logging.config

logging.config.fileConfig('logging.conf')

# 开始使用log功能
logger = logging.getLogger('root')
logger.info('这是 loggging info message')
logger.debug('这是 loggging debug message')
logger.warning('这是 loggging a warning message')
logger.error('这是 an loggging error message')
logger.critical('这是 loggging critical message')

2024-04-03 15:31:01,464 - test.py[line:8] - INFO: 这是 loggging info message
2024-04-03 15:31:01,464 - test.py[line:9] - DEBUG: 这是 loggging debug message
2024-04-03 15:31:01,465 - test.py[line:10] - WARNING: 这是 loggging a warning message
2024-04-03 15:31:01,465 - test.py[line:11] - ERROR: 这是 an loggging error message
2024-04-03 15:31:01,465 - test.py[line:12] - CRITICAL: 这是 loggging critical message
```

# 异常详解

## LogRecord

**当你调用如logger.info('some message')这样的函数时，logging模块会创建一个LogRecord对象，然后将它传递给日志系统的各个部分**

LogRecord对象包含了关于日志事件的所有信息，如时间戳、日志级别、日志消息、源文件名、源文件行号等

```python
class LogRecord(object):

    def __init__(self, name, level, pathname, lineno,
                 msg, args, exc_info, func=None, sinfo=None, **kwargs):
        ct = time.time()
        self.name = name
        self.msg = msg
        if (args and len(args) == 1 and isinstance(args[0], collections.abc.Mapping)
            and args[0]):
            args = args[0]
        self.args = args
        self.levelname = getLevelName(level)
        self.levelno = level
        self.pathname = pathname
        try:
            self.filename = os.path.basename(pathname)
            self.module = os.path.splitext(self.filename)[0]
        except (TypeError, ValueError, AttributeError):
            self.filename = pathname
            self.module = "Unknown module"
        self.exc_info = exc_info
        self.exc_text = None      # used to cache the traceback text
        self.stack_info = sinfo
        self.lineno = lineno
        self.funcName = func
    

    def getMessage(self):
        """
        就是传入的信息
        """
        msg = str(self.msg)
        # self.msg是'Hello, %s!'，self.args是('World',)，那么getMessage方法返回的结果就是'Hello, World!'
        if self.args:
            msg = msg % self.args
        return msg

```

## logger 打印异常

```python
import logging

# 创建一个logger
logger = logging.getLogger('my_logger')
logger.setLevel(logging.DEBUG)

# 创建一个文件处理器并设置级别为DEBUG
file_handler = logging.FileHandler('example.log')
file_handler.setLevel(logging.DEBUG)

# 创建一个文件处理器并设置级别为DEBUG
file_handler = logging.FileHandler('example.log')
file_handler.setLevel(logging.DEBUG)

# 创建一个控制台处理器并设置级别为DEBUG
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.DEBUG)

# 创建一个格式化器
formatter = logging.Formatter('%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s')

# 为处理器添加格式化器
file_handler.setFormatter(formatter)
console_handler.setFormatter(formatter)

logger.addHandler(console_handler)


try:
  
    # 这里是你的代码，可能会抛出异常
    result = 1 / 0
except Exception as e:
   **走的是exc_info**
    # 在捕获异常后，使用logger.error()方法记录异常 
    logger.error("An error occurred: ", exc_info=True)  # 使用exc_info=True来包含异常信息

2024-04-03 17:02:12,658 - test.py[line:39] - ERROR: An error occurred: 
Traceback (most recent call last):
  File "/home/wanglongxin/youling/platsdk/platsdk/local_test/test.py", line 33, in <module>
    result = 1 / 0
ZeroDivisionError: division by zero


try:
    result = 1 / 0
except Exception as e:
    **走的是msg**
    # 在捕获异常后，使用logger.error()方法记录异常   
    err_msg=traceback.format_exc()
    logging.error(f"exception occurs when do math compute,traceback is:\n{err_msg}")

2024-04-03 17:01:07,768 - test.py[line:37] - ERROR: exception occurs when do math compute,traceback is:
Traceback (most recent call last):
  File "/home/wanglongxin/youling/platsdk/platsdk/local_test/test.py", line 33, in <module>
    result = 1 / 0
ZeroDivisionError: division by zero


try:
    result = 1 / 0
except Exception as e:
    **msg=e传入处理，输出是异常的描述，类似print(e)一样的原理**
    logger.error(e)  # division by zero 

2024-04-03 17:02:44,480 - test.py[line:38] - ERROR: division by zero
```

**上述异常打印不同的原因**

无论是.error还是info或者其他都是调用_log(),所以我们就分析这个

```python
def _log(self, level, msg, args, exc_info=None, extra=None, stack_info=False,
             stacklevel=1):
    '''
    一个变量sinfo并初始化为None。然后，它检查全局变量_srcfile是否存在。_srcfile是logging模块的一个内部变量，它表示logging模块的源文件名。如果_srcfile存在，那么它会尝试调用self.findCaller(stack_info, stacklevel)方法来找到调用者的堆栈帧，然后获取源文件名、行号、函数名和堆栈信息。如果在调用findCaller方法时抛出了ValueError异常，那么它会将源文件名、行号和函数名设置为未知。如果_srcfile不存在，那么它也会将源文件名、行号和函数名设置为未知
    '''
    sinfo = None
    if _srcfile:
            fn, lno, func, sinfo = self.findCaller(stack_info, stacklevel)
        except ValueError: # pragma: no cover
            fn, lno, func = "(unknown file)", 0, "(unknown function)"
    else: # pragma: no cover
        fn, lno, func = "(unknown file)", 0, "(unknown function)"
    """
    检查exc_info参数是否存在。exc_info参数是一个异常信息，它可以是一个异常实例，也可以是一个包含异常类型、异常值和追踪信息的元组。如果exc_info是一个异常实例，那么它会将exc_info转换为一个元组；如果exc_info不是一个元组，那么它会调用sys.exc_info()函数来获取当前线程的异常信息
    """
    if exc_info:
        if isinstance(exc_info, BaseException):
            exc_info = (type(exc_info), exc_info, exc_info.__traceback__)
        elif not isinstance(exc_info, tuple):
            'logger.error("An error occurred: ", exc_info=True)就是走了这里'
            exc_info = sys.exc_info()
    '''创建一个LogRecord'''
    record = self.makeRecord(self.name, level, fn, lno, msg, args,
                                exc_info, func, extra, sinfo)
    self.handle(record)
```