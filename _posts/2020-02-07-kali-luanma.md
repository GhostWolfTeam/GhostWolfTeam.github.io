---
layout: post
title:  "解决Kali Linux中文乱码"
categories: Kali Linux
tags: Kali Linux
author: 江居
---



* content
{:toc}








## 前言

新的Kali Linux 2020.1版本系统存在中文乱码的情况，让选择中文语言包的朋友们伤了脑筋，其实解决这个问题，重新安装中文字体就好了，接下来将简单教大家如何配置中文字体

## 开始

启动并登录Kali Linux 2020.1版本系统，打开终端，逐步输入执行下列命令：

> sudo apt-get update ：更新软件包下载地址  
> sudo apt-get install fonts-wqy-zenhei ：安装中文字体  

![图片1.png](https://i.loli.net/2020/02/07/dMpafKgw23oD6tv.png)

如果需要复制粘贴命令，请在输入的过程中右键打开选项卡如图：

![图片2.png](https://i.loli.net/2020/02/07/9GdRyLZ5XUs7YNx.png)

（红色箭头所标识的选项，则第二个为粘贴功能选项，也可以在复制到终端输入处利用<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>V</kbd>进行粘贴）

> 注意：如果字体安装失败，请更换更新源重新配置Kali  
如何配置KaLi可以访问 Snow狼的文章《Kali Linux萌新特别篇》：   
转入https://blog.51cto.com/superwolf/2381199  
（配置安装时应注意网络环境质量）  

在完成以上步骤，安装好中文字体时，接下来使用locales 来进行设置字体：
（执行下条命令时请检查locales是否安装，检查无误后请输入执行下列命令）

命令：

```bash
sudo dpkg-reconfigure locales
```

打开字符编码配置图形化界面，用键盘<kbd>↓</kbd>进行列表浏览，找到en_US.UTF-8、zh_CN.GBK、zh_CN.UTF-8三种字符编码格式并选择（按下空格，选项出现星号“\*”则为已选择）回车确认，到达下一配置页面（如下已经是汉化好的情况）：  

![图片3.png](https://i.loli.net/2020/02/07/qpSdELaDRPco9Wr.png)

选好后回车进入下一个配置页面，再次利用键盘<kbd>↓</kbd>键，然后用空格选择zh_CN.UTF-8，并回车完成配置，界面如下：

![图片4.png](https://i.loli.net/2020/02/07/ToGZvEDILPMU6yK.png)

完成配置后，自动回到终端，输入命令：reboot 重启系统，重启后就可以看到登陆页面显示为中文了，如下图：

![图片5.png](https://i.loli.net/2020/02/07/L1ugyBoYalNMSDp.png)

重启后的效果图，解决问题：

![图片6.png](https://i.loli.net/2020/02/07/HQdYKX59er16Etz.png)
