---
title: 解决加载 qiankun 子应用图片404问题
published: 2024-09-10
updated: 2024-09-10
description: '本文记录解决在`qiankun`微前端架构项目中“父应用无法正确加载子应用静态文件”的问题。主要通过正确配置子应用的静态文件资源目录和父应用资源转发。'
tags: [qiankun]
category: 'Work'
draft: false 
---

### 问题

测试同学发现当前在子应用“权限中心”使用了的两张静态`png`图片，图片在独立运行的子应用上可以正常加载展示，但是在父应用基座上运行时，图片无法正常加载，控制台提示`404`。

### 解决

通过之前对`qiankun`的了解和学习：[主管让我说说 qiankun 是咋回事😶](https://juejin.cn/post/7314196310647423039)

> 父子应用资源的相关知识：
>
> `qiankun`微前端，父应用上加载子应用的静态文件时，需要在`Nginx`（本地通过`proxy`）配置子应用静态文件的代理地址，通过前缀匹配，分别转发到子应用所在的目录下拉取文件。



我很容易地确定这是关于该子应用静态资源请求转发的问题：

> - 父应用nginx把请求地址转发错了
> - 子应用的静态资源目录配置的有问题
> 
> 其实这两个问题都是一个意思🤣


于是我检查父应用的`network`，发现在父应用中已经为权限中心子应用统一配置了`/userCenter/`的前缀，理论上可以成功转发静态文件的请求，请求报`404`的原因最有可能是子应用配置中的`publicPath`、`base`配置有问题。



自然而然地要去检查子应用（`umi`项目）的`config`文件具体代码配置：

```ts
// config.ts
export default defineConfig({
  nodeModulesTransform: {
    type: 'none'
  },
  routes,
  base: CONFIG[NODE_ENV]?.main_base || '/',
  publicPath: CONFIG[NODE_ENV]?.main_publicPath || '/',
  ...
})
    
// config.uat.ts
import baseConfig from './config'
export default defineConfig({
  ...baseConfig,
  title: '用户权限中心',
  layout: false,
  favicon: 'img/xxx.ico',
  define: {
    ...baseConfig.define,
    QIAN_KUN: 'main',
    BUILD_ENV: 'uat'
  },
  qiankun: {
    slave: {}
  }
})


// config.sub.ts
import baseConfig from './config'
export default defineConfig({
  base: CONFIG[NODE_ENV]?.base || '/micro/',
  publicPath: CONFIG[NODE_ENV]?.publicPath || '/micro/',
  ...baseConfig,
  mfsu: undefined,
  dynamicImport: undefined,
  define: {
    ...baseConfig.define,
    BUILD_ENV: 'uat',
    QIAN_KUN: 'sub'
  },
  qiankun: {
    slave: {}
  }
  ...
})
```



从代码配置中可以看到，`config.ts`中为默认配置，在`config.sub.ts`中对`base`、`publicPath`进行了重写。可以理解这里是对作为子应用运行的代码进行了单独的配置，但是并没有生效。貌似`sub.ts`中的代码没有成功重写这两个配置，于是我尝试这两个配置项直接分别写到`config.sub.ts`、`config.uat.ts`中，重新部署子应用。

```ts
// config.uat.ts
export default defineConfig({
  ...
  base: CONFIG[NODE_ENV]?.main_base || '/',
  publicPath: CONFIG[NODE_ENV]?.main_publicPath || '/',
  ...
})

// config.sub.ts
export default defineConfig({
  ...
  base: CONFIG[NODE_ENV]?.base || '/micro/',
  publicPath: CONFIG[NODE_ENV]?.publicPath || '/micro/',
  ...
})
```



重新部署子应用后，刷新父应用页面，发现已经可以成功加载该子应用的静态资源文件。问题得到解决✅。

> 注意：
> 子应用的静态资源需要配置跨域