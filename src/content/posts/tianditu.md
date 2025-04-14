---
title: React 项目引入天地图 API
published: 2024-01-09
updated: 2024-01-09
description: '最近新接到了一个需求，要把公司官网的地图组件改成天地图的地图，本文React（Next）项目引入天地图API的开发过程做一下简单记录。'
tags: [React,Next]
category: 'Work'
draft: false 
---

## 介绍
最近新接到了一个需求，要把公司官网的地图组件改成[天地图](http://lbs.tianditu.gov.cn/home.html)的地图，本文对自己的开发过程做一下简单记录。

## 注册天地图账户
首先到[天地图](http://lbs.tianditu.gov.cn/home.html)官网注册账户，并创建应用。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/114828cb587343c8abd35639c7a96df6~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1540&h=361&s=28987&e=png&b=fbfbfb)

## 设置域名白名单
到应用设置中添加域名白名单。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0206473267074b0b9de97005dfccc719~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=741&h=438&s=40716&e=png&b=fffefe)

## 通过 script 标签将 API 引用到页面中
因为我们的项目使用的是 Next.js 框架，这里以 Next.js 的引入为例。

- 为了不在本地 localhost 开发中的控制台报错，选择在 tianditu.js 文件中根据网页地址动态引入天地图的 API。
```js
(function () {
  if (!window.location.href.includes("xxx.cn"))
    return;
  const f = document.getElementsByTagName('script')[0];
  const j = document.createElement('script');
  j.async = true;
  j.id = 'tianditu';
  j.type = 'text/javascript';
  j.src = "https://api.tianditu.gov.cn/api?v=4.0&tk=你的应用Key";
  f.parentNode.insertBefore(j, f);
})();
```
-  在 _document.tsx 文件中引入 script。
```tsx
<script id="tianditumap" src="/js/tianditu.js" />
```

## 页面代码设置
在需要引入地图组件的页面 init 地图组件。
```tsx
import React, { useEffect, useState } from 'react';
import { Spin } from 'antd';

const About = () => {
  const [loading, setLoading] = useState<boolean>(true);
  const [SecureEnv, setEnv] = useState<boolean>(false);
  const [destory, setDestory] = useState<boolean>(false);
  const initMap = () => {
    setLoading(false);
    if (!window.T) return;
    // 初始化地图对象
    const map = new window.T.Map('map');
    const lngLat = new window.T.LngLat('经度', '纬度');
    // 设置显示地图的中心点和级别
    map.centerAndZoom(lngLat, 18);
    // 允许鼠标滚轮缩放地图
    map.enableScrollWheelZoom();
    // 创建标注对象
    const marker = new window.T.Marker(lngLat);
    // 向地图上添加标注
    map.addOverLay(marker);
      
    // 如果要给标注添加点击事件，例如：跳转高德地图，建议用useRef设置marker，并在useEffect
    // 的return中销毁事件监听器
    // marker.addEventLisner("click",()=>{
    //   window.open("...")
    // })
      
    // 创建信息窗口对象
    const label = new window.T.Label({
      text: '文字标注介绍',
      position: lngLat,
      offset: new window.T.Point(5, -30),
    });
    map.addOverLay(label);
  };

  useEffect(() => {
    const isSecure = window.location.href.includes('xxx.cn');
    if (isSecure) {
      setEnv(isSecure);
      // 有时直接刷新当前页面会找不到window.T.Map，添加setTimeout可以解决这个问题
      setTimeout(() => {
        initMap();
      }, 500);
    }
    return () => {
      setDestory(true);
    };
  }, []);

  return (
    <div>
          // ...
        {!destory && (
          <Spin spinning={loading}>
            <div className={`w-full z-0 ${SecureEnv && 'h-[340px] mt-8'}`} id="map" />
          </Spin>
        )}
          // ...
    </div>
  );
};

export default About;
```

地图的引入方法比较简单，将`div`的`id`作为`window.T.Map()`方法的参数传入，并设置中心点经纬度和缩放级别即可。

其他标注点、文字标注、事件等可自行查阅[官方文档](http://lbs.tianditu.gov.cn/api/js4.0/examples.html)。

> 需要注意，用来初始化地图的 div 的 id 需要避免与其他标签 id 重复。作者在开发时，将 script 标签的 id 和 div 标签的 id 都设置成了 "map"😹，导致地图一直没渲染到页面上，排查了许久才发现问题。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81f0d97363eb4b3982df541080cd85cc~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1911&h=964&s=622850&e=png&b=f8f5f4)

## 效果
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe925f901a944ea18b658ddf7a167af5~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1089&h=381&s=139482&e=png&b=f8f5f2)





