# requuests 基本使用
## 下载

```python
pip3 install requests
```

## 基本使用

```python
import requests  
  
url = 'https://movie.douban.com/top250'  
  
header = {  
	# 请求头、用于反爬虫验证
    "User-Agent":"*Mozilla/5.0 (Windows NT 10.0; Wi*n64; x64) AppleWebKit/537.36 (KHTML, like Gecko) ****"  
}  
res = requests.get(url, headers=header)  
print(res.text)
```

> 上述代码将会发送请求到指定网站、并获取网站源代码、解析为 txt 文件

requests 库的常见请求方式：
- get()
- post
- put
- delete
- patch

> 最常用的还是 post、get 请求其他了解即可。

### url 中传递参数

当需要在 url 中传递参数时（httpbin.org/get?key=val）、可以以字典的方式传递-- **params** 关键字

https://www.httpbin.org/get?name=germey&age=15

python 构建代码：
```python
import requests  
  
url = 'https://movie.douban.com/top250https://www.httpbin.org/get?name=germey&age=15'  
header = {  
    "User-Agent": 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) obsidian/1.2.7 Chrome/112.0.5615.87 Electron/24.1.2 Safari/537.36'  
}  
data = {  
    'name': 'gerty',  
    'age': 25  
}  
res = requests.get(url, params=data, headers=header)  
print(res.text)
```

### 返回值学习

![image.png](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/090717202309071721246870ad97223.png)

### data 参数
实现服战的表单、将字典编码成表单数据。

```python
import requests  
  
payload = {  
    'ky1': "value1",  
    'ky2': "value1"  
}  
  
res = requests.post('https://httpbin.org/post', data=payload)  
print(res.text)
```

效果：
```python
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "ky1": "value1", 
    "ky2": "value1"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "21", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.31.0", 
    "X-Amzn-Trace-Id": "Root=1-64f996e5-4ee39f925261bfa5318f0c13"
  }, 
  "json": null, 
  "origin": "103.47.100.216", 
  "url": "https://httpbin.org/post"
}
```
### JSON 参数

```python
url = 'https://api.github.com/some/endpoint'
payload = {'some': 'data'}

res = requests.post(url, json=payload)
print(res.text)
```

### 文件上传（files 参数）：

```python
url = 'https://httpbin.org/post'
files = {'file': open('report.xls', 'rb')}

r = requests.post(url, files=files)
r.text

{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
```
### cookies 发送

```python
url = 'https://httpbin.org/cookies'
cookies = dict(cookies_are='working')

r = requests.get(url, cookies=cookies)
r.text
'{"cookies": {"cookies_are": "working"}}'
```
### 超时-timeout 参数（超时设置）

```python
requests.get('https://github.com/', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)
```

## 高级用法

### session 回话维持

它会在所有请求中保存一些参数、在 session 实例保留 cookie、并将使用 urllib 3 链接池。

#### 代码演示（保存 cookie）

```python
s = requests.Session()

s.get('https://httpbin.org/cookies/set/sessioncookie/123456789')
r = s.get('https://httpbin.org/cookies')

print(r.text)
# '{"cookies": {"sessioncookie": "123456789"}}'
```

会话还可以用于向请求方法提供默认数据。这是通过向 Session 对象的属性提供数据来完成的：

```python
s = requests.Session()
s.auth = ('user', 'pass')
s.headers.update({'x-test': 'true'})

# both 'x-test' and 'x-test2' are sent
s.get('https://httpbin.org/headers', headers={'x-test2': 'true'})
```

请求头更新：

```python
s = requests.Session()
s.auth = ('user', 'pass')
s.headers.update({'x-test': 'true'})

# both 'x-test' and 'x-test2' are sent
s.get('https://httpbin.org/headers', headers={'x-test2': 'true'})
```
> 您传递给请求方法的任何字典都将与设置的会话级值合并。方法级参数覆盖会话参数。

注意，即使使用会话，方法级参数也不会_在请求之间保留。

```python
s = requests.Session()

r = s.get('https://httpbin.org/cookies', cookies={'from-my': 'browser'})
print(r.text)
# '{"cookies": {"from-my": "browser"}}'

r = s.get('https://httpbin.org/cookies')
print(r.text)
# '{"cookies": {}}'
```
## 回话的上下文管理器
代码实例

```python
with requests.Session() as s:
    s.get('https://httpbin.org/cookies/set/sessioncookie/123456789')
```

> 这将确保一旦 `with` 退出块，会话就会关闭，即使发生未处理的异常也是如此。

## 代理-proxies

### 单个请求配置

```python
import requests

proxies = {
  'http': 'http://10.10.1.10:3128',
  'https': 'http://10.10.1.10:1080',
}

requests.get('http://example.org', proxies=proxies)
```

### 回话配置

```python
import requests

proxies = {
  'http': 'http://10.10.1.10:3128',
  'https': 'http://10.10.1.10:1080',
}
session = requests.Session()
session.proxies.update(proxies)

session.get('http://example.org')
```