---
title: 记一次对 ZzzFun 网站的分析
date: 2020-05-17T13:35:43+08:00
tags:
  - JavaScript
comments: true
---
## 前情提要

其实没什么特别的，就是看到坛友推荐了这个站，而且看起来确实很快，所以想下载回来看，毕竟在线不能加速到习惯的 3 倍速。

但有意思的是，当我按下了 F12，网站直接就回到了首页。看起来是有检测 debugger 的机制在，所以这次是人工直接分析代码了。

## 开始分析

其实这次的分析有点运气的成分在里面，不过最终可以下载了就行。

而且似乎也不太好按照步骤来分，那我就想到哪儿说到哪儿吧。那就以《辉夜大小姐》为例，简要分析一下好了。

### 1. 查看源码

网站虽然在播放页面禁止了 F12，但在分集页面还是没有禁的，这点降低了许多难度。

当然具体的分析还是要先进入播放页面，由于右键同样被禁用，所以只好使用在地址栏添加 view-source 的方式查看源码。

页面内容其实不多，所以可能包含有用信息的部分直接就能看到。

``` JavaScript
var maccms = {
    "path": "",
    "mid":"1",
    "aid":"15",
    "url":"www.zzzfun.com",
    "wapurl":"m.zzzfun.com",
    "mob_status":"1"
};
var player_data = {
    "flag": "play",
    "encrypt": 2,
    "trysee": 0,
    "points": 0,
    "link": "\/vod-play-id-1868-sid-1-nid-1.html",
    "link_next": "\/vod-play-id-1868-sid-1-nid-2.html",
    "link_pre": "",
    "url": "JTdBJTZGJTZFJTY1JTNGJTZCJTY1JTc5JTNEJTMxJTMwJTMwJTM2JTVGJTY1JTY1JTM4JTM3JTMzJTYxJTM0JTY1JTM5JTMzJTM4JTMzJTM0JTM0JTMyJTM0JTYyJTMyJTM1JTM1JTM1JTM5JTYzJTYzJTYxJTMwJTYyJTM0JTY2JTY0JTM0JTM0",
    "url_next": "JTdBJTZGJTZFJTY1JTNGJTZCJTY1JTc5JTNEJTMxJTMwJTMwJTM2JTVGJTM5JTY0JTY2JTMwJTM5JTM1JTMwJTMwJTMyJTM0JTYzJTY1JTM0JTM2JTM0JTMyJTM5JTM3JTM0JTM1JTMzJTYyJTYzJTY0JTM5JTYyJTM3JTY1JTM2JTM4JTMxJTYz",
    "from": "letv",
    "server": "no",
    "note": ""
}
```

其中最可疑的就是其中的 url 字段了，但这个解密实在看起来没什么头绪（现阶段）。

所以只好先去翻一下相关的 js 文件了。

### 2. 搜索 js

在播放页面的源码中搜索了一下 `.js`，然后排除掉库文件，可以得到几个比较可能的文件。

再按照文件名排除了一下，最后只有 `player.js` 比较符合条件。

再在 `player.js` 中搜索了一下 `player_data` 字段，果然有一处匹配，那看来就是我们想要的了。

不过文件的后半部分以 `eval(function(p,a,c,k,e,r)` 开头，多半是经过了混淆，所以只能先找个反混淆工具了。

幸运之处就在于此段代码的混淆程度并不高，所以网上随便找了个反混淆工具就可以解出原始代码。代码由于有点长就不贴了，只贴出找到的关键部分。

``` JavaScript
this.PlayFrom = player_data.from;
this.Path = maccms.path + '/static/player/';

document.write('<scr' + 'ipt src="' + this.Path + this.PlayFrom + '.js"></scr' + 'ipt>')

if (player_data.encrypt == '1') {
    player_data.url = unescape(player_data.url);
} else if (player_data.encrypt == '2') {
    player_data.url = unescape(base64decode(player_data.url));
}
```

在这里我们就能找到个新的 js 地址，以及真正的播放页面的地址。

### 3. 进一步分析

所以根据上一步：

``` JavaScript
js = '/static/player/letv.js'
url = 'zone?key=1006_ee873a4e93834424b25559cca0b4fd44'
```

查看一下这个新的 js，可以得到：

``` JavaScript
MacPlayer.Html = '<iframe width="100%" height="100%" src="'+maccms.path+'/static/danmu/zone.php?'+MacPlayer.PlayUrl+'" frameborder="0" border="0" marginwidth="0" marginheight="0" scrolling="no" allowfullscreen="true" allowtransparency="true"></iframe>';
MacPlayer.Show();
```

最终组合一下 iframe 的地址就是：`/static/danmu/zone.php?zone?key=1006_ee873a4e93834424b25559cca0b4fd44`

进一步访问这个地址。

``` HTML
<source src="http://vwecam.tc.qq.com/1006_ee873a4e93834424b25559cca0b4fd44.f0.mp4?vkey=AF290E0BEC7F19E09D483DFA19CBACE34F1C0DEE1DB044A4A572CE543BFD6FC99252BF939AA656E2A394BFF49D1CFFC824C221B5551E66EC&rf=206" onerror="load_fail[0]()" type="video/mp4">
```

就是我下载所需的原文件了。

### 4. 下载

之后其实没什么好说的，整合一下上面的必需步骤，写了个段 Python 脚本就可以直接下载了。

不过整个这个过程还是相当有意思的。

## 写在最后

这篇文章仅是个人爱好的一篇小结，**请勿**进行直接或间接的滥用。
