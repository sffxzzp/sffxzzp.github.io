---
title: Readably 和 My Reader
date: 2019-11-23T03:57:45.127Z
tags:
  - 安卓
  - Win10
  - RSS
comments: true
---
## 前情提要

作为一个 RSS 用户，觉得 RSS 真的是非常方便，且我也很早就开始使用 RSS 来订阅消息了。

可遗憾的是，Google Reader 早在 13 年就被谷歌给关停了，之后也紧跟着关闭了多家 RSS 服务商。

之后作为替代也使用过 Feedly、TT-RSS、Inoreader 等服务，但要么是访问困难，要么需要自己搭服务器。

最麻烦的还是 RSS 客户端的问题。15 年及之前，一直是在用 NOINNION 的 News+ 应用（安卓），但 15 年之后这个客户端就再也没更新过，支持以及使用体验也越来越差。

后来有很长一段时间就没怎么正经的用过 RSS 了，一直是用 Kindle-Ear 拉取后推送到 Kindle 再看的，但 Kindle 的主业毕竟是泡面盖（大雾）。直到最近一段时间找到了新的替代品，才让我重拾每天阅读 RSS 的快乐。

## Readably

这个 App 是最近在 Play 商店找到的，支持 Feedbin、Inoreader 以及 Fever API。

阅读起来倒是比较不错，对夜间模式没有需求的话也可以不买高级版。

当然最主要还是在这个支持 Fever API 上，如果对 RSS 服务要求比较高的话，可以自行搭建 TT-RSS（带 Fever API 插件）。当然也可以使用 [Stringer](https://github.com/swanson/stringer)，不光支持 Fever API，还支持部署于 Heroku，且提供的免费额度完全够用。

## My Reader

这是一款 Win10 上的 UWP 应用，支持 Feedly、Inoreader、The Old Reader 和本地 RSS，可惜的是不支持 Fever API。

如果你用 Inoreader 的话就可以手机电脑两边都用 App 了，不过 PC 端的话 TT-RSS 和 Stringer 的页面表现也还算是不错。

如果你只用 PC 看 RSS 的话，完全可以用这个软件的本地 RSS，软件会在后台抓取 RSS，你打开就能直接阅读了，使用体验也还算不错，但相比 Readably，在源服务这里支持的比较少一些了。
