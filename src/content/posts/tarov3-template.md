---
title: taro3.x + tailwindcss + zustand 小程序开发
published: 2024-11-11
updated: 2025-03-19
description: '新增支持支付宝小程序；新增埋点统计集成。近期有微信小程序开发需求，主管让我们进行技术选型、编写开发指引和规范文档。这里记录一下我们的技术选型过程，并分享一个模板项目供开发朋友们参考。'
tags: [Taro,React]
category: 'OpenSource'
draft: false 
---


### 代码仓库

大致框架代码可以查看 [HyaCiovo/taro-miniapp-template: 微信/支付宝小程序模板代码](https://github.com/HyaCiovo/taro-miniapp-template)。欢迎各位 **JY** 点点 **star**✨ 、提提 **pr**💫！

> **更新日志**
>
> *   2024/11/11 ： 兼容支持`支付宝`小程序；
>
> *   2024/11/12 ： 集成`友盟+`埋点统计SDK、小程序更新管理；
>
> *   2024/11/14 ： 在`feature/frame-preact`分支尝试使用`preact`减小打包体积；
>
> *   2025/3/19  ： 生产项目踩坑，更新`antmjs/vantui`到最新`^3.6.2`版本解决部分组件无法滚动页面的问题。

> **写在前面**
>
> 本项目没有适配多端，只是为了微信小程序开发搭建的模板项目。
>
> 对其他小程序的适配可以自行比较官方示例仓库添加依赖和代码，除了`H5`以外的适配都比较简单。

> **踩坑记录**
>
> *   `@antmjs/vantui`的`Swiper`组件在只有两个`Item`时，手动拖拽的动画交互很奇怪，看了源代码感觉如果要修复需要整个重构。建议优先使用`Taro`框架自带的轮播图组件。

### 背景

近期有微信小程序开发需求，主管让我们进行技术选型、编写开发指引和规范文档。这里记录一下我们的技术选型过程，并分享一个空的只包含主要依赖工具的模板项目供开发朋友们参考。

### 技术选型

|                         框架                        |                           UI库                           |                                          样式方案                                         |                   状态管理                   |
| :-----------------------------------------------: | :-----------------------------------------------------: | :-----------------------------------------------------------------------------------: | :--------------------------------------: |
| [Taro3.x(React18、Webpack5)](https://taro.js.org/) | [@antmjs/vantui](https://antmjs.github.io/vantui/main/) | [weapp-tailwindcss](https://weapp-tw.icebreaker.top/docs/quick-start/frameworks/taro) | [zustand](https://zustand-demo.pmnd.rs/) |

### 原生 or Taro or uni-app

毫无疑问，对于没有跨端需求的我们，写原生必然是性能和可扩展性最好的方案。`uni-app`也是大多数人首选的小程序开发框架，对比`Taro`，其拥有更大的用户群体、更丰富的生态，还有我没测试过但是社区反馈的更好的性能。但是鉴于我们已经存在使用`Taro`开发的小程序项目，其没有出现明显的性能短板，且只有`Taro`支持`React`写法，另外主管倾向于不引入新的前端框架增加学习成本。所以我们还是选择了使用`Taro`搭建项目模板。

`Taro`官方团队已经推出[Taro 4.x](https://juejin.cn/post/7330792655125463067)，支持了`vite`编译、开发鸿蒙应用等功能。不过鉴于我们对`vite`编译带来的性能优化（个人感觉很奇怪，当前小程序都是全量编译后才能在开发者工具上运行的，`vite`的异步加载`js`的性能优势该怎么体现出来呢？）和`鸿蒙`等功能没有硬需求，且`Taro`团队仍会继续就框架的问题修复和性能优化支持维护一段时间，所以还是倾向于选择相对稳定的`3.x`版本。

### UI组件库

在以往的支付宝小程序开发经历中，发现了该`@antmjs/vantui`组件库对于`Taro（React）`的支持较好、组件丰富度也比官方推荐的几个组件库优秀，且维护团队回答问题、解决bug很及时。所以在后续的小程序开发中选择采用该组件库作为小程序模板项目的组件库。

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/d690e56ab34d443eb0379fa43b6026e7~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228542&x-orig-sign=tkUopBxW%2F6Ay6UVAm%2BL84RQ30Mk%3D">

模板代码也是参考[AntmJS/pure-project-vantui](https://github.com/AntmJS/pure-project-vantui)修改而来，对于`H5`或其他小程序的适配可以参考该工程。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/1edc97d47aa74b66bf9fa406047f9a6c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228542&x-orig-sign=fUVZ7jTwXr2PZUIIm3bKX0M3tNM%3D)

### 样式方案

我们PC端项目已经很普遍地在使用`tailwindcss`，原子化`CSS`带来的开发爽感本人深有体会，果断选择继续引入`tailwindcss`，官方已经给出了引入方案：[使用 Tailwind CSS | Taro 文档](https://docs.taro.zone/docs/tailwindcss)。引入`weapp-tailwindcss`，并按要求配置即可体验。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/d5512814cd97428b9713b73eb57dda86~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228542&x-orig-sign=cGYoqYsgMpxF%2Fs5gakwfripf%2Bhg%3D)

### 状态管理

`React`项目的状态管理工具五花八门。`Taro`官方文档介绍的是`Redux`，个人感觉`Redux`太重、太繁琐，果断转向选择了广受社区好评的`zustand`，自己也在项目中使用过，配置方便、引入简单，也可以在`React`之外使用，方便扩展。参考[welives](https://github.com/welives)大佬的[Taro-React工程搭建 | 学习笔记](https://welives.github.io/blog/front-end/engineering/taro/create-react.html)成功引入了`zustand`，并扩展了可以通过`Taro`的`API`实现持久化的功能。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/3257c82157e144acb01c26fd071cd039~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228542&x-orig-sign=Vg4Jr6i9CseBZYjDwLnI98%2FHiaU%3D)

### 多端兼容

> 目前只支持`微信`、`支付宝`小程序。

#### CSS样式兼容

`微信`和`支付宝`小程序的样式无法直接兼容，需要通过`postcss`进行单位转换。

安装`postcss-rem-to-responsive-pixel`插件后，通过在`postcss.config.js`文件中添加以下代码便可以实现基本的样式兼容。

```js
plugins: {
    tailwindcss: {},
    autoprefixer: {},
    'postcss-rem-to-responsive-pixel': {
      rootValue: 32, // 1rem = 32rpx
      propList: ['*'], // 默认所有属性都转化
      transformUnit: process.env.TARO_ENV === 'h5' ? 'px' : 'rpx', // 转化的单位,可以变成 px / rpx
    },
  }
```

### 效果

<table>
  <tbody align="center">
      <tr>
          <td>微信小程序</td>
          <td>支付宝小程序</td>
      </tr>
      <tr>
    <td><img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/ee1c40d301534fdb82f865513f077952~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228542&x-orig-sign=p%2F9vVSHlK%2FFaVpmNbZ18aaT0a80%3D" width="350"></td>
    <td><img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/38c2f6a9f8ef465fb1a89830814cadce~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228542&x-orig-sign=vBxfPVlHkLMSsSQ09UnirTdFvRQ%3D" width="350"></td>
  </tr>
</tbody></table>

### 预期

#### TailwindCSS v4

最近`TailwindCSS v4`已经发布了，[icebreaker](https://github.com/sonofmagic) 大佬也是跟进发布了最新版本的 `weapp-tailwindcss`，所以我计划等 4.0 版本的`weapp-tailwindcss`稳定后，跟进到模板项目中。

#### Taro 4.x

`Taro 4.x`也已经发布了很久了，虽然我们团队暂时还没有考虑跟进，不过可以在自己的demo上尝试一下，应该会有不小的性能提升吧😂，不过还是期待支持`Rsbuild`构建，性能提升的同时，迁移成本更小。
