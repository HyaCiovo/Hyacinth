---
title: 记录后台项目从node-sass切换到dart-sass
published: 2024-10-31
updated: 2024-10-31
description: '本地开发苦 node-sass 久矣。想要换到 dart-sass，却又被使用的依赖包指定使用 node-sass ？那就全都换掉😡！'
tags: [sass,React]
category: '工作记录'
draft: false 
---


## 背景

后台项目使用 scss 文件来编写项目的样式文件，并使用`antd-scss-theme-plugin`这个插件来使用 sass 变量设置`antd 3`组件库的主题（`antd 3`官方使用 less 变量实现）。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/df7edf5197b347468b3a0600a45be93d~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228649&x-orig-sign=3%2Fc7fC3fk791IzMp2mpHJdwXNeo%3D)

该`antd-scss-theme-plugin`已经停止维护，其依赖的也已经停止维护的`scss-to-json`包指定了`node-sass^3.1.2, ^4.0.0`为 sass 编译器。

而`node-sass`作为一个已经停止更新的包，因为与`sass-loader`和`node`版本强绑定一直为社区诟病。官方已经停止更新并推荐使用`dart-sass`，其兼容`node-sass`的所有 api，并且得到了更好的维护和支持。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/6e5fe6a81d12415286625a441c68cd80~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228649&x-orig-sign=tqwJCCQOUFRY4SzKTVRMUqc6EP0%3D)

再看看JY们在一篇不久前发布的文章 [又报gyp ERR！为什么有那么多人被node-sass 坑过？node-sass: Command failed](https://juejin.cn/post/7408606153393307660?searchId=2024103114543697C8478E622E8E88D0DF) 评论区的吐槽:

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/01a494d823fd4c8eb2d0e88812dc0f91~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228649&x-orig-sign=ov1EpiDo81J%2F2PXbbZfKRRniIKI%3D" alt="e895ef9e9fc305a663fefe915245230" style="height:300px;">
<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/cfa8cdfe00f04a78b13e13305f5ea3b3~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228649&x-orig-sign=ZmmIaE2veawwfH0eGlzQvT9E3H0%3D" alt="e895ef9e9fc305a663fefe915245230" style="height:300px;">

在我的开发过程中，也是苦`node-sass`久矣，本地安装项目依赖会因为各种问题（主要是`node`版本、网络等）导致`node-sass`安装失败，严重影响开发效率😡。所以我决定将项目的`node-sass`切换到`dart-sass`上。

## 对比

*   维护和支持：`dart-sass`是 Sass 官方推荐的编译器，得到了更好的维护和支持；
*   性能：虽然`node-sass`在某些情况下编译速度更快，但`dart-sass`在不断的更新中已经显著提高了性能，并且更加稳定；
*   跨平台兼容性：`dart-sass`不依赖于本地编译，因此在不同的操作系统上的一致性和兼容性更好；
*   发展：Sass 团队明确表示，未来的功能和改进将主要集中在`dart-sass`上。

## 变更

因为是`scss-to-json`这个包指定使用的`node-sass`，所以尝试`fork`该`npm`包修改后发布私包，在该包`Github`的`issue`中，发现了`scss-json`这个项目兼容了`scss-to-json`的`api`并且使用`dart-sass`作为编译器，完美符合我的需求。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/23979545f1b74d179bbc72b4e48e74ae~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228649&x-orig-sign=bbHGJXgW4I7qK8EM%2Bwuzz8gbccU%3D)

于是我`fork`了`antd-scss-theme-plugin`，并指定使用`scss-json`替代`scss-to-json`来编译`scss`文件。重新安装依赖后，项目可以正常编译运行。但是`dart-sass`在`1.79.1`版本后会提示`^2.0`版本将会废弃部分项目中已经在使用的`api`[sass - npm (npmjs.com)](https://www.npmjs.com/package/sass/v/1.79.1)，虽然不影响项目运行，但是控制台有太多警告，影响开发体验。所以我锁定了`dart-sass`的版本在`scss-json`这个包中指定的`sass`最低版本`1.50.1`上,来解决本地运行警告问题。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/fc72893f13a64f96aea1ee33d63d0fd7~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228649&x-orig-sign=aQ336LQ4JRQD5wjNnulI2BEaB08%3D)

使用`compileString`替代`renderSync`使符合`dart-sass`写法，并优化字符串操作方法获得有限的性能提升。

```js
var sass = require("sass");
​
// var cssmin = require("cssmin");
// var s = sass.renderSync({ data: scssString });
// return cssmin(String(s.css))
​
var s = sass.compileString(scssString,{ style: "compressed", charset: false })
return s.css
```

这样项目就成功地从`node-sass`切换到了`dart-sass`，使用多个版本的`node`都可以成功安装运行✅。

## 运行对比

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/7b9ab82dc046430aa54209278ed348fc~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228649&x-orig-sign=wHcw6oJD%2BWZOfagWuxwegGZPneg%3D)

## 最后

虽然，`dart-sass`可以完美兼容`node-sass`的所有`api`，但是对于大型项目，`node-sass`还是有着 ta 的性能优势。官方也发布了[嵌入式 Sass（embedded-sass）](https://sass.js.cn/blog/embedded-sass-is-live/)来解决该性能问题，但目前我并没有发现社区有很多关于`embedded-sass`的使用经验，且我们的后台项目在切换到`dart-sass`后也没有明显的性能问题，所以暂时不考虑升级到`embedded-sass`上。

## 加更

我尝试了使用`sass-embedded`来作为`scss-json`中的 sass 编译器，在直接替换未进行任何代码改动的情况下，项目本地冷启动从原本的`2-4`分钟延长到了`6-7`分钟，生成物（含`sourcemap`）从`40MB`增大到了`48MB`😥。暂时还是不考虑引入了......
