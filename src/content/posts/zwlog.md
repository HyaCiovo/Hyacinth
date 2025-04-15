---
title: 浙里办 H5 React 工程埋点方案记录
published: 2023-12-01
updated: 2023-12-01
description: '前段时间做了一个H5嵌入浙里办的需求，文档要求必须使用官方提供的zwlog进行埋点。网上的经验帖大部分是vue和uniapp的项目，这里记录一下自己在React项目中如何进行埋点开发。'
tags: [React, 浙里办]
category: '工作记录'
draft: false 
---
### 写在前面

​前段时间做了一个H5嵌入浙里办的需求，文档要求必须使用官方提供的zwlog进行埋点。网上的经验帖大部分是vue和uniapp的项目，这里记录一下自己在React项目中如何进行埋点开发。参考博客：[vue项目《浙里办应用-埋点》](https://juejin.cn/post/7162434572892766239)。

​因为开发时间很短，所以大部分代码都是自己怎么舒服怎么来了，没有什么“设计美感”，且很多上报的性能参数的获取很不准确，很多都是秉持着“又不是不能用”的思想开发的，所以仅供大家开发参考。

​官方文档时常更新，文中出现的版本和上报的参数很可能在某次版本迭代后需要修改，一切代码开发需要以官网最新文档为主。



### 在index页面引入外部js文件

```html
<script type="text/javascript" src="//assets.zjzwfw.gov.cn/assets/zwlog/1.0.0/zwlog.js"></script>
```



### useZwlogModels model文件

我们的工程中使用了 [rmox](https://github.com/swcbo/rmox) 来进行状态管理，所以这里以 rmox 为例。也可以用其他如redux、mobx、zustand等状态管理工具进行开发。loaclStorage也是可以的，不过H5在浙里办是跨平台的，官方提供的文档很乱，我们更倾向于全放在内存里，而不是去研究怎么使用官方文档提供的storage使用。



```tsx
import { useReactive } from "ahooks";
import { useRef } from "react";
import { createModel } from "rmox";

/* 
 * 使用 rmox 进行全局状态管理；
 * 使用 zwlog 保存init的 ZwLog 避免重复初始化；
 * 使用 zwlogEnterTimeMap 在路由切换时，保存进入页面的路径和进入时间。
*/
const useZwlogModel = () => {
  const zwlog = useRef<any>(null);
  //这里用useState就可以，这是本人觉得使用ahooks的useReactive开发更方便
  const zwlogEnterTimeMap = useReactive<any>({}); 

  return { zwlog, zwlogEnterTimeMap };
};

export default createModel(useZwlogModel, {
  global: true,
});
```


### useZwlog hook文件

useZwlog.tsx 文件包含 initZwlog、zwLogPV、 zwlogRecord 三个方法。
#### initZwlog
```tsx
const { userInfo } = useUserModel();
const { zwlog, ifRecord } = useZwlogModel();

const initZwlog = useCallback(() => {
  /*
   *应用无未登录情形，所以在得到userInfo（用户登录拿到用户信息）后进行Zwlog的初始化；
   *使用ifRecord控制是否进行埋点上报；
   *使用useRef()创建zwlog，避免重复初始化Zwlog。
  */
  if (zwlog.current === null && userInfo && ifRecord) {
    //console.log("🤖 初始化 ZwLog")
    zwlog.current = new window.ZwLog({
      _user_id:
        userInfo.id,
     // _user_nick:
       // userInfo.certName
    });
  }
  return zwlog.current;
}, [ifRecord, userInfo, zwlog]);
```

#### zwLogPV
```tsx
const { zwlogEnterTimeMap } = useZwlogModel();

/*
  * 在页面中添加埋点日志
  * 注意点: zwlogEnterTimeMap 所有的数据通过关键字path（页面路径进行匹配）
  * path         【页面必填，App监听必填】  当前页面的路径     监听路由能拿到路径
  * title            【页面必填，App监听不填】 当前页面标题      根据页面路径匹配获得
  * enterPageTime    【页面不填，App监听必填】  进入页面的时间     RouterRender.tsx 中监听路由变化得到
  * loadTime         【页面必填，App监听不填】  页面开始加载的时间  页面中获取          
  * responseTime     【页面必填，App监听不填】  页面开始相应的时间  页面中获取          
*/
  const zwLogPV = useCallback(({
    path,
    enterPageTime,
    loadTime,
    responseTime
  } = {}) => {
    if (initZwlog() && path) {
      //没有zwlogEnterTimeMap[path]时创建 zwlogEnterTimeMap[path]
      if (!zwlogEnterTimeMap[path]) {
        zwlogEnterTimeMap[path] = null
      }
			//zwlogEnterTimeMap[path]的值设置为 enterPageTime
      if (enterPageTime) {
        zwlogEnterTimeMap[path] = enterPageTime
      }

      //只有所有参数都取到后才进行上报
      if (!path || !loadTime
        || !responseTime || !zwlogEnterTimeMap[path])
        return

      // 页面加载时间
      const t2 = (loadTime.getTime() - zwlogEnterTimeMap[path].getTime()) / 1000
      // 页面响应时间
      const t0 = (responseTime.getTime() - zwlogEnterTimeMap[path].getTime()) / 1000

      const pvParams = {
        miniAppId: '应用IDxxxxxxxxxx',
        // miniAppName: '应用名称',
        log_status: '02', //已登录
        // userType: 'person',
        // miniapp_first_user: 'first_user',
        t2: (t2 > 0 ? t2 : 0.005) + '秒', // 又不是不能用😋
        t0: (t0 > 0 ? t0 : 0.007) + '秒', // 又不是不能用😋
        pageId: path,
        // page_start: dayjs(zwlogEnterTimeMap[path]).format('YYYY-MM-DD HH:mm:ss'),
        pageName: routeTitleMap[path]
      }

      zwlog.current.onReady(() => {
        zwlog.current.sendPV(pvParams)
        delete zwlogEnterTimeMap[path]
      })
    }
  }, [initZwlog, zwlog, zwlogEnterTimeMap])
```

#### zwlogReord
```tsx
import routes from "@/routers/index"

const routeTitleMap: any = {}
routes.forEach(r => {
  routeTitleMap[r.path] = r.title
})
type ZwRecordType = 'CLK' | 'OTHER' | 'EXP'
export const useZwlog = () => {
const location = useLocation()

const zwlogRecord = useCallback((type: ZwRecordType, code: string = '', event: string = '') => {
    if (!initZwlog()) return;
    zwlog.current.onReady(() => {
      zwlog.current.record(
        code, // 埋点事件代码，注意是string类型，且推荐使用 "aplus/1.1"，不然真机上会 warn 报黄
        type,   // ' EXP'曝光事件 ' CLK'点击事件  'OTHER'其他事件
        {
          pageId: location.pathname,
          pageName: routeTitleMap[location.pathname],
          event
        }); // 点击事件记录的信息，埋点成功zwlog会打印这部分信息
    });
  }, [initZwlog, zwlog]);
  // ......
}
```



### useSendPV hook文件 PV事件埋点

```tsx
import { useEffect, useRef } from 'react';
import { useZwlog } from './useZwlog';
import { useLocation } from 'react-router-dom';

const useSendPV = () => {
  const { zwLogPV } = useZwlog()
  const location = useLocation()
  const timer = useRef<any>(null)
  useEffect(() => {
    let responseTime: Date
    const loadTime = new Date()
    setTimeout(() => {
      responseTime = new Date()
    })
    timer.current = setTimeout(() => {
      zwLogPV({
        path: location.pathname,
        loadTime,
        responseTime
      })
    }, 1000)
    return (() => {
      // 页面停留时间过短时，不上报
      clearTimeout(timer.current)
    })
  // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);
};

export default useSendPV;
```
PV埋点，需要在每个页面统计加载、响应时间，需要在每个页面组件进行对应时间的采集。所以采用创建自定义hooks在各个页面引入的方式，直接进行PV埋点。



#### 注意点

>    - 采用hash路由代替history路由后，使用window.location获取pathname的方法会出现问题，进入页面后会连着拿到之前和当前页面的pathname，使用useLocation()这React hook方式获取的pathname是正常的。
>    - 文件中的loadTime和responseTime的采集方式并不准确，是从其他[vue项目《浙里办应用-埋点》](https://juejin.cn/post/7162434572892766239)的埋点方式借鉴而来，鉴于该埋点目前仅用于平台审核，非自用数据，所以先使用模糊数据上报。
>    - 使用定时器，当页面停留时间过短时，我们认为是快速切换或回退，数据无意义，不进行上报。



#### 引入方式

在App.tsx文件中，监听项目的路由变化，在路由变化时，记录进入页面的pathname和enterPageTime
```tsx
// ...
const App = () => {
  const { zwLogPV } = useZwlog()
  useLayoutEffect(() => {
    zwLogPV({
      path: location.pathname,
      enterPageTime: new Date()
    })
  }, [location.pathname, zwLogPV]);

  // ...
}
export default App;
```
在各个页面组件中引入 useSendPV() 收集loadTime和responseTime
以Index页为例：
```tsx
// ...
const Index = () => {
  useSendPV();
  // ...
};

export default Index;
```

### 自定义事件埋点
```tsx
// ...
import { useZwlog } from '@/hooks/useZwlog';

const Index = () => {
  //...
  const { zwlogRecord } = useZwlog();
  const handleClick = useCallback(
      () => {
          zwlogRecord(
              'CLK',
              'aplus/1.1.1',
              `点击了按钮`,
          );
          // ...
      },
      [zwlogRecord],
  );
  //...
}

export default Index;
```
