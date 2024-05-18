---
title: flex布局初步认识
tags:
  - flex
  - 布局
categories: 前端
abbrlink: 37862
date: 2022-02-23 15:00:49
---

## 关于flex布局的初步认识

### flex布局简单介绍

Flex 全称 Flexible box 布局模型，通常称为 flexbox 或 flex，也称为弹性盒子或弹性布局。一种比较高效率的 css3 布局方案。

既然是盒子，首先需要一个容器 container，然后是项目 item。容器包裹着项目，再通过配置不同的属性，实现各种各样的排列分布。

flex 有两根轴线，分别是主轴和交叉轴，主轴的方向由 `flex-diretion` 属性控制，交叉轴始终垂直于主轴。

容器为项目的分布提供的空间，轴线则控制项目的排列跟对齐的方向， flex 是一种一维的布局，一个容器只能是一行（左右）或者一列（上下），通过主轴控制项目排列的方向（上下/左右），交叉轴配合实现不同的对齐方式。有时候一行放不下，可以实现多行的布局，但是对于 flex 来说，新的一行就是一个新的独立的 flex 容器。作为对比的是另外一个二维布局 CSS Grid 布局，可以同时处理行和列上的布局。



### flex基本属性

flex由`flex-diretion`控制主轴方向，一般是横轴和纵轴。之后再由`justify-content`控制容器内部元素在主轴上的排列方式，主要分为center、space-around、space-between、flex-start、flex-end。而后用align-items控制元素在垂直主轴方向的排列样式。



### flex布局演示

```html
<view>示例一</view>
<view class="menu">
  <view class="item">
    <image src="image1.jpg"></image>
    <text>item1</text>
  </view>
  <view class="item">
    <image src="image2.jpg"></image>
    <text>item2</text>
  </view>
  <view class="item">
    <image src="image3.jpg"></image>
    <text>item3</text>
  </view>
  <view class="item">
    <image src="image4.jpg"></image>
    <text>item4</text>
  </view>
</view>
```

```css
.menu image {
  width: 150rpx;
  height: 120rpx;
}

.menu {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}

.menu .item {
  display: flex;
  flex-direction: column;
  align-items: center;
}
```

