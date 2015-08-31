---
layout: post
title: "文件系统 file_system"
description: "狭义的讲，文件系统不过是用来管理磁盘数据的方式"
category: os
tags: [os]
---
{% include JB/setup %}

人们对数据的持久保存有需求，于是一般的数据，程序都存储在诸如磁盘等介质之上．
广义的讲，文件可以指任何序列化的二进制数据，但是一般的讨论都会限制在磁盘文件的讨论之上．

#### 文件描述符

所有与文件相关的系统调用都涉及到*文件描述的概念*.
*文件描述符*这个抽象的概念，指能够最终索引到文件数据，并且提供一定文件控制的一个你内存数据结构．当然我们的重点还是对文件数据的索引．
实现中，我们一般将它设置为inode位图的索引，即通过它可以直接索引到这个文件对应的inode_entry.

#### 文件的状态

同一时刻可能存在多个进程同时处理一个文件，处理的进度可以使用文件的偏移量来指定．一些相关的进程希望共享偏移量，而其他无关的进程希望拥有独立的但是对于同一文件的偏移量值．

系统保存这些偏移量，具体而言，就是使用系统*系统打开文件表*，*用户打开文件表*来维护文件偏移量．如图：
![process-system-open-file]({{site.baseurl}}/assets/pic/System-open-file-table.jpg)

#### 重定向和管道
```
ls > a.txt
```
在shell中的实现大致如下：
{% highlight c %}
if((pid = fork()) == 0) {
	// child
    close(STDOUT_FILENO);
    open("a.txt");
    exec("ls");
}
else {
	// father
    waitpid(pid);
}
{% endhighlight %}

在子进程，标准输出文件被关闭，当打开a.txt时候，标准输出的描述符被赋予了a.txt．在ls进程中，往标准输出写数据就相当于往a.txt写数据，而它却毫不知情．

管道，是内存中的一段区域，对它的操作需要遵循先进先出的规则．创建管道的系统调用为：

```
int pipe(int pipefd[2]);
```

pipefd[0]为读端，pipefd[1]为写端．
linux的终端下的管道使用：

```
command1 | command2
```

这里要求command1会往标准输出写数据，command2会从标准输入读数据.
那么实现方式就是创建一个匿名管道，将command1的标准输出重定向到pipefd[1],将command2的标准输入重定向到pipefd[0]

#### 文件块的管理

ext2使用inode来组织文件块，如下图:

<img style="float: left;clear:both" src="{{site.baseurl}}/assets/pic/ext2-inode.gif">
<div style="clear:both"/>
相应的结构为：

{% highlight c %}
typedef uint32_t block_t;
struct iNode_entry　｛
	char filename[32];
    size_t size;
    int type;
    int dev_id;
    block_t index[15];
};
{% endhighlight %}

很显然，只要拿到一个文件的inode数据结构，就能得到文件的所有数据．
要获取某个偏移量上的数据，就跟随者index来找，如果是直接映射，那么载入相应的文件块，找到对应的偏移数据；如果是简单映射，文件块上存储的是索引块，解释其中的索引块来获得直接索引．
在内存中的布局如下：
<img style="float: left;" src="{{site.baseurl}}/assets/pic/disk-layout.png">
<div style="clear:both"/>

#### 目录的管理

目录只是一种特殊的文件，文件的每个块记录着结构化的数据，定义如下：

{% highlight c %}
typedef int inode_t;
struct dir_entry {
	char filename[32];
    inode_t inode;
}
{% endhighlight %}

目录提供了文件名到inode节点的映射，所有文件被组织成树状结构，树根目录被称为根目录，记录在内核中，所有的进程都能访问，一般使用第一个inode结构存储．

每个进程还拥有*工作目录*

当创建一个目录时，要自动为里面添加两个目录项，分别命名为"."与".."，分别代表当前目录和父目录．

#### 硬链接和删除文件

硬链接指在一个目录下创建一个目录项，其文件名与参数文件名不同，但是指向同一个inode.

删除文件涉及到删除制定目录下的目录项，同时释放目标文件inode节点中涉及到的所有数据（包括inode数据结构和所有占用的物理文件块）．
