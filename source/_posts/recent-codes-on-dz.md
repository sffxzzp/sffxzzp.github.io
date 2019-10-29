---
title: 最近写Discuz插件时的一些记录
date: 2019-10-29T16:13:05.520Z
tags:
  - Discuz
  - PHP
  - 随笔
comments: true
---
## 前言

最近撸 Discuz 的插件的时候，遇到了一些问题。

不得不说 Discuz 官方这个 Doc 是真的坑，要什么什么没有，全得自己找。

不过好在这次遇到的问题都并不算困难，而且很多都参考了以前的插件，还是很快就搞定了。

不过考虑到以后可能用到，还是先写一点记一下。

### Array 注入点与 $postlist

首先是每个 post 的注入点如何针对楼层内容进行处理。

Array 注入点本身其实没什么好说的，是几就显示在几的位置。

麻烦的是 Doc 里没提到的这个 $postlist 变量，这个变量存储了当前页面所有 post 的数据。与这个配合，才能更好的针对每一层的内容（主要是pid）进行处理。

``` PHP
function viewthread_postaction_output()
{
    global $postlist;
    $output = [];
    foreach ($postlist as $pid => $data) {
        $output[] = $pid;
    }
    return $output;
}
```

### 关于发帖后获取当前帖信息

这个也非常坑，搜索了很久，最后还是被 luckypost 救了。

不得不说，DZ 对很多东西进行了魔改，比如说变量 $_G，前端里的 js（包括但不限于选择器、ajax等），还有就是这次的 $_GET。

本来以为发帖之后获取当前帖子的信息这一点要修改 DZ 的原始文件，没想到并不用。

但搜到的 [v2ex 帖子](https://www.v2ex.com/t/274551)写的非常含糊。

不过最后还是被我给找到了正确的使用方法，如下：

``` PHP
// $value = '{"param":["post_newthread_succeed","forum.php?mod=viewthread&tid=1&extra=",{"fid":"1","tid":1,"pid":1,"coverimg":"","sechash":""},[],0]}';
function post_<pluginId>_message($value)
{
    // 这里就可以做发帖之后的流程了，$value 里是能拿到的一些信息。
    // debug() 函数可以停止跳转并直接输出，配合 var_dump() 很舒服。
}
```

## 结语

仅作为备忘。没什么其他好说的了。
