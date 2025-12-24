---
created: 2025-12-23 14:41
author: natsume37
category:
tags:
  - 
---

## 无限debugger反扒策略

通过不断触发 JavaScript 的 `debugger` 语句，**让浏览器开发者工具不停地暂停执行**，从而**阻止你调试、阅读或修改代码**。

本文章以[码上爬第三题为例](https://www.mashangpa.com/problem-detail/3/) 

![](../../../../Assets/无限debugger反调试策略/file-20251223144904530.png)

鼠标右键显示：

![](../../../../Assets/无限debugger反调试策略/file-20251223144904529.png)

是的、BUFF叠满。

chorm 快捷键打开 `ctrl shift i` 打开后台

![](../../../../Assets/无限debugger反调试策略/file-20251223144904524.png)

页面被替换、代码被 `debugger` 断住无法正常查看请求。

![](Assets/无限debugger反调试策略/file-20251223145040957.png)
只有两个调用栈、最下面的就是debugger的地方
## 调试方案

快速方案： 
1. 全局禁用debugger （此处没用、页面已经被修改）
2. 断点处 Never pause here：从不在此处暂停  （此处也没用）

### 替换文件

鼠标右键 - 点击替换文件 - 选择一个文件夹 将一下内容注释掉（debugger内容代码）刷新页面即可
![](Assets/无限debugger反调试策略/file-20251223145901339.png)

![](Assets/无限debugger反调试策略/file-20251223150037746.png)

### hook钩子技术

把“钩子”挂在关键函数上，原代码一执行，就先被你拦下来。

```js
原函数  →  Hook函数  →  原函数
```

js反扒代码

```js
var debugflag = false;
document.onkeydown = function () {
    if ((e.ctrlKey) && (e.keyCode === 83)) {
        alert("检测到非法调试，小心我抽你");
        return false
    }
}
;
document.onkeydown = function () {
    var e = window.event || arguments[0];
    if (e.keyCode === 123) {
        alert("检测到非法调试，小菜鸡别爬了");
        return false
    }
}
;
document.oncontextmenu = function () {
    alert("检测到非法调试，小菜鸡别爬了");
    return false
}
;
!function () {
    if (window.outerWidth - window.innerWidth > 210 || window.outerHeight - window.innerHeight > 210) {
        document.getElementsByTagName("body")[0].innerHTML = '检测到非法调试, 请关闭调试终端后刷新本页面重试!<br/>Welcome for People, Not Welcome for Machine!<br/>';
        window.location.href = "about:blank";
    }
    let handler = setInterval(() => {
            if (window.outerWidth - window.innerWidth > 210 || window.outerHeight - window.innerHeight > 210) {
                document.getElementsByTagName("body")[0].innerHTML = '检测到非法调试, 请关闭调试终端后刷新本页面重试!<br/>Welcome for People, Not Welcome for Machine!<br/>';

                debugflag = true
            }
            let before = new Date();
            (function () {
            }
                ["constructor"]("debugger")());
            let after = new Date();
            let cost = after.getTime() - before.getTime();
            if (cost > 50) {
                debugflag = true;
                try {
                    document.getElementsByTagName("body")[0].innerHTML = '检测到非法调试, 请关闭调试终端后刷新本页面重试!<br/>Welcome for People, Not Welcome for Machine!<br/>';
                    document.write('检测到非法调试, 请关闭调试终端后刷新本页面重试!<br/>');
                    document.write("Welcome for People, Not Welcome for Machine!<br/>")
                } catch (err) {
                    alert('检测到非法调试, 请关闭调试终端后刷新本页面重试!')
                }
            }
        }
        , 2000)
}();
```

主要有以下反扒策略

- 调试、快捷键禁用
- 窗口大小检查
- 定时器循环

#### 油猴脚本

```js
// ==UserScript==
// @name         码上爬hook脚本
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  绕过禁止F12、右键限制以及 debugger 循环检测
// @author       Martin
// @match        https://www.mashangpa.com/problem-detail/3/
// @run-at       document-start
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // 1. Hook Function 构造函数，防止执行 (function(){})["constructor"]("debugger")()
    const originalConstructor = Function.prototype.constructor;
    Function.prototype.constructor = function (string) {
        if (string === "debugger") {
            console.log("已拦截 debugger 检测");
            return function () {};
        }
        return originalConstructor.apply(this, arguments);
    };

    // 2. 劫持并过滤 setInterval，防止运行定时检测代码
    const originalSetInterval = window.setInterval;
    window.setInterval = function (handler, timeout) {
        let code = handler.toString();
        if (code.includes("debugger") || code.includes("outerWidth") || code.includes("debugflag")) {
            console.log("已拦截定时反调试任务");
            return -1;
        }
        return originalSetInterval.apply(this, arguments);
    };

    // 3. 屏蔽窗口大小检测 (使 outerWidth - innerWidth 永远等于 0)
    Object.defineProperty(window, 'outerWidth', { get: () => window.innerWidth });
    Object.defineProperty(window, 'outerHeight', { get: () => window.innerHeight });

    // 4. 在页面加载后强制清除所有阻碍事件
    window.addEventListener('load', function() {
        document.onkeydown = null;
        document.oncontextmenu = null;
        console.log("按键限制已解除");
    }, true);

    // 5. 防止页面重定向到 about:blank
    window.onbeforeunload = function() {
        return "防止页面因为检测到调试而跳转";
    };

})();
```

