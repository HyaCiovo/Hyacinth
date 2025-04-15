---
title: 使用 Vercel 部署 SolidJS+daisyUI 的纯前端人脸识别项目
published: 2024-05-24
updated: 2024-05-24
description: '使用Vercel部署SolidJS+daisyUI的纯前端人脸识别项目。 使用tracking.js识别人脸，并上传到百度智能云进行人脸对比。解决Vercel上传项目的接口跨域问题。'
tags: [Solid.js,Vercel]
category: '兴趣扩展'
draft: false 
---

### 背景

又是一年毕业季，到了大学生毕设答辩的时候了。想起来自己两年前的毕设被老师点评过人脸识别功能摆的位置不合理（为了凑工作量加的功能🫠），趁着这两天还不算忙，自己试试做一下她建议的人脸识别登录，这里记录一下过程。

### 代码仓库与演示地址：

代码仓库：[HyaCiovo/FaceLoginHya (github.com)](https://github.com/HyaCiovo/FaceLoginHya)

演示地址：[HyaCiovo/FaceLoginHya (vercel.app)](https://face-login-hya.vercel.app)
>（因为删除了个人第三方平台的信息，所以无法演示完整功能😥，想要体验人脸登录功能需要自行注册百度智能云平台，使用免费的人脸对比API。体验人脸库操作的人脸回显功能需使用到[阿里云oss](https://www.aliyun.com/activity?userCode=txcdw8rg)）

### 相关代码库与第三方平台：

前端：[SolidJS](https://www.solidjs.com/)（写起来比React还爽，就是没有我最爱的Antd😥）、[solid-router](https://github.com/solidjs/solid-router)、[solid-use](https://solidjs-use.github.io/solidjs-use/)、[daisyUI](https://daisyui.com/)（好用好看，写小项目小Demo很适合😙）、[tracking.js](https://trackingjs.com/)

第三方平台：[阿里云oss](https://www.aliyun.com/product/oss?userCode=txcdw8rg)、[百度智能云](https://cloud.baidu.com/)、[Vercel](https://vercel.com/)

### 流程：

主要是浏览器调起前置摄像头借助tracking.js识别人脸；识别到人脸后用canvas“拍照”并上传到百度智能云平台；百度智能云平台将照片与人脸库进行对比，并返回对比结果到客户端；客户端将返回的结果与用户信息对比来实现登录功能。通过Vercel平台直接部署本项目。

### 效果：

#### 移动端：

![7ec5a517ff2917f662afea17a2bb6e8.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9209a1c74234d59a51fe1f7b49c7a4a~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=3072&h=3014&s=1389899&e=jpg&b=e6effc)

#### PC端：

![c6cda4b434e5e051ef5dd5681183a00.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39bb3f7788d246ccb96191fbe3444aa5~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1896&h=960&s=2171755&e=png&a=1&b=07193b)

![9614b21395fcd14dbe9e979b9b685ab.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0392ec196dbc4c84b44136e0aac437d3~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1900&h=960&s=2001455&e=png&a=1&b=07183a)

### 注意：

- 由于隐私问题，不会上传完整代码。阿里云oss和百度智能云相关id和密钥配置可在"src/configs/constants.ts"文件中修改；
  ![image-20240524110432114.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fd04f9f6faa44478c31ba4a236910c1~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1365&h=680&s=121900&e=png&b=282c34)
  
- 登录页的很多背景图因为都是本人从别家扒的，所以也不放进项目中了👻；

- 本人因为图省事纯前端操作，阿里云oss设置了公共读，写操作也是直接客户端操作没怎么搞STS访问控制，这是不安全的；

- 人脸识别本地调试时，摄像头只能在localhost和https上调起，如果要在移动端调起需要配上https，可在vite.config.ts中打开注释

  ```ts
  import { defineConfig } from "vite";
  import solidPlugin from "vite-plugin-solid";
  import { resolve } from "path";
  // import basicSsl from "@vitejs/plugin-basic-ssl"; // https调试
  
  export default defineConfig({
    // ...
    plugins: [
      solidPlugin(),
      // basicSsl() // 本地https调试
    ],
    server: {
      // https: true, // 本地https调试
      host: "0.0.0.0",
      open: true,
      port: 3000,
  	// ...
    },
  });
  ```
- 直接部署到Vercel上的前端项目不会去处理跨域问题，需要开发者在根目录新建`vercel.json`和`api/proxy.js`文件并参考`vite.config.ts`来配置:

  > `vite.config.ts:`
  ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2f902827f874ba59aaeb3466851be89~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=789&h=356&s=40158&e=png&b=282c34)
  `vercel.json:`
  ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25963643bb1045a09cbe5c5a79c4a60c~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=680&h=388&s=30207&e=png&b=282c34)
  `api/proxy.js:`
  ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ce7e333ab3643e5b0c0f15a7bcbd1bb~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=962&h=718&s=148137&e=png&b=282c34)
  切记安装 `http-proxy-middleware` ！！！😈😈😈😈
  
### 最后
本文仅做记录，这个项目只是“小玩具”，用来练练手，没有任何准确性和安全性可言😎。精准的人脸识别项目还是要花钱上各大厂商的成熟产品。