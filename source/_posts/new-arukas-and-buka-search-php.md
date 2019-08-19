---
title: new-arukas-and-buka-search-php
date: 2019-08-19T13:03:21.270Z
tags:
  - 随笔
  - PHP
  - Docker
comments: true
---
自从上次说想要给布卡搜索做一个网页版的，这几天也在看相关的一些信息。
首先是主机，本来想用的是 now.sh，但后来看了一下升级后的 2.0 版本，发现并不能用 docker 而且只能做前后端分离式的感觉。

正巧又想起之前用过的 Arukas，现在支持 zfb 认证了，而且还是有免费配额的。
于是想了想还是认证了一下，又解了绑定。现在用的还是挺不错的。

网页版搜索基于 PHP 和 SQLite，由于对 PHP 的 SQLite 不熟，所以直接用了 Medoo 框架，非常舒适，直接把搜索部分移植过来了。
Arukas 也是基于 Docker，所以花了点时间写 Dockerfile，而且没调试直接丢上 Docker Hub 进行自动构建，然后再推到 Arukas，一气呵成。

网址见（可能由于各种原因变更）：[https://bukasearch.ore-imo.tk](https://bukasearch.ore-imo.tk)

总之我个人觉得还是挺满意的，之后可以把一些其他的东西也都移到 Arukas 这边来。
