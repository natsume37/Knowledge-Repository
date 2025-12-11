# logging模块



## 一、日志级别

```python
import logging
logging.debug('调试日志')
logging.info('消息日志')
logging.warning('警告日志')
logging.error('错误日志')
logging.critical('严重错误日志')
```



## 二、日志配置

```python
# 日志基本配置
logging.basicConfig(
    # 1、日志级别
    level=30,
    # DEBUG:10
    # INFO:20
    # WARNING:30
    # ERROR:40
    # CRITICAL:50
    # 2、日志输出格式
    # format='%(asctime)s %(name)s [%(pathname)s line:%(lineno)d] %(levelname)s %(message)s',
    # 3、asctime的时间格式
    # datefmt='%Y-%m-%d %H:%M:%S',
    # 4、日志输出位置：终端/文件
    # filename='user.log', # 不指定此配置，默认打印到终端
)
'''
%(name)s
Logger的名字(getlogger时指定的名字)
%(levelno)s 数字形式的日志级别
%(levelname)s 文本形式的日志级别
%(pathname)s 调用日志输出日志的完整路径名
%(filename)s 调用日志输出日志的文件名
%(module)s 调用日志输出日志的模块名
%(funcName)s 调用日志输出日志的函数名
%(lineno)d 调用日志输出函数的语句所在的代码行
%(created)f 当前时间，用UNIX标准的表示时间的浮 点数表示
%(relativeCreated)d 输出日志信息时的，自Logger创建以 来的毫秒数
%(asctime)s 字符串形式的当前时间,默认格式是 “2022-07-30 22:15:53,394”
%(thread)d 线程ID,可能没有
%(threadName)s 线程名,可能没有
%(process)d 进程ID,可能没有
%(message)s 用户输出的消息
'''
'''
logging模块有三个比较重要的功能组件：
    1、loggers 配置文件可定义一些输出日志的appname
    2、handler 配置日志的分隔大小,输出位置,日志文件创建等
    3、formatters 配置日志输出的格式
'''
```





## 日志配置字典

```python
# 核心就在于CV
import logging
import logging.config
import os.path

# 路径配置
BASE_DIR = os.path.dirname(os.path.dirname(__file__))  # 项目根目录
INFO_LOG_DIR = os.path.join(BASE_DIR, "log", 'info.log')
ERROR_LOG_DIR = os.path.join(BASE_DIR, "log", 'error.log')

LOGGING_DIC = {
    'version': 1.0,
    'disable_existing_loggers': False,
    # 日志格式
    'formatters': {
        'standard': {
            'format': '%(asctime)s %(threadName)s:%(thread)d [%(name)s] %(levelname)s [%(pathname)s:%(lineno)d] %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S',
        },
        'simple': {
            'format': '%(asctime)s [%(name)s] %(levelname)s %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S',
        },
        'test': {
            'format': '%(asctime)s %(message)s',
        },
    },
    'filters': {},
    # 日志处理器
    'handlers': {
        'console_debug_handler': {
            'level': 'DEBUG',  # 日志处理的级别限制
            'class': 'logging.StreamHandler',  # 输出到终端
            'formatter': 'simple'  # 日志格式
        },
        'file_info_handler': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',  # 保存到文件,日志轮转
            'filename': 'user.log',
            'maxBytes': 1024 * 1024 * 10,  # 日志大小 10M
            'backupCount': 10,  # 日志文件保存数量限制
            'encoding': 'utf-8',
            'formatter': 'standard',
        },
        'file_debug_handler': {
            'level': 'DEBUG',
            'class': 'logging.FileHandler',  # 保存到文件
            'filename': 'test.log',  # 日志存放的路径
            'encoding': 'utf-8',  # 日志文件的编码
            'formatter': 'test',
        },
    },
    # 日志记录器
    'loggers': {
        'logger1': {  # 导入时logging.getLogger时使用的app_name
            'handlers': ['console_debug_handler'],  # 日志分配到哪个handlers中
            'level': 'DEBUG',  # 日志记录的级别限制
            'propagate': False,  # 默认为True，向上（更高级别的logger）传递，设置为False即可，否则会一份日志向上层层传递
        },
        'logger2': {
            'handlers': ['console_debug_handler', 'file_debug_handler'],
            'level': 'INFO',
            'propagate': False,
        },
    }
}
# 注：进行日志轮转的日志文件，不能和其他handler共用，不然会导致文件被占用无法更名而报错！
```



## 三、日志使用

```python
import logging.config
import settings

logging.config.dictConfig(settings.LOGGING_DIC)

logger1 = logging.getLogger('logger1')
logger1.info('xxx登录了')

logger2 = logging.getLogger('logger2')
logger2.info('xxx充值了5毛钱')
```



## 四、配置文件方式

```python
创建一个以 .cfg  or .ini  结尾的配置文件
```



但是配置文件不能更改编码方式、如果你的文件是以UTF-8的方式编码的在Windows系统就会乱码报错，因为底层代码编码为None，

1、更改源代码为 UTF-8

2、使用配置文件。

>   该源代码不太好所以还是用配置文件吧





# 第二个ai版本



```python
import logging
import logging.config
import os

# 路径配置
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))  # 项目根目录
LOG_DIR = os.path.join(BASE_DIR, "log")  # 日志目录
INFO_LOG_PATH = os.path.join(LOG_DIR, "info.log")  # 信息日志路径
DEBUG_LOG_PATH = os.path.join(LOG_DIR, "debug.log")  # 调试日志路径

# 确保日志目录存在
if not os.path.exists(LOG_DIR):
    os.makedirs(LOG_DIR)

# 用户设置值：是否将 DEBUG 日志打印到终端
debug_to_console = False  # 默认不打印 DEBUG 日志到终端

# 日志配置字典
LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    # 日志格式
    'formatters': {
        'standard': {
            'format': '%(asctime)s [%(name)s] %(levelname)s [%(filename)s:%(lineno)d] %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S',
        },
        'simple': {
            'format': '%(asctime)s [%(name)s] %(levelname)s %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S',
        },
    },
    # 日志处理器
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple',
        },
        'file_info': {
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': INFO_LOG_PATH,
            'maxBytes': 10 * 1024 * 1024,  # 10MB
            'backupCount': 5,  # 保留5个备份文件
            'encoding': 'utf-8',
            'formatter': 'standard',
        },
        'file_debug': {
            'level': 'DEBUG',
            'class': 'logging.FileHandler',
            'filename': DEBUG_LOG_PATH,
            'encoding': 'utf-8',
            'formatter': 'standard',
        },
    },
    # 日志记录器
    'loggers': {
        '': {  # 根日志记录器，捕获所有未明确配置的日志
            'handlers': ['file_info'],
            'level': 'WARNING',
            'propagate': True,
        },
        'app': {  # 应用日志记录器
            'handlers': ['file_info'],
            'level': 'INFO',
            'propagate': False,
        },
        'debug': {  # 调试日志记录器
            'handlers': ['file_debug'],
            'level': 'DEBUG',
            'propagate': False,
        },
    }
}

# 根据用户设置值动态调整日志配置
if debug_to_console:
    # 如果用户设置为 True，将 DEBUG 日志输出到控制台
    LOGGING_CONFIG['loggers']['']['handlers'].append('console')
    LOGGING_CONFIG['loggers']['app']['handlers'].append('console')
    LOGGING_CONFIG['loggers']['debug']['handlers'].append('console')

# 应用日志配置
logging.config.dictConfig(LOGGING_CONFIG)

# 获取日志记录器
logger_app = logging.getLogger('app')  # 用于应用日志
logger_debug = logging.getLogger('debug')  # 用于调试日志
```

