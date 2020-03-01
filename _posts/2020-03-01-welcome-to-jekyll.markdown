---
layout: post
title:  "git 代理設置"
date:   2020-03-01 16:57:59 +0800
tag: [git]
---

搭建这个Jekyll博客花了一个下午的时间 :sweat_smile:

最大的问题还是爬不动墙，每次看着不停转圈圈的chrome和一动不动的命令行，就十分的难受。

第一篇个人博客，先庆祝一下！！ :star2: :collision:

这篇博客先把经常用到的代理设置放进来吧，免得用到的时候又是查查查

### git 
[参考][git]
{% highlight console %}
--设置代理
git config --global http.proxy 'socks5://127.0.0.1:1080' -- 替换成你自己的代理服务
git config --global https.proxy 'socks5://127.0.0.1:1080'

--取消代理
git config --global --unset http.proxy 
git config --global --unset https.proxy

{% endhighlight %}