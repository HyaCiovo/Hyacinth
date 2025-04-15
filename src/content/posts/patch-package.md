---
title: 使用 patch-package 修改第三方包源码，提升开发体验
published: 2024-08-26
updated: 2024-08-26
description: '当你需要直接修改第三方包源码，但又不想或者不能直接向原仓库提交 PR 时，你可以使用 patch-package 这个小工具实现优雅地修改第三方包代码😎'
tags: [React]
category: '工作记录'
draft: false 
---

## 背景

最近总算开始开发之前总是提到的接手的 `umi+qiankun` 项目了，要给父子应用加上一堆`if...else...` 。

在有了之前积累的一些 `qiankun` 相关开发知识之后，我也是很快地将项目运行了起来，于是我习惯性地`F12`打开控制台......

映入眼帘的是大量的error、warning 红的黄的打满了 log😅

![image-20240826184239626.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/a064f419c1bf4818ac27e615b531e9a9~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=7dnSZUa2meQf0MKkMctSrsyAuB0%3D)

![image-20240826184335483.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/170af8e5aa344f69a449b3abcc73141b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=zAlDGGCTVgjWVdevNtYUwPH6rTM%3D)

看来第三方移交的代码质量真的堪忧啊😇，虽然也不是不能用，但是开发的时候想找到自己打的 log 体验有点太差了，还是尽量修一下吧，毕竟大部分还是循环生成的组件没加 key 这些问题。

## 难题

当我正沉浸在一两分钟消灭掉一行 error 的成就感中时，我遇到了一个不知道怎么解决的问题：

项目父应用是`umi4 + React17 + antd5`, antd 的 `Form` 组件中使用 mode 为"multiple"的 `Select` 组件，当组件切换时，控制台会报错：`Can't perform a React state update on an unmounted component.`

![image-20240826192707155.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/cbef3afc195f4dca93360beeb94c7502~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=c2Wl1A2NZ9di4gj9RXgDDa1%2FYDk%3D)

这该怎么解决？看了一下项目代码根本没有 setState 相关的，那应该是在 antd 项目代码中了，于是我查了一下 ant-design 的 issue，果然让我查到了一样的问题。

![image-20240826194521121.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/66129364f7ce408a92f0c0b33035e0b3~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=2BV%2FZ0ZWDnN9Gc4bI6SR3KRa2dk%3D)

下面的回复是这是 React17 的问题，React18 团队在源码中已经删掉了这一警告：

![image-20240826194629569.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/0020027d1e4643d995f4228277ee9920~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=MqBDmvC29wvqxRKm%2FzIKvwPJ3Ng%3D)

但是现在我们的项目是不敢轻易升 React 版本的，如果升了版本父应用两个平台，还有4个子应用的测试成本实在太高，而且只为了去掉这一个警告做这么多工作，一样没有性价比。那该怎么办呢？

## 解决

我想到要不直接去改源码，反正React团队也说这个警告没啥用处了，直接进去把这行log给注释掉好了。但不知道该怎么和同事们同步这个更改更合适。

于是我请教同事，她提出了几个可行的解决方案：

*   本地 node\_modules 搜 react-dom 包里的 warning 字符串处理掉 + patch-package 打补丁

*   本地 node\_modules 搜 react-dom包里的 warning 字符串处理掉 + npm 发包到私库 + 固定版本

考虑了一下工作量，我果断选择了第一种解决方案，以下是我的解决步骤：

1.  安装 patch-package

    ```bash
    yarn add patch-package -D
    ```

2.  修改 react-dom 源码

    ![image-20240826195942300.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/3b32e43eb4884269b14b9c0bc9de51d3~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=0fqO4YC1D%2F3CVuzokxw1jfynjxE%3D)

3.  生成补丁文件

    ```bash
    npx patch-package react-dom
    ```

    ![image-20240826200404092.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/8aa7bf2df00a475db64bad0e10741233~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=NPp1djjVkrLZaHN5yDQ9mklPqVM%3D)

    运行成功后会在根目录生成一个名为`patches`的文件夹,其中就有刚刚生成的`.patch`补丁文件，其中是一些 `git diff` 描述：

    ![image-20240826200745027.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/339cd112cb8543189fcbce21b9b5654a~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=gBLCuGwBirmuD6xpupDY%2BnTMfvI%3D)

4.  同步补丁

    将生成的 `patch` 文件提交到 `git`，并修改 `package.json` 中的 `postinstall` 命令:

    ```json
    {
      ...,
      "scripts":{
          ...,
          "postinstall": "umi setup && patch-package",
          ...
      }
      ...
    }
    ```

    > 这样便可以在每次运行 `install` 命令后，会自动将 `patch` 补丁打到依赖包上。

    ![image-20240826201258571.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/bcd7bb54ebf94eccb936062e590a1381~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745284834&x-orig-sign=QxmBQhi%2F09PhbYauxMac0WHOFUk%3D)

> 注意：
> umi项目在修改完`node_modules`源码后，需要删除`src/.umi`和 `node_mdoules/cache`构建产物和缓存，这样本地修改才会生效。

以上便是我使用 `patch-package` 工具应用和提交对 react-dom 包小修改的全步骤。当你需要直接修改第三方包源码，但又不想或者不能直接向原仓库提交 PR 时，这个工具就非常有用。
