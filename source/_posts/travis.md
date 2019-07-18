---
title: 最近用Travis-CI有感
date: 2019-07-17T18:22:26.000Z
tags:
  - 随笔
comments: true
---
一直以来更新[Starbound汉化](https://github.com/sffxzzp/Starbound-Chinese)都是手动更新。
也是机缘巧合，前段时间正好想起Travis，出于尝试的心态，给项目设置了Travis。

目前在push之后会自动检查文件是否符合JSON Patch格式，自动转换编码至utf-8，自动打包pak并发布到Github Releases。
没想到的是修正汉化的时候竟然可以获取Python的退出信息直接报错。
也是花了一番功夫，不过实际效果确实不错，当然也要感谢一下xw。

目前我在用Travis的项目有4个，分别是汉化，布卡搜索，Steam汉化信息，还有就是这个Hexo了。
总体来说相当好使，觉得以后可以继续深挖一下。
就说这么多吧。
