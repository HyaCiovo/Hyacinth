---
title: 哎？这列表里的图片怎么有些不见了
published: 2024-03-22
updated: 2024-03-22
description: '今天测试同学发现了我负责开发的一个组件有一个很奇怪的bug：列表渲染的组件有一部分顶部的 logo 图片不展示。🧐'
tags: [React,CSS]
category: '工作记录'
draft: false 
---


最近都是一些很简单的增删改查的小需求，没什么好特别记录的。不过今天测试同学发现了我负责开发的一个组件有一个很奇怪的bug：部分顶部 logo 图片不展示。

![img](../images/1744768409458.jpg)

第一反应是“甩锅”（bushi），“后端接口肯定有些偷懒没造数据！”

于是我 F12 打开 network 看了下接口，后端是正常返回了图片的 oss 地址的。（坏了，这下是我的锅了）

但是为什么有些会展示，有些不会展示呢？ 我又看了下接口里每个返回的 url ：
```json
logoUrl: "upload/2024/2/2/xxxxxxxx/xx.jpg"  // 正常展示
logoUrl: "upload/2024/3/1/xxxxxxx/xx(1).jpg"  //不展示
```
观察 element 结构和代码，发现我在这里是复制其他文件已经拼接好的 oss 图片结构，其使用的不是 img 标签，而是通过 div 标签设置`background-image`属性为拼接的 url 来展示图片

```tsx
<div className="bg-img h-[50px] w-[182px]"
style={{backgroundImage: `url('${baseUrl}${item.logoUrl}')` }}/>
```

奇怪的是列表里有一部分没有了`background-image`这个属性

![image-20250416100213922](./../images/image-20250416100213922.png)

既然有些图片可以展示，有些不可以，那么我拼接的方式应该是没问题的，应该是有些 url 格式比较“特殊”，导致`background-image`看不懂这个 url。于是我仔细检查没有展示出的图片的 url，发现他们都有一个相同的特征：含有`(` `)`括号！

百度了一下，我才知道`background-image`属性当 url 有括号时是有问题的，背景图不会展示出来。

而它的解决方法是在 url 外再包一层`''`。即：

```json
background-image: url(logoUrl)    // 👎
background-image: url('logoUrl')  // 👍
```

于是，我将代码修改为：

```tsx
<div className="bg-img h-[50px] w-[182px]"
style={{backgroundImage: `url('${baseUrl}${item.logoUrl}')` }}/>
```

图片便可以正常展示了。

其实，平时写死地址时都会加上`''`的，但是之前的开发是在内联样式中拼接的地址，导致没注意加上这个引号。而`background-image`又是可以处理这种不带括号的 url 的，之前的图片也没有带括号的地址，导致这个问题一直没有被发现。结果我这个偷懒倒霉蛋复制过来用给碰上了🤣。
