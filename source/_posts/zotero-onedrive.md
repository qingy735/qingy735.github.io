---
title: Zotero+OneDrive实现论文自由
tags:
  - Zotero
  - OneDrive
categories: 论文
abbrlink: 36836
date: 2022-11-07 16:52:48
---



> 本篇文章主要介绍如何在Zotero中使用OneDrive实现备份



## Zotero

> Zotero下载地址[Zotero | Downloads](https://www.zotero.org/download/)

Zotero作为一款非常好用的论文阅读管理器，本人也是一直在使用。但是由于工作电脑和个人笔记本电脑都需要阅读文献，也就衍生出了同步和备份问题，虽然Zotero自带300M空间，但是随着论文下载阅读的增多，突然有一天不能继续备份同步了。所以就发现了可以配合OneDrive来进行备份使用。

### Zotero网页插件

#### 下载

点击上面下载链接，安装浏览器插件即可实现论文自动添加

![image-20221107174231484](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221107174231484.png)

#### 使用说明：

以下载论文《Attention Is All You Need》为例，打开论文页面，会发现浏览器右上角部分会出现类似图标，点击即可选择保存到Zotero什么位置。同时，可以配合插件*Sci-Hub*进行使用，同步获取到论文pdf文件。

![image-20221107174834763](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221107174834763.png)



### Zotero同步与备份

首先可以选择修改Zotero文件存放位置：**编辑**->**首选项**->**高级**->**文件与文件夹**，将下图圈出的部分更改为自己想要存储的位置。

![image-20221107175716986](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221107175716986.png)



然后点击**同步**，自己注册账号然后将数据同步全部勾选，数据同步指的是论文中的注释、链接、标签等（除了附件之外的所有内容），同时数据同步是免费和无限制的。当然你也可以将文件同步全部勾选，但是当云端300M空间用完了就没办法继续使用了。

![image-20221107180229339](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221107180229339.png)





## OneDrive



OneDrive是微软公司推出的用来存储文件和照片的云存储空间应用，一般的，个人用户都有5G免费空间可供使用。又因为OneDrive是直接在windows中有存储映射，也就是将文件放入OneDrive文件夹中就可以实现上传备份。所以这里我们可以利用**软链接**的方式实现Zotero存储文件夹和OneDrive文件夹的连接。

### 建立软链接

一般的话Zotero的论文都存储在**Zotero/storage**文件夹中，首先将整个文件夹[storage]移动到OneDrive所在的文件夹中。然后**win+R**打开**cmd**，输入`mklink /j [storage原来路径] [OneDrive中storage路径]`,回车之后便创建了两个文件夹之间的软链接。切记：一定要先把Zotero中的storage文件夹剪切到OneDrive中，然后再输入上述命令，不然会报错文件夹已存在。创建成功后会发现之前的Zotero文件夹中会有一个新的storage文件夹（左下角有个箭头），类似下图这样，出现这样就代表成功了。

![image-20221107181751232](https://gitee.com/qingy735/blogimg/raw/master/img/image-20221107181751232.png)

### 扩容OneDrive

当然，如果你觉得5G内存还是不够你使用，也还有办法，只要你拥有教育邮箱，就可以先注册一个Office教育账号([学校和学生免费使用 Microsoft Office 365 | Microsoft 教育](https://www.microsoft.com/zh-cn/education/products/office))。账号注册完成后就会发现你拥有了一个具有1T的OneDrive账号（如果还不够...)



























