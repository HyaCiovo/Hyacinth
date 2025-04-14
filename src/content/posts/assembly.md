---
title: AS 不就是 AnyScript 嘛，那我可太擅长了
published: 2023-12-28
updated: 2023-12-28
description: '刚入职时，遇到了一个线上 bug，用户点击复制按钮没办法复制文本，产品经理震怒，“这么简单的一个功能也能出问题？当时是谁验收的？”，因为我刚来还闲着，就把我派去解决这个问题。'
tags: [Assembly, crypto]
category: 'Learning'
draft: false 
---

## 前言

接之前提到的主管让我调研在浏览器 WebAssembly 里跑加解密过程的问题，这里记录一下自己学习使用 AS 编译 wasm 文件并实现前端调用的小 demo 的过程。

## 为什么选 AssemblyScript

主管说他了解了一下，wasm 文件可以在浏览器的一个隔离环境里运行，很适合做加解密（真的吗？这个隔离环境真没办法写脚本扒出来吗？😦）。然后呢，他们一群“老叔叔”都是 Javaer，也没了解到有 Java 编译成 .wasm文件的解决方案。好巧不巧他又找到了 AssemblyScript，说是类似 TypeScript 的语法，好嘛，我这个天天写 AnyScript 的不搁这呢吗，AnyScript 也是 AS ！正好我刚搞完之前布置给我的 qiankun 问题闲了下来，一个 Markdown 文件就甩到消息列表里了：1、了解一下 AssemblyScript；2、写个 HelloWorld demo；3、前端调用；4、看看能不能实现加解密（找找有没有现成的类似 crypto-js 加密库）。

## 了解 AssemblyScript

[介绍 | AssemblyScript 中文网 (nodejs.cn)](https://assemblyscript.nodejs.cn/introduction.html#从-webassembly-的角度来看)

[Introduction | The AssemblyScript Book](https://www.assemblyscript.org/introduction.html#from-a-webassembly-perspective)

> ​	官网介绍：它与 TypeScript 类似，但对于 **WebAssembly 类型**，由于 **提前** 编译 **严格类型化** 代码有一些限制，但也有一些源自 WebAssembly 功能集的添加。 虽然并非所有 TypeScript 都能得到支持，但它与 JavaScript 的密切关系使其成为已经习惯为 Web 编写代码的开发者的熟悉选择，并且还具有与现有 Web 平台概念无缝集成的潜力，以产生精益和平均的结果 WebAssembly 模块。

简单了解完，就是我们可以写 ts 文件，然后通过编译器用二进制编译为 wasm 文件，但是它写的 ts 并不等同于我们平时写的 ts ，不能把已有的 ts 程序编译成 wasm 文件。

## Aseembly 启动！

1. 在 cmd 输入以下命令

   ```bash
   npm init
   npm install --save-dev assemblyscript
   npx asinit .
   ```

   跟着官网的指南，新建了一个项目，生成的项目如下：

   
<p align=center><img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/493e8744d060452c93dbef056bc128ff~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1919&h=1077&s=197149&e=png&b=262a32" alt="image.png"  /></p>

2. 然后我们把 index.ts 中内容修改为：

   ```ts
   export function hello(): string {
     return "Hello World";
   }
   ```

3. 运行 npm run asbuild

   在 build/ 目录中生成了以下文件：

   
<p align=center><img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8eacbfd59ff24286bf2ea1df57d262bf~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=419&h=401&s=25969&e=png&b=22262c" alt="image.png"  /></p>
到这里已经编译出了我们需要的 wasm 文件，然后我发现这里还生成了一个 js 文件，然后分析了一下 release.js 和项目 index.html 中的已有代码，发现 AssemblyScript 的编译器贴心的帮我把如何调用 wasm 文件的 js 文件一起编译好了 👍👍，这个功能实在是太赞了（PS：因为我在后期尝试使用 TinyGo 编译 wasm 文件时，需要自己写一些 js 代码调用），这样我们完全可以在需要调用的地方，导入 release.js 中 export 出的函数。</p>


4. 将 release.wasm 和 release.js 文件拷贝到前端工程中，在页面中直接 import 我们的 release.js 文件。

   
<p align=center><img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d0ba336e1034e5788bce72d4c317107~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=420&h=162&s=10783&e=png&b=24282f" alt="image.png"  /></p>
<p align=center><img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/701eaa3a28be4a3c8cbc51a511597388~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=812&h=264&s=46302&e=png&b=282c34" alt="image.png"  /></p>

5. 启动前端项目，查看效果（tailwind + daisyui 真的香）

   
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3f150521f7a47988b419279b5b7e2af~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1134&h=972&s=114916&e=png&b=fefefe)

   Bingo，成功在 React 项目中调用了 wasm 文件👻。

## Later ...

现在该写加密算法了！ 然后我熟练地 npm i @types/crypto-js ；import CryptoJS from "crypto-js"；用 AES 算法写了个 encode 和 decode 算法；npm run asbuild 😎。 

报错！啊对，我写的不是原来的 TypeScript，原来的工具库不能用！快去 Github 搜搜！

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47bfed13d54b4d52bb3a41b9604e9506~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1912&h=967&s=221788&e=png&b=ffffff)

结果没有一个能全部提供满足我们使用的一系列对称加密、非对称加密、摘要算法的😫。

AssemblyScript 官方仓库也有相关的 issue，看大佬的回复要使用 AssemblyScript 的主机绑定？并不知道这个主机绑定是啥意思，看文档也是云里雾里，可能是在 WebAssembly 里直接套娃运行 JavaScript？那考虑性能的话用我之前写的 Golang 更合适吧 ...... （然后我又去试了一下 TinyGo ，难度有点大，最后还是放弃了）

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6881ef597e94aaeba248165e97e1819~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1357&h=907&s=119746&e=png&b=ffffff)


## 最后

没办法，找主管汇报吧！

“AS我了解了！是 AssemblyScript ，不是 AnyScript ！Demo写出来了！Hello啥我都能给你返回出来，我还抄了个简单的 encode 和 decode 方法😙，加解密也能写！”

**“老大，你不会觉得我能写出来 AssemblyScript 的 crypto-as 吧😧”**

































