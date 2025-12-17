---
created: 2025-12-17 15:23
author: natsume37
category:
tags:
  - 
---
# B站视频下载分析

最近在学习一些东西需要用到B站的原视频、通过原视频加上阿里的[通义听悟](https://tingwu.aliyun.com/home)能够快速的做记笔记、生成文稿、同时通过采集音频等信息也给后续的学习添加了少许便利。

![](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/file-20251217153456057.png)


随便打开一个视频（这是我python的启蒙老师他的课程真的很不错、全是干货）

## 请求分析

![](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/file-20251217154228036.png)

这个请求就是我们浏览器URL的地址 观察返回的数据

![](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/file-20251217154357156.png)

返回的是一个 `HTML` 页面

![](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/file-20251217154542019.png)

在 `script` 代码里定义了有我们需要的信息

![](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/file-20251217154709356.png)

![](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/file-20251217154741714.png)

主要包括：**当前账户支持的**不同分辨率的视频链接、以及音频链接

经过gemini了解到：B站使用 **DASH (Dynamic Adaptive Streaming over HTTP)** 技术来播放视频。这意味着视频（画面）和音频（声音）是分开传输的，并且视频有多种清晰度和编码格式。

- 请求视频URL：  
https://www.bilibili.com/video/BV1Hx2DBhEve/?vd_source=1a5796df38d45b002f615567954d7129&spm_id_from=333.788.player.switch
- 用正则表达式截取 `window\.__playinfo__=(.*?)</script>` 的内容将其转为python的字典格式，方面我们对url链接进行提取
- 用 `requests` 库携带 `cookie` 以及 `header` 进行请求获取 `video` 和 `audio`
- 用 `FFmpeg` 工具对m4s切片进行合并以及转码处理、转为MP4视频文件



### 注意

在后续请求中一定要加 `Referer` 要不然一直443

（说起来很奇怪、我爬的时候一直打不开、我写文章的时候能正常请求了???）

## 实验代码

```python
import requests  
  
url = r'https://www.bilibili.com/video/BV1Hx2DBhEve/'  
cookie = "你自己的cookie"  
headers = {  
    "Referer": url,  
    "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36",  
  
}  
s = requests.Session()  
res = s.get(url, headers=headers)  
print(res.text)  
  
url1 = 'https://upos-sz-estghw.bilivideo.com/upgcxcode/03/23/34616772303/34616772303-1-30120.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&oi=3747235012&gen=playurlv3&deadline=1765956052&nbs=1&platform=pc&trid=dae5b19cf24146a5a2cd90e416074bdu&mid=646388511&os=estghw&og=hw&uipk=5&upsig=36f83070a157c648882e2358468116dc&uparams=e,oi,gen,deadline,nbs,platform,trid,mid,os,og,uipk&bvc=vod&nettype=0&bw=1610484&build=0&dl=0&f=u_0_0&agrr=0&buvid=D303155E-E607-49D6-67EB-9F307CD1320157336infoc&orderid=0,3'  
video = s.get(url1, headers=headers)  
with open('video.m4s', 'wb') as f:  
    f.write(video.content)  
  
print("video downloaded")  
  
url1 = 'https://upos-sz-estghw.bilivideo.com/upgcxcode/03/23/34616772303/34616772303-1-30280.m4s?e=ig8euxZM2rNcNbdlhoNvNC8BqJIzNbfqXBvEqxTEto8BTrNvN0GvT90W5JZMkX_YN0MvXg8gNEV4NC8xNEV4N03eN0B5tZlqNxTEto8BTrNvNeZVuJ10Kj_g2UB02J0mN0B5tZlqNCNEto8BTrNvNC7MTX502C8f2jmMQJ6mqF2fka1mqx6gqj0eN0B599M=&uipk=5&platform=pc&mid=646388511&og=hw&deadline=1765956052&nbs=1&trid=dae5b19cf24146a5a2cd90e416074bdu&oi=3747235012&gen=playurlv3&os=estghw&upsig=782c79a8a8c15e8b53c6d9b6a4d0e0a6&uparams=e,uipk,platform,mid,og,deadline,nbs,trid,oi,gen,os&bvc=vod&nettype=0&bw=53929&agrr=0&buvid=D303155E-E607-49D6-67EB-9F307CD1320157336infoc&build=0&dl=0&f=u_0_0&orderid=0,3'  
audio = s.get(url1, headers=headers)  
with open('audio.m4s', 'wb') as f:  
    f.write(audio.content)  
  
print("audio downloaded")
```

audio和video被正常下载

## 参数化

将常量改为变量处理、这里唯一需要变的就是 `cookie` 和 `视频URL` 。我尝试写登录脚本、奈何现状有二维码验证（还没学）、目前只能手动复制 `cookie` 来解决。

通过 FFmpeg 进行合并转码操作、将m4s切片转为MP4等其它格式。
### 效果

![](https://natsume-1316988601.cos.ap-chengdu.myqcloud.com/image_own/file-20251217162433097.png)

