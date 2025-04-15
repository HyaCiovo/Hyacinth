---
title: 新手 Nuxt3 爬坑小指南
published: 2022-12-28
updated: 2022-12-28
description: '菜鸟前端，近来没有什么业务需求，就想着了解一下之前没接触过的一些流行的前端技术。不过在学习Nuxt3时kuangkuang踩坑，这里做一下记录，希望在方便自己复习的同时，也能帮到后面学习的同学。'
tags: [Vue3, Nuxt]
category: '兴趣扩展'
draft: false 
---
## 前言

菜鸟前端，近来没有什么业务需求，所以就想着了解一下之前没接触过的一些流行的前端技术。这一个月大概过了一遍`Vue3、Next和Nuxt3`。因为之前实习和现在工作积累了一些`Vue2和React`的使用经验，所以`Vue3`接受起来并不困难。`Next`也大致过了一遍文档，了解了一下约定式路由，跟着写了个分别实现了`SSG、ISR和SSR`的小 demo。不过在`Nuxt3`这里受到了很大的阻力（踩坑），这里做一下记录，希望在方便自己复习的同时，也能帮到后面学习的同学。😁😁

### 这里先摆一张知乎看到的图🤣

![img](https://pic1.zhimg.com/v2-d71f5670d6b8bef77ba1faf3d9b6eaa0_xld.png)

## 安装

```
# npx
npx nuxi init nuxt-app
# pnpm
pnpm dlx nuxi init nuxt-app
```

这里有坑，大概率报错🙃 （我想不少同学在这就被劝退了hh）

由于国内的`GFW`，这个域名受到了`DNS`污染，没法连接。但即使是科学上网也是大概率挂掉的。

查找原因后了解到`nuxt`的脚手架`nuxi`使用了`giget`来从`nuxt`项目模板仓库中获取文件。

giget干的事情很简单，就是利用`node`从`github`上拉取相应仓库。实际上`giget`貌似是`nuxt`团队对另一个相似的项目`degit`的仿制。两者都可以用方便的命令从`github`拉取仓库。唯一的不同就是`degit`支持自动从环境变量中获取`https_proxy`进行代理，而`giget`完全没有考虑这一点😅

github上已经有人提了pr 并且开发团队已采纳 不过我在这之后安装还是会报错。

试了网上不同的解决方法，目前有用的是[nuxt3项目初始化失败 - 掘金 (juejin.cn)](https://juejin.cn/post/7154586714416087076)

直接去修改本地的hosts文件：

新增一行， `185.199.108.133 raw.githubusercontent.com`

在稳定的网络环境并且可以`ping`通raw.githubusercontent.com后再运行安装命令（我这里使用的是`npx`版本的命令）可以成功初始化安装Nuxt3项目文件。

`issue`里有提到修改DNS为 `114.114.114.114` 但我试过后没有成功。

`issue`里还提供了一种，安装`nuxi`后，去修改`nuxi`的文件中的`proxy`地址，这里作者没有尝试，有兴趣的同学可以去下面的网址参考。

[npx nuxi init nuxt3-app and download template error · Issue #3430 · nuxt/framework (github.com)](https://github.com/nuxt/framework/issues/3430)

## 获取数据

因为`Nuxt3`是`SSR`的方案，所以你可能不仅仅只是想要在浏览器端发送请求获取数据，还想在服务器端就获取到数据并渲染组件。

`Nuxt3`提供了 4 种方式使得你可以在服务器端异步获取数据

-   useAsyncData
-   useLazyAsyncData （useAsyncData+lazy:true）
-   useFetch
-   useLazyFetch （useFetch+lazy:true）

> 注意：他们只能在**`setup`**或者是`生命周期钩子`中使用

> useAsyncData/useFetch默认是服务端获取数据，但其暴露出的`refresh方法是从客户端获取数据`；
>
> 如果不想在服务端获取数据，可以在`options`里设置 `server : false` , 这样就可以在客户端获取数据；
>
> Nuxt3提供了自己的请求命令`$fetch` 但我们仍可以使用 `axios`来获取，方便我们从已有Vue3项目迁移。

踩坑记录：

-   使用`useFetch`获取数据时，发现在`network`里找不到请求信息

`useFetch`默认服务端获取数据，当然不会在客户端中获取到`Fetch/XHR`信息；如果想要在客户端请求，需要在`options`中配置 `server : false`

-   使用`useFetch`的`refresh`在请求`Nuxt`的`server`中的`api`时正常使用，在请求本地`node`服务时报跨域错误

`useAsyncData/useFetch`的 `refresh` 是客户端发出的请求，服务端请求是不会产生跨域问题的，但客户端 会产生跨域问题，开发时就需要服务端配置 *Access-Control-Allow-Origin* 或者前端项目配置`proxy`代理

-   在项目中配置前端代理`proxy`无效

`Nuxt3`文档中并未提及如何配置`proxy`，应该是默认了开发时跨域问题由服务端解决。网上搜索并没有很多 `Nuxt3`的博客，且大部分都不提设置`proxy`（难道全是前后端不分离用`node`做后端？），找到的几个提到`proxy`的博客都是说`nuxt.config.ts`里支持了`vite`，在`vite`属性中配置类似之前`Vite`的代理方式就可以了。亲测没用👍

```
export default defineNuxtConfig({
  vite: {
    server: {
      open: true,
      // https: true,
      proxy: {
        "/devApi": {
          target: "http://127.0.0.1:8085/",
          changeOrigin: true,
          rewrite: (path) => path.replace(/^/devApi/, ""),
        },
      },
    },
  }
})
```

后面在Nuxt3的issue里发现是要在nitro属性中设置，配置后便解决了开发跨域的问题。

```
export default defineNuxtConfig({
  nitro: {
    devProxy: {
      "/proxy": {
        target: "http://127.0.0.1:8085/",
        changeOrigin: true,
        prependPath: true,
      }
    }
  }
})
```

## Pages

`Nuxt3`的`Pages`对于`Next`的约定式文件路由做了大量的“借鉴”，包括动态路由的实现方式

但ta的使用体验个人觉得是不如`Next13`的，

嵌套路由`Nested Router`是`Nuxt3`对于作者来说的一个`feature`，虽然目前本菜鸟还没遇到需要使用嵌套路由才能实现的业务场景😂

> 这里有个坑，中文文档嵌套路由的实现是引入`<NuxtChild />`组件，但试了之后没有用。
>
> 看了github的issue后发现是已经修改为了`<NuxtPage />`组件，中文文档并没有同步更新。

# ......

其他的本人都跟着文档敲了一遍，没有再碰到其他特别难以解决的坑了。

# 后话

遇到问题不要慌，文档查一查，博客搜一搜，issue扒一扒，基本都能解决。

> 最好提高下自己的英语水平，因为中文文档很多不能够与官方文档保持同步更新，落后版本 === kuangkuang掉大坑🕳

<img src="https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9uVnNaR081ZHZxbFJsVDNPRmhwWWljUVdJWVg4dDBpYWhRb0ZyR3ZWN2NZdTR0V1ZrdTFLUll5clZSa2NjUVNybXpNSG9pY0VzZ3RnU1F6T2tPaWI2V1BMdVEvNjQw?x-oss-process=image/format,png" />
