---
layout: post
title: "记一次简单的渗透测试"
categories: 渗透测试
tags: 漏洞 渗透测试 渗透
author: Red ink man
---



* content
{:toc}








## 测试地址

[http://www.xxhdfc.com](http://www.xxhdfc.com)

## 工具

啊d，明小子，御剑（手注截图没有保存）

## 正式步骤

手工判断可注入  
利用啊D sql注入检测，扫面注入点，检测表段，字段。可以获得加密的后台账号，密码  
![图片1.png](https://i.loli.net/2020/01/21/iuIvopHXQc7SYKn.png)

> 后台账号： admin  
> 后台密码： 7d923c064c0e9486

显然后台密码是加过密的，在线解密MD5，可知后台密码为xxhdfc.com  
利用御剑扫秒后台  
![图片2.png](https://i.loli.net/2020/01/21/RfbCxyLaOF8nYq4.png)  
扫到后台地址为http://www.xxhdfc.com/user/login.asp  
尝试登陆  
![图片3.png](https://i.loli.net/2020/01/21/PDJhTVd3lYfjvcG.png)  
登陆成功    
![图片4.png](https://i.loli.net/2020/01/21/gcuAXotawZV1flN.png)  
现已联系站长修复，修复后为下图所示：  
![图片5.png](https://i.loli.net/2020/01/21/YsaJtcK7dwMqC8o.png)  
建议：  
>    1. 所有的查询语句都使用数据库提供的参数化查询接口，参数化的语句使用参数而不是将用户输入变量嵌入到SQL语句中。当前几乎所有的数据库系统都提供了参数化SQL语句执行接口，使用此接口可以非常有效的防止SQL注入攻击  
>    2. 对进入数据库的特殊字符（\’”<>&\*;等）进行转义处理，或编码转换  
>    3. 严格限制变量类型，比如整型变量就采用intval()函数过滤，数据库中的存储字段必须对应为int型  
>    4. 数据长度应该严格规定，能在一定程度上防止比较长的SQL注入语句无法正确执行  
>    5. 网站每个数据层的编码统一，建议全部使用UTF-8编码，上下层编码不一致有可能导致一些过滤模型被绕过  
>    6. 严格限制网站用户的数据库的操作权限，给此用户提供仅仅能够满足其工作的权限，从而最大限度的减少注入攻击对数据库的危害  
>    7. 避免网站显示SQL错误信息，比如类型错误、字段不匹配等，防止攻击者利用这些错误信息进行一些判断  
>    8. 在网站发布之前建议使用一些专业的SQL注入检测工具进行检测，及时修补这些SQL注入漏洞  
>    9. 确认PHP配置文件中的magicquotesgpc选项保持开启  
