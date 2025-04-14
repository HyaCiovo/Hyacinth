---
title: Next.js 服务端渲染超时导致生产事故
published: 2025-03-05
updated: 2025-03-05
description: '前几天公司平台首页崩溃了 504 Time-out，最后确定是因为服务端渲染时接口请求超时导致页面返回超时。这里记录一下问题排查和解决的过程。'
tags: [React, Next]
category: 'Work'
draft: false 
---


## 前言

前几天公司平台首页崩溃了 504 Time-out，这里记录一下问题排查和解决的过程。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/637736f484584f628b3a940ec0a06f8c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228181&x-orig-sign=5e8XdPkvHoWGHLR6FH7mb9fQo4U%3D)

## 排查和解决

### 排查过程

*   step 1：检查测试环境、开发环境未出现问题；
*   step 2：检查生产环境流水线，没有最新部署，排除前端新部署代码影响；
*   step 3：检查生产环境其他页面`/about-us`页面展示正常，但是部分接口报错超时。猜测是因为生产环境接口超时导致首页的服务端请求返回超时，导致首页返回超时；
*   step 4：本地开发调用生产环境接口，将所有在`getServerSideProps`中发起的请求全部注释后，首页可以正常加载。

### 问题定位

首页开启了`SSR`渲染，`getServerSideProps`中的请求使用了简单的`fetch`请求，只对普通错误做了兜底，但没有考虑到请求超时的问题，请求超时导致了页面返回超时，首页报了`504`错误。

### 解决方案

#### 优化方案

*   放弃使用`SSR`渲染，全部采用客户端请求渲染；
*   修改现有`fetch`，通过`AbortController`、`setTimeout`手动添加超时处理；
*   将`getServerSideProps`的请求方式从`fetch`迁移到`axios`，借助`axios`本身自带的超时配置，添加接口超时处理。

#### 方案比较

*   放弃`SSR`不可取，因为客户端并发请求返回顺序的问题可能会导致首页的入场动画混乱，且失去了部分首屏性能优化；
*   修改`fetch`方法，需要手写大量代码，且`AbortController`是在`Node.js 14.17.0+`开始支持，当然也可以不去放弃掉请求，可以通过`setTimeout`和`Promise.race`提前`reject`，不过需要因为是自行实现，风险大。
*   `axios`是十分成熟的请求库，且项目中已经引入，通过`axios`为请求添加超时处理十分方便。

#### 实际解决方案

将`fetch`迁移到`axios`，借助`axios`已有的超时处理配置解决当前问题，添加首页对`getServerSideProps`中请求超时的兜底，服务端请求超时后，立即返回空数组。

首页所有的数据消费组件继续兜底，当从`getServerSideProps`中获取的数据为空数组时，会在客户端再发一次请求，尝试从客户端获取数据。

```js
import axios from 'axios';

const ssrAxios = axios.create({
  timeout: 3000,
  timeoutErrorMessage: 'Request timed out'
});

export default ssrAxios;
```

```tsx
export const getServerSideProps = async () => {
  const addr = process.env.API_ADDR;

  // 使用 Promise.allSettled 获取所有请求的结果
  const [
    list1Res,
    // ... ,
    list2Res,
  ] = await Promise.allSettled([
    ssrAxios.get(`${addr}/api/list1`),
    // ... ,
    ssrAxios.get(`${addr}/api/list2`),
  ]);

  // 处理每个请求的结果
  const list1 = handleFetchResult(list1Res, 'list1');
  // ... ;
  const list2 = handleFetchResult(list2Res, 'list2');

  return {
    props: {
      list1: list1 ?? [],
      // ... ,
      list2: list2 ?? []
    },
  };
};

// Helper function to handle fetch results
const handleFetchResult =  (result: PromiseSettledResult<any>, key: string) => {
  if (result.status === 'rejected') {
    console.error(`Failed to fetch ${key}:`, result.reason);
    return [];
  }

  const { data, success } = result.value.data;
  if (!success) {
    console.error(`Failed to fetch ${key}`);
    return [];
  }

  switch (key) {
    case 'list1':
      return data?.result || [];
	// ... ;
    case 'list2':
      return data?.result || [];
    default:
      return [];
  }
};
```

## 问题探究

### 问题猜测

*   **接口性能瓶颈**：生产环境接口响应时间波动大，`SSR`并发请求存在短板
*   **SSR 超时机制缺失**：未设置合理的请求超时阈值，导致慢请求影响页面的生成和响应
*   **错误处理不完善**：前端错误处理考虑不全面，未区分超时错误与常规错误，只做了无返回或返回错误的处理，没有对请求超时做兜底。

### 验证猜想

本地模拟接口返回超时，首页会无法成功加载。添加超时处理兜底后，首页其他内容可以正常展示。

询问`deepseek`后，得知`Next.js`的`getServerSideProps`没有内置超时处理机制，因此长时间运行的`getServerSideProps`可能导致页面无法及时返回，甚至触发部署平台（如`Vercel`）的超时错误。

### 结论

首页开启了`SSR`渲染，`Next.js` 必须在完成`getServerSideProps`中的`所有`操作后才能生成页面并返回给客户端。 需要在得到所有需要的`props`后，才会返回整个页面。因为生产环境某个接口请求超时，并且没有正确处理，可能会导致整个`getServerSideProps`的执行时间延长，超过服务器的响应时间限制，从而使得页面无法及时返回，出现超时错误。

## 思考改进

### 思考

虽然是因为后端服务崩了，接口请求超时导致的问题，但是因为部分服务崩掉导致整个前端首页无法成功加载，作为前端开发人员的我还是有很大责任的。开发时我在对可能出现的错误进行兜底时，需要尽可能地考虑全面。

### 改进

> 服务端请求优化，错误处理添加对请求超时的处理。

① 首次访问使用SSR获取最新数据；

② 当SSR超时时自动降级到 CSR版本 ；

③ 保持首屏加载性能的同时提升可用性。
