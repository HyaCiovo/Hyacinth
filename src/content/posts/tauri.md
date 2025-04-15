---
title: 使用 Tauri 构建的小工具体积居然这么小
published: 2024-05-24
updated: 2024-05-24
description: '使用Svelte+Vite+daisyUI帮测试同学做的DecryptKit解密小工具，Tauri打包后的程序体积居然只有3.5MB。'
tags: [Tauri,Svelte,Crypto]
category: 'Learning'
draft: false 
---

### 背景

最近第三方公司在陆续交付新平台了，测试同学发现很多接口返回的数据是加密了的，测试时很不方便。

> ps：说实话，我很难理解这种加密需求，这些数据本来就是要展示的，为什么还要加一层加解密过程呢？前端要解密浏览器也是一定能拿到密钥的吧，懂得看接口数据的人也不会找不到密钥在哪的呀🤔，真“防君子不防小人”？

于是他让我看看前端是怎么展示这些加密数据的，用的什么加解密算法。

于是我熟练地全局搜索"crypto"，发现是使用了"gm-crypto"的 `SM4`算法。想着最近事情不多，我就直接写个ui界面让测试同学用了。

`代码和演示地址在文末。`

### 开发

于是我把之前摆弄Svelte时建的仓库拿来，起了一个小项目:

```bash
pnpm add gm-crypto
```

然后用daisyUI手搓了一个`SM4`解密界面：

  ![image-20240524154556300.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c659cd5ce6a4f3bb1a0e91b3887f101~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1902&h=952&s=115047&e=png&b=282a35)



daisyUI 的样式搭配 tailwindcss 在开发小工具的时候真的是十分方便。

### 开箱即用

开发完之后，我想到总不能把项目发给测试同学，每次用的时候再本地`npm run dev`🤣吧。总得整个他能很方便打开的方式。于是我尝试了以下几种方式：

1.  **部署上线**。首先是我本人最常用的 Vercel 一键部署，直接到 Vercel 控制台“点点点”，就可以很方便地将网站部署到 Vecel 的免费服务器上🥳。
   
    ![image-20240524160428563.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3696f139b95d4821b2bad1b02441740f~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1884&h=932&s=130231&e=png&b=fdfdfd)

    可惜前段时间Vercel被墙了，想访问在Vercel上部署的网站只能通过魔法。

2.  **打包成.exe文件**。直接打包成可在 win 上运行的 .exe 文件，把安装包发给测试同学不就可以了！不过之前自己了解 Electron 时，用 Electron 打包了一个纯前端的 Vue3 项目，一共4个页面，最后的程序有100多MB。我寻思我这次就一个页面，如果打包成这么大的项目也不太行。恰好前段时间为了了解 wasm 我安装了 Rust，又了解到了 Tauri 打包的程序体积很小，而我这个纯前端项目也不用写 Rust 服务，还不用考虑兼容性（全部win11），那么就试试 Tauri 吧！

    *   准备：参考 [预先准备 | Tauri Apps](https://tauri.app/zh-cn/v1/guides/getting-started/prerequisites)

    *   集成：参考 [集成到已有项目 | Tauri Apps](https://tauri.app/zh-cn/v1/guides/getting-started/setup/integrate)

    *   配置：主要需要注意 tauri.conf.json 中的以下配置。

        ```json
        {
          "$schema": "../node_modules/@tauri-apps/cli/schema.json",
          "build": {
            "beforeBuildCommand": "pnpm run build:browser",  //对应 package.json
            "beforeDevCommand": "pnpm run dev:browser", //对应 package.json
            "devPath": "http://localhost:8080", //对应 vite.config.ts
            "distDir": "../dist"  //对应 vite.config.ts
          },
          // ...
          "tauri": {
            // ...
            "bundle": {
              // ...
              "identifier": "DecryptKit", // 随便修改一个特定的标识
              // ...
            },
            // ...
          }
        }
        ```

    *   本地开发：

        ```bash
        tauri dev
        ```

        > 首次启动时，需要下载和编译很多依赖，编译时间较长，可以先去做别的事。还可能因为网络问题报错，`删除target文件夹`并重新运行即可。 后续再启动就会很快了。
        >
        > 启动成功后： ![image-20240524162916529.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a72ebf5c34b44ebcaa335c39cdf016c9~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=964&h=1079&s=68937&e=png&b=ffffff)

    *   打包：

        ```bash
        tauri build
        ```

        > 同上，首次打包时也很慢。
        >
        > 打包成功后：
        >
        > ![image-20240524164006301.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87916f65a74a4888b3c8f3784c819a35~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=595&h=897&s=60454&e=png&b=22262c)


打包生成后的.exe程序竟然只有3.5MB🥳!

![image-20240524164145053.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed2411ed5fcf4846a3ff2081a51413cb~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=394&h=147&s=6824&e=png&b=fcfbfb)

### 代码与演示地址

*   代码仓库：[HyaCiovo/DecryptKit-SM4 (github.com)](https://github.com/HyaCiovo/DecryptKit-SM4)
*   演示地址：[DecryptKit (decryptkit-sm4.vercel.app)](https://decryptkit-sm4.vercel.app/)
*   软件：<https://pan.xunlei.com/s/VNydp8EVIHU_b-l1b5LtUMD8A1?pwd=h2u3#>  提取码：h2u3
