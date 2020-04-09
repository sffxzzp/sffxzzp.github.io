---
title: 用 CF-Workers 来做 Github Releases 的下载加速
date: 2020-04-09T22:26:58+08:00
tags:
  - JavaScript
  - CloudFlare
  - Workers
comments: true
---
## 前情提要

Github 的下载一直都非常令人诟病，虽然这锅并不能让 Github 或者微软来背，但总体来说对国内的用户还是非常不友好的。

jsdelivr 有一个对 Github 的 CDN 加速，但尝试了半天似乎并不能够直接下载 Release 里附带的文件（assets），只能下载仓库里的代码文件。虽然说把二进制文件 push 到仓库里也可以曲线救国，但 push 的过程如果没有魔法的话，也着实令人觉得有些痛苦。

## CloudFlare Workers

CloudFlare Workers 顾名思义，是另一个 CDN 服务商 CloudFlare 提供的一个可以部署无服务器的 JavaScript 应用的服务。

但我估计这项服务的本来用意，应当是利用边缘服务器的计算能力来降低源服务器负载的。将一些普通而简单的处理直接在边缘服务器完成，也可以提高响应速度。

## 说正题

这次要说的就是利用 Workers 的功能来对 Github 的文件进行转发，虽然在某些地区 CloudFlare 的网速也是拉跨，但对于移动用户来说还是比较福音的。而且目前每天的免费配额是 10w 次每天，非常富裕了。

总体来说就以下几步：

1. 根据访问链接获取参数
2. 根据参数获取目标文件链接
3. 下载（缓存）并转发

那么一步步来。

------

### 1. 根据访问链接获取参数

首先创建一个 Worker 后，会给出以下默认代码。

``` JavaScript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

/**
 * Respond to the request
 * @param {Request} request
 */
async function handleRequest(request) {
  return new Response('hello world', {status: 200})
}
```

其中 handleRequest 就是我们用于处理请求的函数了，接下来我们的工作就是对这个函数进行修改。

------

首先我想要最终用于加速的 URL 链接和 jsdelivr 类似，应当形如 `https://domain.com/gh/JustArchiNET/ArchiSteamFarm/ASF-generic.zip` 一样（此处以 ASF 的下载链接为例）。

格式大约为 `https://<domain>/gh/<user>/<repo>[/tag]/<file>`。

其中 `/tag` 部分为可选，如果有就是下载指定 tag 下的文件，没有就是从最新的 Release 中下载文件。

那么把链接以 `/` 分割开来，去掉域名用的那个之后，剩下都是我们需要的参数了。

然后再请求 Github 的 WebAPI，就可以顺利获取文件的下载地址了。

所以对函数稍作修改如下：

``` JavaScript
async function handleRequest(request) {
  // 分割路径
  var pathname = (new URL(request.url)).pathname.split('/');
  // 判断第二个参数是不是 gh，第一个是域名，就忽略掉了。
  if (pathname[1] == 'gh') {
    var jsonurl, targetfile;
  // 看参数个数是不是 5 个（包括域名），这是不含 tag 的。
    if (pathname.length == 5) {
  // 设置 Github WebAPI 链接，以及目标文件名。
      jsonurl = `https://api.github.com/repos/${pathname[2]}/${pathname[3]}/releases/latest`;
      targetfile = pathname[4];
    }
  // 参数个数为 6，包含 tag。
    else if (pathname.length == 6) {
      jsonurl = `https://api.github.com/repos/${pathname[2]}/${pathname[3]}/releases/tags/${pathname[4]}`;
      targetfile = pathname[5];
    }
  // 接下来的代码写在这里
  }
  return new Response('hello world', {status: 200});
}
```

这样就可以处理完所有 URL 参数了，不过因为没有访问接口，所以并没有获取到文件链接。

那么接下来要做的是第二步。

------

### 2. 根据参数获取目标文件链接

已经根据参数拼接好 API 地址之后，接下来就是从 API 获取文件链接了。

不过一开始写的时候由于对 fetch 函数不熟悉浪费了不少时间。

``` JavaScript
// 从 API 地址获取数据
var jdata = await fetch(jsonurl, {headers: {'User-Agent': 'CloudFlare-Workers'}});
jdata = await jdata.json();
// 判断指定的 tag 是否包含 assets 文件。
if ('assets' in jdata) {
  var assets = [];
  for (let i in jdata.assets) {
    let ass = jdata.assets[i];
// 判断是否与目标文件名相同
    if (targetfile == ass.name) {
// 获取文件下载链接
      var downloadLink = ass.browser_download_url;
// 接下来处理下载、缓存并转发。
    }
  }
// 文件名没有相同的，报找不到文件错误。
  return new Response('file not found', {status: 404});
}
```

这一段没什么好说的，进入第三步。

------

### 3. 下载（缓存）并转发

其实转发简单的很，直接 return 一个 fetch 即可。至于文件缓存，只要去查看一下 CloudFlare Workers 的[文档](https://developers.cloudflare.com/workers/about/using-cache/)即可。

即只要给 fetch 加上 `{cf: { cacheEverything: true, cacheTtl: 3600 }}` 的参数即可。

``` JavaScript
return fetch(downloadLink, {cf: { cacheEverything: true, cacheTtl: 3600 }});
```

------

稍微综合一下代码：

``` JavaScript
if (pathname[1] == 'gh') {
  var jsonurl, targetfile;
  if (pathname.length == 5) {
    jsonurl = `https://api.github.com/repos/${pathname[2]}/${pathname[3]}/releases/latest`;
    targetfile = pathname[4];
  } else if (pathname.length == 6) {
    jsonurl = `https://api.github.com/repos/${pathname[2]}/${pathname[3]}/releases/tags/${pathname[4]}`;
    targetfile = pathname[5];
  }
  var jdata = await fetch(jsonurl, {headers: {'User-Agent': 'CloudFlare-Workers'}});
  jdata = await jdata.json();
  if ('assets' in jdata) {
    var assets = [];
    for (let i in jdata.assets) {
      let ass = jdata.assets[i];
      if (targetfile == ass.name) {
        return fetch(ass.browser_download_url, {cf: { cacheEverything: true, cacheTtl: 3600 }});
      }
    }
    return new Response('File Not Found!', { status: 404 });
  }
  else {
    return new Response(JSON.stringify(jdata), { status: 200 });
  }
}
```

大功告成，具体的源码可以[访问此处](https://github.com/sffxzzp/cfworker-scripts/blob/master/cdn/index.js)查看。
