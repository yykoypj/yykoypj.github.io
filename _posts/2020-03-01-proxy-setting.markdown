---
layout: post
title:  "各种代理設置"
date:   2020-03-01 22:37:59 +0800
tag: [proxy, git, npm]
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

--查看代理：
git config --global --get http.proxy
git config --global --get https.proxy

{% endhighlight %}

### npm 
{% highlight console %}
--查看当前所用的源
npm config get registry

-- 设置为淘宝源
npm config set registry http://registry.npm.taobao.org/

-- 设置回官方源(发布自己的包时需先设置回官方源)
npm config set registry https://registry.npmjs.org/

{% endhighlight %}

[git]: https://gist.github.com/laispace/666dd7b27e9116faece6
