---
title: Hexo博客备份
tags:
  - backup
  - hexo
categories:
  - Blog
abbrlink: 20231
date: 2022-08-09 16:23:54
---



## Hexo博客的备份及恢复

### 前提 and 机制

Hexo能够备份的前提是你已经初始化好了需要备份的博客（各种需要的环境配置）

Hexo每次执行`hexo d`其实是将hexo编译后的文件上传部署到gitee或者github，用来生成网页，不包含博客源文件。也就是上传本地目录生成的

`.deploy_git`文件夹里的内容，其他内容包括`source`、`主题文件`、`配置文件`均没有上传到gitee上，所以可以将这部分内容上传到项目的另一个分支中

![image-20220809163025528](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220809163025528.png)



### 备份博客

#### 创建新分支

因为github速度问题这里已gitee为例。首先在原有的master分支基础上新建分支`sources`，并将其设置为默认分支（方便后续备份上传）

![image-20220809163515897](https://gitee.com/qingy735/blogimg/raw/master/img/image-20220809163515897.png)