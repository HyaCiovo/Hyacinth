---
title: Taro 数字滚动组件（支持React/Solid）
published: 2025-04-12
updated: 2025-04-12
description: 'Taro-CountUp 是一款简单易用的Taro数字滚动组件，支持React和Solid.js框架写法。该组件适用于平滑滚动数字到指定值的场景，支持自定义滚动时间、缓动效果、小数位数显示等功能。'
tags: [Taro,React,Solid.js,开源]
category: 'Work'
draft: false 
---

## 更新日志

**2025/4/13**： 支持`Solid.js`框架写法。

## 背景

作者前段时间在搭建公司的小程序模板仓库 [taro3.x + tailwindcss + zustand 小程序开发 | 掘金](https://juejin.cn/post/7435887935751716898)。结果发现了一些项目所用组件库 [@antmjs/vantui](https://github.com/AntmJS/vantui) 的`bug`。当时自己试着修复一下提下`pr`，结果维护团队响应太快了🤣自己修好上去了。自己就这么失去了第一次提`pr`的机会......不服气的我就去翻了`issue`，想着一定得提一个上去，结果让我看到了 [可以增加一个数字滚动组件么 · Issue #632](https://github.com/AntmJS/vantui/issues/632)
![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/b3fc44f543c244fa985cf2dd4c633daa~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745226908&x-orig-sign=u%2BTunV4doJfZb3776yNzYiTv8fI%3D)

我祁厅长都发话了，一年了都没给落实吗😡？于是我简单了解了一下，发现现有的`Taro`组件库好像都没找到有数字滚动这个组件。最后实在隔壁家的 [uView | CountTo 数字滚动](https://uviewui.com/components/countTo.html) 找到了。

![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/aa6f63785ded480592d3674e614f8f09~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745226908&x-orig-sign=%2FSs8tGKZsBFlX%2B071GGdmITawfY%3D)
`uni`有的，我们`taro`怎么能没有。看了一下`API`，和现在流行的数字滚动组件类似，都是基于 [CountUp.js](https://inorganik.github.io/countUp.js/) 设计的。原本看`CountUp.js`的源码功能很多，无从下手。这下有了`uView`的现成作业抄就简单多了🫡。
于是我总算是完成了我这“艰难”的`pr之旅`。下文介绍的是我单独抽出来的组件，可以直接复制代码到自己的项目中使用。

## 简介

`Taro-Countup-React`是一个基于`@tarojs/components`封装的支持`React/Solid.js`框架写法的`Taro`数字滚动组件。

该组件适用于需要平滑滚动数字到指定值的场景，支持自定义滚动时间、缓动效果、小数位数显示、千分位分隔符等功能。通过 `ref` 选择器，你可以方便地控制组件的开始、暂停、继续和重置操作。

API设计和实现原理参考自 [react-countup](https://github.com/glennreyes/react-countup) 和 [uView | CountTo 数字滚动](https://uviewui.com/components/countTo.html)，可以直接复制到`Taro v3/v4`项目中使用。因为完全基于Taro自带的组件库开发，理论上支持`Taro`支持的所有端（不过作者只测试了`h5、微信和支付宝小程序`）。

## 代码仓库

[HyaCiovo/taro-countup](https://github.com/HyaCiovo/taro-countup)

> 请切换分支查看不同框架写法的组件
>
> ![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/b20f9b704408404d94a5ec16c4a6f6cb~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745226908&x-orig-sign=Yp2cZL%2FH7v0PflN8o5c8VdFvoiQ%3D)

## 效果

![a18c1e2e0621bcef2ee8abbdfcd1ae3.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/b0c1371d37ae4823a2e50cf04cc34602~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745226908&x-orig-sign=LhOX7HKj1BQhB9VEuGEyRUJGQq4%3D)

## 在线演示

作者懒得搞，等什么时候`pr`过了✅，会把组件库演示文档上的链接放过来😁。

## 代码演示

### 基本用法

通过`startVal`设置开始值，`endVal`设置结束值

```tsx
<CountUp startVal={30} endVal={500} />
```

### 设置滚动相关参数

*   通过`duration`设置从开始直到结束值整个滚动过程所需的时间，单位毫秒
*   通过`useEasing`设置在滚动快结束的时候，是否放慢滚动的速度，给用户更好的视觉效果

```tsx
<CountUp startVal={30} endVal={500} duration={2000} useEasing={false} />
```

### 是否显示小数位

通过`decimals`设置显示的小数位，在滚动过程中，小数位会一起变化。如果`startVal`和`endVal`是带小数的，应该设置`decimals`为`startVal`和`endVal`一样的小数位数值，如`endVal`为 `500.55`，那么`decimals`应该设置为`2`。

```tsx
<CountUp startVal={30} endVal={500.55} decimals={2} />
```

### 千分位分隔符

通过`separator`配置千分位分隔符，默认为空字符串，可以设置英文逗号`","`，此参数表现为`endVal`值超过`1000`时，比如为`"1234"`，那么会变成`"1,234"`。

```tsx
<CountUp endVal={1500} separator=',' />
```

### 手动控制

通过 `ref` 选择器获取到组件实例后，可以调用`start`、`pause`、`resume`、`reset`方法。

```tsx
/* eslint-disable */
import react from 'react'
import { View, Button } from '@tarojs/components'

export default function Demo() {
  const CountToRef = react.useRef<any>()
  const handleFinish = () => {
    console.log('count finish')
  }

  return (
    <View>
      <CountTo
        startVal={30}
        endVal={500}
        ref={CountToRef}
        onFinish={handleFinish}
      />
      <View>
        <Button onClick={() => CountToRef.current?.start()} >开始</Button>
        <Button onClick={() => CountToRef.current?.pause()} >暂停</Button>
        <Button onClick={() => CountToRef.current?.resume()} >继续</Button>
        <Button onClick={() => CountToRef.current?.reset()} >重置</Button>
      </View>
    </View>
  )
}

```

### 效果

## API

### ICountUpRef

| 参数     | 说明                           | 类型                 |
| ------ | ---------------------------- | ------------------ |
| start  | autoplay 为 false 时，通过此方法启动滚动 |   () =\> void<br/> |
| pause  | 暂停滚动                         |   () =\> void<br/> |
| resume | 暂停后重新开始滚动(从暂停前的值开始滚动)        |   () =\> void<br/> |
| reset  | 重设至初始值                       |   () =\> void<br/> |

### CountUpProps

| 参数        | 说明             | 类型                        | 默认值  | 必填      |
| --------- | -------------- | ------------------------- | ---- | ------- |
| startVal  | 滚动开始值          |   number<br/>             | 0    | `false` |
| endVal    | 滚动结束值          |   number<br/>             | 0    | `false` |
| duration  | 滚动过程所需的时间，单位毫秒 |   number<br/>             | 3000 | `false` |
| autoStart | 是否自动开始滚动       |   boolean<br/>            | true | `false` |
| decimals  | 要显示的小数位数       |   number ¦ string<br/>    | 0    | `false` |
| decimal   | 十进制分隔          |   string<br/>             | .    | `false` |
| prefix    | 前缀             |   string ¦ ReactNode<br/> | -    | `false` |
| suffix    | 后缀             |   string ¦ ReactNode<br/> | -    | `false` |
| useEasing | 是否缓动结束滚动       |   boolean<br/>            | true | `false` |
| separator | 千分位分隔符         |   string<br/>             | -    | `false` |
| onFinish  | 滚动结束时触发        |   () =\> void<br/>        | -    | `false` |
| ref       | 数字滚动实例         | any<br/>                  | -    | `false` |

## 最后

希望我的组件能帮到大家，欢迎各位 **JY** 来[仓库](https://github.com/HyaCiovo/taro-countup-react)点点 **star**✨ 、提提 **pr**💫！
