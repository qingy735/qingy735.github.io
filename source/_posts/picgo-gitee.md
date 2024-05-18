---
title: PicGo + Gitee + Typora实现图床
tags:
  - PicGo
  - 图床
  - Typora
categories: Blog
abbrlink: 7360
date: 2022-02-26 19:18:27
---



## 使用PicGo + Gitee + Typora实现博客图床功能

### 1、Gitee账户配置

#### 新建仓库

![image-20220226192440682](https://gitee.com/qingy735/blogimg/raw/master/img/202202261924052.png)

> 注意修改为`开源`



#### 私人令牌token配置获取

- 找到`设置`->`安全设置`->`私人令牌`，点击生成新令牌，按照下图勾选后提交并保存token

  ![](https://gitee.com/qingy735/blogimg/raw/master/img/202202261929035.png)

到此gitee部分配置完成~



### 2、PicGo配置

#### 下载PicGo

点击此[下载链接](https://github.com/Molunerfinn/PicGo/releases)进行下载

#### 插件安装

![](https://gitee.com/qingy735/blogimg/raw/master/img/202202261935142.png)

#### 插件配置

![](https://gitee.com/qingy735/blogimg/raw/master/img/202202261936660.png)

#### 上传测试

![](https://gitee.com/qingy735/blogimg/raw/master/img/202202261937942.png)

上传成功后可以在相册看到

![image-20220226193838206](https://gitee.com/qingy735/blogimg/raw/master/img/202202261938285.png)

#### 设置快捷上传按键（根据个人习惯）

![image-20220226193943129](https://gitee.com/qingy735/blogimg/raw/master/img/202202261939201.png)

这样可以直接截图或复制网图后直接上传到gitee然后会返回粘贴板一个Markdown的图片链接形式



### 3、Typora配置

#### 配置图像设置

`文件`->`偏好设置`->`图像`

![image-20220226194231638](https://gitee.com/qingy735/blogimg/raw/master/img/202202261942775.png)

根据上图这样配置后点击验证图片上传选项测试是否成功

![image-20220226194312798](https://gitee.com/qingy735/blogimg/raw/master/img/202202261943926.png)

大功告成！
