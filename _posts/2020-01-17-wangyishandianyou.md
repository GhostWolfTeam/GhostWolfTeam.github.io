---
layout: post
title:  "关于网易闪电邮的逆向"
categories: 逆向
tags: 逆向
author: 红龙HL
---



* content
{:toc}








## 前言

最近有一个需求，需要批量导入邮箱，并且自动接收相关邮件，提取邮件中的需要数据另存为文本  
采用了自写socks5代理服务器代理发现网易闪电邮就是连接不上自个服务器，测试中代理服务器一切正常，所以pass这个思路，采用逆向的方式实现一系列功能

## 开始操作

客户端主进程为：  
![图片1](https://i.loli.net/2020/01/17/2jM7oW4FDLC9NTr.png)  
但是使用OD直接打开会弹出提示  
![图片2](https://i.loli.net/2020/01/17/SZ49nolFbmGPOW3.png)  
![图片3](https://i.loli.net/2020/01/17/K6QvzRdxhtP1GCi.png)  
OD打开Start发现这玩意儿基本上就只是去启动进程FlashMail.exe，所以为了方便调试，首先去掉这个提示。过程不过详解，断点MessageBox马上就找到弹窗所在点然后跳过就可以  
现在就可以使用OD直接打开主程序FlashMail.exe  
经过简单分析发现主模块为Mail.dll,加载模块Mail.dll  
我们需要在新邮件到达的时候下断  
具体断点查找方法通过测试接收邮件得知在具体接收到邮件时会在该邮件用户目录写下日志（文件为in.mbox），  
所以在WriteFile下断，就截获邮件内容  
具体特征位置为  
![图片4](https://i.loli.net/2020/01/17/SqmRjIQ7sAoChHe.png)  
在接收到邮件时会改写多个文件，需要一些判断，具体是当断点时 EBX=9 就是我们需要

> 接收邮箱：ESP - 2A4  
> 邮箱密码：ESP - 29C (BASE64加密)  
> 邮件发送邮箱：ESP - 4E0 (包含邮箱昵称)  
> 邮件标题：ESP - 4DC  
> 邮件内容：ESP - 4A0  

对于邮件内容的解析：  
（我仅记录我需要的）  
![图片5](https://i.loli.net/2020/01/17/2b3ker1EaxoJFLq.png)  
查找字符串：  
***
Content-Type: image/png; name="icon_warm.png"  
Mime-Version: 1.0     （+4字节跳过换行）  
***
结束字符串：（-2字节跳过换行）--  
后期处理：去除截获字符串中所有的{0XD,0XA}(换行符)  
BASE64解密得到最终需要字符串  
到此，邮件接收过程分析过程结束

## 批量导入

下面是批量导入账号功能，我选择使用自写他的用户目录文件达到批量导入的效果  
通过查找查看用户目录下的用户基本信息分析得到其中有一个.cfg文件实际是SQLITE3数据库文件，打开查看就能看到里面记录了基本信息，所以构造出这个文件就可以达到添加账户效果  
其中大部分内容都没有问题，主要在于password一栏为加密  
![图片6](https://i.loli.net/2020/01/17/jCaz4fGkXIZADgy.png)  
乍一看为BASE64加密，实际上却解不开  
目测问题出在下面调用Common.dll中的CommCalc方法上，  
![图片7](https://i.loli.net/2020/01/17/j52WAGgeFi4JnPS.png)  
此时需要还原代码了，果断打开IDA，加载上common.dll，通过查找出CommCalc发现，果不其然对密码进行了二次加密。  
![图片8](https://i.loli.net/2020/01/17/bvmN3h2AOajlPzk.png)  
。。。。。。  
![图片9](https://i.loli.net/2020/01/17/AaFQtv9PDTLZsHk.png)  
变形后的字符串Base64加密后输出保存  
IDA中看出是AES加密，并且发现AES密钥是固定的（那你AES意义何在）  
```bash
KEY：0xF7,0x1E,0x19,0x18,0xFA,0x2E,0x26,0x25,0x10,0x4D,0x40,0x3F,0x10,0x8B,0x73,0x74

int CommCalc(char *a1, int a2)
{
	int v2; // esi
	unsigned int v3; // esi
	unsigned __int8 *v4; // ebx
	_DWORD *v5; // edi
	signed int v6; // ebx
	char *v7; // esi
	int v8; // edx
	char v9; // cl
	char *v10; // esi
	int v11; // edx
	char v12; // cl
	char v13; // cl
	char v14; // dl
	int v16; // [esp+14h] [ebp-3E8h]
	unsigned __int8 *v17; // [esp+18h] [ebp-3E4h]
	unsigned __int8 *v18; // [esp+1Ch] [ebp-3E0h]
	int v20; // [esp+25h] [ebp-3D7h]
	int v21; // [esp+29h] [ebp-3D3h]
	int v22; // [esp+2Dh] [ebp-3CFh]
	int v23; // [esp+31h] [ebp-3CBh]
	string v24; // [esp+38h] [ebp-3C4h]
	char v25; // [esp+54h] [ebp-3A8h]
	int v26; // [esp+58h] [ebp-3A4h]
	char v27[32] = {0}; // [esp+5Ch] [ebp-3A0h]
	int v28; // [esp+A0h] [ebp-35Ch]
	char v29[128]; // [esp+A4h] [ebp-358h]
	char v30[128]; // [esp+194h] [ebp-268h]
	char v31[32]; // [esp+284h] [ebp-178h]
	char v32[32] = {0}; // [esp+2A4h] [ebp-158h]
	int v33; // [esp+3F8h] [ebp-4h]

	v2 = a1[5];
	v16 = a1[5];
	if (v2 % 16 > 0)
	{
		v2 += 16 - v2 % 16;
		v16 = v2;
	}
	v3 = v2 + 1;
	v4 = (unsigned __int8 *)operator new[](v3);
	v18 = v4;
	v17 = (unsigned __int8 *)operator new[](v3);
	memset(v4, 0, v3);
	memset(v17, 0, v3);
	if (a1[6] < 0x10u)
		v5 = (_DWORD*)a1 + 1;
	else
		v5 = (_DWORD *)&a1[1];
	memcpy(v4, v5, a1[5]);
	sub_10002AC0(v32);
	v20 = 0;
	v21 = 0;
	v22 = 0;
	v23 = 0;
	v25 = 0;
	v26 = 128;
	strncpy(v27, v32, 0x20u);
	v6 = 0;
	v7 = v27;
	do
	{
		v8 = *v7;
		v9 = v8 - 48;
		v10 = v7 + 1;
		if ((unsigned int)(v8 - '0') > 9)
		{
			if ((unsigned int)(v8 - 'a') > 5)
			{
				if ((unsigned int)(v8 - 'A') > 5)
					goto LABEL_21;
				v9 = v8 - 55;
			}
			else
			{
				v9 = v8 - 87;
			}
		}
		v11 = *v10;
		v12 = 16 * v9;
		v7 = v10 + 1;
		if ((unsigned int)(v11 - 48) > 9)
		{
			if ((unsigned int)(v11 - 97) > 5)
			{
				if ((unsigned int)(v11 - 65) > 5)
					goto LABEL_21;
				v14 = v11 - 55;
			}
			else
			{
				v14 = v11 - 87;
			}
			v13 = v14 ^ v12;
		}
		else
		{
			v13 = (v11 - 48) ^ v12;
		}
		v31[v6++] = v13;
	} while (v6 < 16);
LABEL_21:
	//AES加密--128加密
	//0xF7,0x1E,0x19,0x18,0xFA,0x2E,0x26,0x25,0x10,0x4D,0x40,0x3F,0x10,0x8B,0x73,0x74
 。。。
 base64_encode();
 得出SQLITE32中的密钥值，操作下就可以另类批量导入了
 }


```

关于自己构建用户目录问题，如下图  
![图片10](https://i.loli.net/2020/01/17/Nni7pWh5IU24qac.png)  
带indx的文件必须存在，但是可以是空文件，仅仅识别文件名而已  
主要构建cfg文件，账号密码对上即可  
接下来就是写代码HOOK的过程了，告一段落  
