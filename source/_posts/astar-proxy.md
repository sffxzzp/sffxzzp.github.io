---
title: Astar 插件分析
date: 2020-10-27T17:45:13+08:00
tags:
  - 随笔
  - Python
  - JavaScript
comments: true
---
## 前情提要

之前有人给我推荐这个插件，并且问是不是能提取出来用。我试了下体验确实不错，所以花了点时间分析了下这个插件。

## 开始分析

这次的插件来自谷歌市场，插件ID是`jajilbjjinjmgcibalaakngmkilboobh`。

### 1. 查找 manifest.json

谷歌浏览器插件一般都有个`manifest.json`来存储插件的各种属性。

所以此次也是从这个文件开始。

其中的`background`段就是此次的目标了。[[引用](https://developer.chrome.com/extensions/background_pages)]

``` JSON
"background": {
    "scripts": [ "/js/jquery-2.1.1.js", "/js/MD5.js", "/js/aes.js", "/js/ss.js"]
}
```

对文件名稍作分析可知，也就只有`ss.js`可能与本次分析相关了。

### 2. 分析 js 文件

打开这个文件阅读代码，这压缩过的代码看着头疼，先整个 beautifier 美化一下代码，增加可读性。

然后很轻松就能找到 clientFun() 函数，附录如下（对域名做省略处理）。

``` JavaScript
function clientFun() {
    this.init = function (e) {
        $.ajax({
            url: "https://.../getProxyList?" + (new Date).getTime(),
            type: "post",
            dataType: "json",
            data: {
                strP: chrome.runtime.id
            },
            success: function (o) {
                var t = CryptoJS.enc.Utf8.parse(hex_md5(o.s).substring(0, 16)),
                    r = CryptoJS.AES.decrypt(o.d, t, {
                        mode: CryptoJS.mode.ECB,
                        padding: CryptoJS.pad.Pkcs7
                    }),
                    n = "" + CryptoJS.enc.Utf8.stringify(r),
                    i = $.parseJSON(n)
                if (0 != i.nCode) return localStorage.state = "0", localStorage._click = "1", chrome.browserAction.setBadgeBackgroundColor({
                    color: "#FFFFFF"
                }), chrome.browserAction.setBadgeText({
                    text: ""
                }), server.req({
                    n: 0
                }), void console.info("service exception")
                localStorage._sl = JSON.stringify(i.jsonObject), localStorage._s = o.s
                var s = localStorage.state
                return void 0 == s ? void(void 0 != e && null != e && server.req({
                    n: e
                })) : "0" == s ? void(void 0 != e && null != e && server.req({
                    n: e
                })) : (p.exceptionNumber = 0, void client.getProxy())
            },
            error: function () {
                console.info("service net exception")
            }
        })
    }, this.getProxy = function () {
        var e = localStorage._s
        if (void 0 != e) {
            var o = localStorage._i
            void 0 != o && $.ajax({
                url: "https://.../getProxy?" + (new Date).getTime(),
                type: "post",
                dataType: "json",
                data: {
                    strP: chrome.runtime.id,
                    strtoken: e,
                    lid: o
                },
                success: function (e) {
                    var t = CryptoJS.enc.Utf8.parse(hex_md5(e.s).substring(0, 16)),
                        r = CryptoJS.AES.decrypt(e.d, t, {
                            mode: CryptoJS.mode.ECB,
                            padding: CryptoJS.pad.Pkcs7
                        }),
                        n = "" + CryptoJS.enc.Utf8.stringify(r),
                        i = $.parseJSON(n)
                    if (102 != i.nCode) return localStorage.state = "0", localStorage._click = "1", chrome.browserAction.setBadgeBackgroundColor({
                        color: "#FFFFFF"
                    }), chrome.browserAction.setBadgeText({
                        text: ""
                    }), server.req({
                        n: 0
                    }), void console.info("proxy line exception,please select other proxy line.")
                    p.on(i.jsonObject), localStorage._click = "1", server.req({
                        n: 1
                    })
                    var s = localStorage._sl
                    if (void 0 != s) {
                        for (var i = JSON.parse(s), c = 0; c < i.d.length; c++) void 0 == o ? 0 == c && (chrome.browserAction.setBadgeBackgroundColor({
                            color: [16, 201, 33, 100]
                        }), chrome.browserAction.setBadgeText({
                            text: i.d[c].p.replace(".png", "")
                        })) : i.d[c].i == o && (chrome.browserAction.setBadgeBackgroundColor({
                            color: [16, 201, 33, 100]
                        }), chrome.browserAction.setBadgeText({
                            text: i.d[c].p.replace(".png", "")
                        }))
                        p.exceptionState = 0
                    }
                },
                error: function () {
                    console.info("service net exception")
                }
            })
        }
    }, this.heartDump = function () {
        var e = localStorage._s,
            o = localStorage._i,
            t = localStorage._sl,
            r = localStorage.state
        if (void 0 != e && void 0 != o && void 0 != t && 0 != r) {
            for (var n = JSON.parse(t), i = "", s = 0; s < n.d.length; s++) void 0 == o ? 0 == s && (i = n.d[s].n) : n.d[s].i == o && (i = n.d[s].n)
            $.ajax({
                url: "https://.../heartDump?" + (new Date).getTime(),
                type: "post",
                dataType: "json",
                data: {
                    strP: chrome.runtime.id,
                    strtoken: e,
                    lid: o,
                    strlognid: i
                },
                success: function (e) {},
                error: function () {}
            })
        }
    }, this.timeSend = function () {}
}
```

根据 ajax 代码很容易就知道发送了哪些请求。

首先 POST 了 getProxyList 这个 URL，来获取节点列表。

然后根据获取的节点信息再次 POST getProxy，来获取具体的节点配置。

### 3. 使用 Python 来获取信息

首先不管三七二十一，先整上两个请求。

但直接请求拿到的内容却是 404？

从浏览器 background.html 中用 fetch 试了一下，又是可以拿到数据的。

仔细比对了两个请求，发现竟然是对 Origin 头做了验证，仿照一下就能正常拿到数据了。

getProxyList 返回结果如下：

``` JSON
{
    "s":"0d327617ce254bc881bfedfc87ee28e7",
    "d":"af4Gqt15mMWCabEiFWd9AVYn3xYxZB7UeKc2GGL/9iBDTglBIp6a/hDN8MxF9tEioKYdZ16v5f90OUtcUhtAkwK4rJC+kW+HMAU6igJ4jcjXPBzDOJKhYM9Kmk5LEYrfKqnxnwldGIsvhYqGVEw4oRltxKlB6D5EQH5idI5iP2PWSCSOuxeeiST0sYGEBi/9gIUs7KKs/XJ3sWA3ILZJKG7FDSczQFpDjK5GlzcEU7MHppALTJxEDyCXBOco6VmrA8d3u8cLylVspFNfFzpRnAiQ/Q4g5u6n7tYXcv4BjVK1JhExTQEorSZEclg7TC/u/JgWbKOb6foRFwYFgEqHA1dyHuiF+XcbmxwgWz/jyl/aOqxb0ydHe3onzhGbSUZBslaTJb4sdhQeaS9se28HZ66fzQAZQVfHFgYvcT7assnCLbNPijR70MkrbTmMtGMpAxeCccPxLYqnny+juAhbwWk0FNjFjvXPxymNqsWl5811jrR0Vhn2hY1gOpROaVmTRsJV0hfDlF0qckyenzkGcGPSy9yAfrJsYUm+ZC7KpmegqHAqM0rzTMF0rq0a6ppEM4738cSitwZ79bHxOCX/7u8DwmOQMUvmGHHA3FPoQ99hF9qJA1A9LGk6KggRi19gClD7ctLhNRArmxwb3LlJ1POl3hgki19hzrR5kf6J7HPMSK/av9F57ZidCcM6SHxRKNnUtmx+jsgoZCrUfY/BijfJoYAvu5RhLRrvAD7fmO8="
}
```

从上文 js 代码可知，要对返回的数据做一套 AES 解密才能拿到真实数据。

解密后内容如下：

``` JSON
{
    'strText': 'succ',
    'nCode': 0,
    'jsonObject': {
        'd': [
            {'n': 'Los Angeles.US', 'i': 25, 'p': 'US.png', 'l': 2},
            {'n': 'Japan', 'i': 17, 'p': 'JP.png', 'l': 2},
            {'n': 'Singapore', 'i': 34, 'p': 'SG.png', 'l': 2},
            {'n': 'Germany', 'i': 26, 'p': 'DE.png', 'l': 2},
            {'n': 'Canada', 'i': 27, 'p': 'CA.png', 'l': 2},
            {'n': 'India', 'i': 35, 'p': 'IN.png', 'l': 2},
            {'n': 'France', 'i': 20, 'p': 'FR.png', 'l': 2},
            {'n': 'Poland', 'i': 12, 'p': 'PL.png', 'l': 2},
            {'n': 'Netherlands', 'i': 15, 'p': 'NL.png', 'l': 2},
            {'n': 'Australia', 'i': 19, 'p': 'AU.png', 'l': 2},
            {'n': 'United Kingdom', 'i': 31, 'p': 'GB.png', 'l': 2}
        ],
        'i': '125.127.210.252'
    }
}
```

节点列表就有了，然后继续访问 getProxy。

可得：

``` JSON
{
    "s":"5e3ff55c11154cf1b7865b101e8358b5",
    "d":"DPc330eCV/RpmAXU2OncatIN476wsUcpvP8EtND85CxUtrVDrbIsTzu5hRUVcp2l4SHQiFkJ6W9Mi+QfuMhwK53f+0bieRXd7kJqRlUu92dABCtV7b8T4XfppwM8JCK2HsicDGLtn1iHiiPG0SawdcPKHSud6pev/7RwH8pV/9IUZjkBHs61lv6RUmHEsxiVg1v4ufLC0Ifm18o8L0fHFsqRW2gdwL3O3EBEGCvkuD/DJaFflIEq2lq1VjnSOcatMKLFwN24fSQwSv9SCcW6vCcol6PxhTVe2xRkAiIQSy5fEiRkrOsfBuizD6tj3eWisB01WrauFx2lVoA59jJ0Obu4GV3B/xjBaRTT/NzBQsC4q0rz2zhjxF5cvm82t+yPAQXN1ylStvdctxdLpYbLrGJE82PlkDA9P94g5xu/UAH/9mvshsdzmlYlE1Z1N79QfVxUE9fiwimtdE6NM0U7uTWY63A9wTgW8HG+tXNjCabHAc1zMyTm2C5TT8q95SqSFscR12s5+zKo4GY6t8lPmltBll9aDIcuYfb9mgLyxPj47NyB5syaBpkbx5TnvcveIKSmmz6aIik2dHViVtfLuPl3m4SfnZKdKzubaQKecz/Nk4eFCX6wYGI2cbYYBNXL"
}
```

再次解密可得：

``` JSON
{
    'jsonObject': {
        '_s': 'TJCIcFqpRJ8Q57YwSu81AgZ+WnE04YlycPm6lNPCAh3JBKqCU6ZSXUkjUQcyw5fbrCbFRrja5NyuntYZmtTff3iQcWcWffd00Bq5SYsLxVjOmVlKmQOSkrlx44YS2Pkdo2nLwDtulNiYjL1QugF0+8itMxq2uFJJ4X8Hn4FhXh8YEd2ErpKzo9X5AUjKk9vQXuHgjHW27mKVUkRHDJBYlgqlBNzBnTkXNGCCaVXGq54/leypsveEyKRYhJOi84c8P36hLDbTidhxi34foLmQ52ymPyq6LbmHJvveb8ky3Aj8FvFM9lSzfejzFkr5pvPXiWehqafpf4VnYyKtYM02gUeT4awNTaoo6e3X8Im2CO5fO3Zf0MLsQOvZ7mYpPWJ/',
        '_p': '32b3b045a013494dbf6c3177e34f63b4'
    },
    'nCode': 102,
    'strText': 'succ'
}
```

然后对 jsonObject 的内容再次解密，就可以获得 pac 地址了。

再从 pac 中提取出代理地址即可直接使用。

## 写在最后

这篇文章仅是个人爱好的一篇小结，**请勿**进行直接或间接的滥用。
