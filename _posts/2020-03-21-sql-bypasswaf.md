---
layout: post
title:  "SQL绕过waf"
categories: 漏洞
tags: 渗透测试
author: Sincere
---



* content
{:toc}








## 准备

本来想写怎么绕过阿里的waf,没借到服务器(因为贫穷没钱买)，只能本地搭建一个360主机卫士演示，但是思路是差不多的。

1.我用的本地搭建的sqli，先用burp抓个包(具体怎么抓包就不演示了，应该都会)

![1.PNG](https://i.loli.net/2020/03/21/daBRbrFHT2NXjen.png)

2.把抓到的包转到repeater里方便我们看返回的内容

![2.PNG](https://i.loli.net/2020/03/21/rmYkvQntlz2N57b.png)

3.我们尝试一下注入，’union select 1,2# 可以发现被拦截了。

![3.PNG](https://i.loli.net/2020/03/21/aFMzAxSoZIWmet2.png)

4.可以尝试一下内联注入，/\*!select*/  
原理：/\*!select*/等于select  
可以看到并没有被拦截，我之前试阿里的时候这样是被拦截的，需要在加一些数字  
/\*!50000select*/,这个数字是有范围的，比如你mysql是5.12.34，这个数字就不能大于他版本号乘以1万，如果还拦截可以添加空格/\*%0a!50000select*/，代替空格有很多，可以多做尝试  

![4.PNG](https://i.loli.net/2020/03/21/oQPCMkSmOajUZvD.png)

5.用-错回显位看一下，发现也是可以的，接下来就进行数据库表读取，这里就不演示了，大概思路就是这样。这只是其中的一种绕过方法，像修改形式传输数据，切割形式传输等等，后期能找到阿里的我会具体演示(360主机绕过比较容易点毕竟都停止更新了)。

![5.PNG](https://i.loli.net/2020/03/21/1XvRD6ZwtpQjn8a.png)

6.还可以用sqlmap的tamper脚本进行绕过，sqlmap自带的的tamper绕过waf脚本很多，其中大部分脚本主要用正则模块替换攻击载荷字符编码的方式进行绕过的。

![6.PNG](https://i.loli.net/2020/03/21/GFYfM6d9bhJ57Qr.png)

7.tamper脚本的格式很简单，大概结构就是priority变量定义和dependencies,tamper函数定义，def tamper(payload,\**kwargs)函数接收payload和**kwargs返回一个payload。(稍微有点python基础应该都可以看懂)

![7.PNG](https://i.loli.net/2020/03/21/JZcD7GXENwyAvL6.png)

8.网上对sqlmap的每个tamper脚本的介绍有很多(毕竟有59个)，感兴趣的可以去看看。
