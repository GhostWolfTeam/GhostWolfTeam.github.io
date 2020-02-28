---
layout: post
title: "kernel内核模块开发"
categories: 开发
tags: C 开发
author: Alcatraz
---



* content
{:toc}








#  kernel 内核开发——模块

> 源于一次老师留的作业，参考书籍是操作系统原理（龙书）第七版和英文的龙书第九版
>
> 这次的实验在Ubuntu18.04_x64实体机上进行，内核版本为4.15.0-72-generic
>
> ——Edit by Alcatraz

###  这一次开发用到的基础知识

​	首先我们要知道，Linux内核是一个极其庞大的结构，我们只能窥探其中一小部分，只一次实验，我们通过内核模块的构造，完成一个输出进程树的工作（啥用没有，就是纯粹为了给老师装个B，因为都是没讲过的知识）

​	好了，有了大概的方向，我们就要开始着手准备自己编程需要的知识了。首先是，要编写一个内核模块，那就要知道内核模块的基本构造，其次，要通过系统调用来完成我们进程树的绘制，所以我们要理解一下下什么是系统调用以及怎么调用，接着，我们要完成进程树的绘制，所以我们要了解什么是task_struck结构，要明白怎么把我们遍历的进程数据通过raw_copy_to_user()内置函数从内核态的数据空间拷贝到用户态空间，最后，就是使用一些别的知识把这个玩意写到一个txt中，便于看。

​	OK，我们的思路已经很清晰了，开始一步一步来。

###  第一步是内核模块的基本构造和编译调用

​	这一部分，我参考了龙书的第九版第二章的课后习题。通过看书，我们可以了解到：首先一个模块调用要有调用和退出，这才可以保证模块的正常运作。

> 这个结构就是我们开发的模板，我们新增的功能就要加到这里，我们把这个模块命名为mrx，于是这个主体代码为mrx.c

​	结构如下：

```c
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
/* This function is called when the module is loaded. */
int simple init(void)
{
printk(KERN INFO "Loading Module\n");
return 0;
}
/* This function is called when the module is removed. */
void simple exit(void)
{
printk(KERN INFO "Removing Module\n");
}

/* Macros for registering module entry and exit points. */
module init(simple init);
module exit(simple exit);

MODULE LICENSE("GPL");
MODULE DESCRIPTION("Simple Module");
MODULE AUTHOR("SGG");
```

​	So，我门看一下，有一个init函数是为了初始模块，有一个exit函数，是为了结束模块，这样有始有终才算完整。最后三行是一种常规的声明，首先license是一定要有的，表示支持GPL规范，你就可以用自己写的模块了，其他的，没啥必要倒是。

​	<b><font color=#FF0000>可以看到，这是个C语言编写的程序，含有大量的内核库的调用，所以为了生成一个模块调用，我们需要使用且仅能使用Makefile编译！！！ </font></b>

​	所以我们要了解一下，这个Makefile到底怎么写，查书，发现有一个固定的模式（其实我们应该学习一下makefile语法的，奈何我太懒，就没学，所以一直用模板，不过大概规律很好掌握的）

​	很简洁，结构如下：

```c
obj-m += mrx.o  //此处的mrx.o对应上面的mrx.c
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

​	嗯，咋样，很简单把，要是文件换名字了，那就改个名字，照样用，简单吧。

​	有了mrx.c和Makefile，这时候就可以编译运行了。在命令台中``` make``` 就完事了，然后你能在目录下找到一个mrx.ko的文件，嗯，这就是我们编译出来的内核模块了。

​	我们可以先安装一下看看有什么效果哈，提示，可能后面你自己添加内容之后会出现模块出错导致卸载不掉的情况，也没事，重启可解。

> 这里，推荐在安装之前先执行一下``` dmesg -C``` 命令，清除一下内核日志，便于查看结果。

​	执行```sudo insmod mrx.ko``` 安装模块，看不到任何输出的，所以我们执行``` dmesg``` 可以看到``` Loading Module``` 出现，证明安装成功了。执行``` sudo lsmod``` 可以看到：

![1.png](https://i.loli.net/2020/02/28/IGvOnbiEYaloHuw.png)

​	这里看两个，一是我们的模块名字出现，另一个是没有被占用，所以是0，这样才可以正常的移除。执行``` sudo rmmod mrx.ko``` 移除模块，同样执行一下``` dmesg``` 和``` lsmod ``` 查看结果。

###  第二步是增加系统调用——修改sys_call_table

​	首先我们要由sys_call_table的知识，然后还有要知道每一个系统调用都对应自己独一无二的系统调用号，我们查一下哈，理论上讲，对于64位机器，0到500多号调用都是被占用的，这里呢，我们随便挑一个覆盖掉，当然你在后边加也行。我挑的223号，跟网上的一样。

这个系统调用表在这里：

```bash
/usr/src/linux-headers-4.15.0-72-generic/arch/x86/include/generated/asm/syscalls_64.h
```  

![2.png](https://i.loli.net/2020/02/28/2ImcT6UdRpleOGw.png)

然后呢，我们还要查一下自己的系统调用表地址，不然函数指针指不过去，也没法调用，执行命令：``` sudo cat /proc/kallsyms | grep sys_call_table``` ，可以看到：

![3.png](https://i.loli.net/2020/02/28/1QqLMKgzu6GNd5A.png)

把这个地址记住，一会要用。

然后我们可以看到R，这玩意就是告诉我们，我们没有权限操作sys_call_table，所以呢，我们也别想往里加我们自己的系统调用，尽管我已经使用root了，也没用。

所以，这里我们要知道，cr0寄存器的第十六位是这个的写保护位，把第十六位改掉，我们就可以写入啦，然后呢加了以后我们再把它复原，哈哈哈哈，神不知鬼不觉。

这里就是硬核的计算机组成原理知识了，不会的自己学哈，代码实现如下：

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/unistd.h>
#include <asm/uaccess.h>
#include <linux/sched.h>

#define my_syscall_num 223
#define sys_call_table_address 0xffffffff94800240

static int counter = 0;
struct process{
	int pid;
	int depth;
};

struct process a[512];

unsigned int clear_and_return_cr0(void);
void setback_cr0(unsigned int val);
asmlinkage long sys_mycall(char __user *buf);
int orig_cr0;
unsigned long *sys_call_table = 0;
static int (*anything_saved)(void);


unsigned int clear_cr0(void);
void setback_cr0(int val);

unsigned int clear_cr0(void){
	unsigned int cr0=0;
	unsigned int ret;
	asm volatile("movq %%cr0, %%rax": "=a"(cr0));
	ret=cr0;
	cr0&=0xfffffffffffeffff;
	asm volatile("movq %%rax, %%cr0"::"a"(cr0));
	return ret;
}

void setback_cr0(int val){
	asm volatile("movq %%eax, %%cr0"::"a"(val));
}
```

有了这个最最关键的模块，我们就可以操作系统最底层的东西了。

###  第二步是增加系统调用——task_struct结构体

​	接着就是我们给自己的模块增加一个能用用户态操作的系统调用了，所以呢，我们增加自己的函数，processtree。

它的作用是调用task_struct结构体，用系统定义好的结构遍历并寻找自己想要的进程信息。

首先，我们要了解一下task_struct结构体，嗯，这个我实在是不想再看然后粘贴一遍了，你直接百度：Linux内核 task_struct结构体，然后我感觉写的比较直观的是简书的一篇，虽然他不是原创。

了解了这个东西，就好办了，我上代码你就看得懂了：

``` c
void processtree(struct task_struct * p,int b)
{
	struct list_head * l;
	a[counter].pid = p -> pid;
	a[counter].depth = b;
	counter ++;
	for(l = p -> children.next; l != &(p->children); l = l->next)
	{
		struct task_struct *t = list_entry(l,struct task_struct,sibling);
		processtree(t,b+1);
	}
}
```

这个就是我们一会执行调用的时候的processtree的动作。

然后呢，我们增加自己的系统调用，让系统调用调用processtree函数去：

``` c
asmlinkage long sys_mycall(char __user * buf)
{
    int b = 0;
	struct task_struct * p;
	printk("This is a test hacking syscall:) My pid is %d, gona kill me?\n",current->pid);
	for(p = current; p->pid !=0; p = p->parent ){
		processtree(p,b);
	}

    //下面这个函数是为了把内核态数据拷贝到用户态空间，因为用户态无法直接读取内核态空间按的内容！！！
	if(raw_copy_to_user((struct process *)buf,a,512*sizeof(struct process)))
		return -EFAULT;
	else
		return sizeof(a);
}
```

然后呢，我们注意，这里还没有替换系统的原始调用，所以呢，我们放到init中做，恢复的话，我们就放到exit部分做：

```c
static int __init init_addsyscall(void)
{
	printk(KERN_INFO "syscall initializing...:)\n");
	sys_call_table = (unsigned long *)sys_call_table_address;
	anything_saved = (int(*)(void)) (sys_call_table[my_syscall_num]);//把系统原来的函数指针保存，方便恢复
	orig_cr0 = clear_and_return_cr0(); //清理cr0
	sys_call_table[my_syscall_num]= (unsigned long)&sys_mycall;//换成自己的系统调用函数指针地址
	setback_cr0(orig_cr0);//恢复cr0
	printk(KERN_INFO "syscall initialized...:)\n");
	return 0;
}

static void __exit exit_addsyscall(void)
{
	printk(KERN_INFO "syscall exiting...\n");
	orig_cr0 = clear_and_return_cr0();
	sys_call_table[my_syscall_num]= (unsigned long)anything_saved;//恢复原先的系统调用函数
	setback_cr0(orig_cr0);
	printk(KERN_INFO "syscall exited...\n");
}
```

至此，我们主体编程完成，下一步是，写个Makefile用。

如下：没区别吧！

```c
obj-m += mrx.o  
all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

```

嘿嘿嘿，其实一会你make的时候你会发现，编译会报错应该，大概是说什么一个h文件里有什么结构没声明，这里我的解决办法是：\<font color="FF0000"> 找到那个文件，然后进去，用sudo修改一下，加一条include为：``` include <linux/sched.h> ``` ，然后你保存退出回来再编译就OK了。</font>

###  最后，写一个test测试函数，调用我们自己的系统调用

代码如下，贼简单：

``` c
#include <linux/unistd.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <stdio.h>
#include <linux/kernel.h>
#include <unistd.h>
struct process
{
	int pid;
	int depth;
};

struct process a[512];

int main()
{
	// long a;
 //    if(a==0xffffffff94800240){
 //        print("Please run command (sudo cat /proc/kallsyms | grep sys_call_table) to get your unique address!!\n");
 //        print("Notice that your unqiue address is just the first result of the command.\n");
 //        print("And don't forget to renew the address in pid.c, just repalce the sys_call_table_address with your own.\n");
 //        print("Good luck!!--mrx");
 //        return 0;
 //    }
	int i,j;
 	FILE * f;
 	f=fopen("proc_tree.txt","a+");
	printf("the result is:%ld\n",syscall(223,&a));

	for(i = 0; i < 512; i++)
	{
		for(j = 0; j < a[i].depth; j++)
			fprintf(f,"|-");
		fprintf(f,"%d\n",a[i].pid);

		if(a[i+1].pid == 0)
			break;
	}
	return 0;
}
```

然后我们一会在目录下就会看到一个proc_tree.txt，它就是我们记录结果的地方。

至此，结束调用模块，查看``` dmesg``` ，然后实验圆满结束！
