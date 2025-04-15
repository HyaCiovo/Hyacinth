---
title: 浙里办 H5 mgop 请求封装
published: 2023-12-14
updated: 2023-12-14
description: '前段时间做了一个H5嵌入浙里办的需求，文档要求必须使用官方提供的zwlog进行埋点。网上的经验帖大部分是vue和uniapp的项目，这里记录一下自己在React项目中如何进行埋点开发。'
tags: [React, 浙里办]
category: '工作记录'
draft: false 
---
### 前言

两周前水了一篇浙里办H5 React工程埋点，有JY私信问 mgop 请求的问题。不过我们项目的请求封装不是我做的，这里放一下我自己修改后的 mgop 请求封装。

> 注意：官方文档时常更新，文中出现的版本和上报的参数很可能在某次版本迭代后需要修改，一切代码开发需要以官网最新文档为主🫤。



### 引入 ZWJSBridge

> 在 head 标签中引入 bridge 文件。 
>
> 开发时 bridge 版本为 1.1.0。

```html
<script type="text/javascript" src="https://assets.zjzwfw.gov.cn/assets/ZWJSBridge/1.1.0/zwjsbridge.js"></script>
```



### 封装 mogop 请求

> 封装 mogop 请求，从 localStorage 中取出 token 加到 haeder 中。在开发过程中我们发现 mgop 的 cookie 不会自动加到 header里，从接口返回里截取出来又比较麻烦。我们就根据之前支付宝小程序的开发经验，将 token 放到 response 中返回，前端手动将 token 存到 localStorage 中。

```ts
import { mgop } from "@aligov/jssdk-mgop";

type RequestResponse<T> = {
  code: number;
  data: T;
  message: string;
};

/**
 * 使用 mgop 发送请求
 * 使用typescript时无需填入type、dataType
 * @param api 调用的接口路径，是在irs开发者平台中注册的rpc的api名称
 * @param data 请求的params/data。
 */
function mgopRequest<T>(api: string, data?: any): Promise<RequestResponse<T>> {
  // 打印请求的接口路径和参数，便于调试
  console.info(api, "mgop请求入参:", data);
  
  // 从本地存储获取用户令牌
  const saToken = window.localStorage.getItem("saToken");
  
  // 配置对象，包含请求的基础信息
  const config = {
    host: "https://mapi.zjzwfw.gov.cn/",
    appKey: "应用appKey",
    header: {
      // 1：转发至测试环境联调；如上生产就注释掉
      // isTestUrl: "1",
      // 添加用户令牌到请求头
      satoken: saToken || "",
    },
  };
  
  // 返回一个新的Promise对象，用于处理异步请求
  return new Promise((resolve, reject) => {
    // 构造mgop请求对象
    const mgopReceiveObj = {
      api,
      data,
      ...config,
      // 请求成功的回调函数
      onSuccess: async (res: any) => {
        console.log("请求成功：", res);
        try {
          // 处理请求成功后的响应
          const result: RequestResponse<T> = await handleOnMgopSuccess(
            res,
            api
          );
          return resolve(result);
        } catch (error) {
          // 处理成功回调中的错误
          return reject(error);
        }
      },
      // 请求失败的回调函数
      onFail: async (error: any) => {
        console.error("请求失败：", error);
        try {
          // 处理请求失败后的错误
          await handleOnMgopFailed(error, api);
        } catch (error) {
          // 处理失败回调中的错误
          reject(error);
        }
      },
    };
    // 发送mgop请求
    mgop(mgopReceiveObj);
  });
}

/**
 * mgop请求成功响应拦截器：该回调方法在mgop官方请求成功时做处理。
 * 对我方接口返回的数据结构做判断，当errorCode 不为 200 的情况都会被处理成异常，200时为正常获取我方接口数据。
 * @param res mgop请求成功返回结构
 * @param api 当前请求的api
 * @returns 返回一个Promise，根据请求结果进行resolve或reject
 */
function handleOnMgopSuccess<T>(
  res: any,
  api: string
): Promise<RequestResponse<T>> {
  return new Promise((resolve, reject) => {
    // 解析响应中的错误代码和消息
    const code = parseInt(res.data?.errorCode);
    const message = res.data?.message || "无详情";
    // 代表了业务异常的情况，根据项目具体需要做处理
    if (code !== 200) {
      // 当errorCode不为200时，创建错误对象并reject
      const error = { code, message, api };
      return reject(error);
    } else {
      // 当errorCode为200时，代表请求成功，resolve响应数据
      resolve(res.data);
    }
  });
}

/**
 * mgop请求失败响应拦截器：
 * 对浙里办应用的mgop请求接口返回的数据结构做判断。
 * 这个拦截器返回的 Promise 永远是 reject，并且对返回的异常结构做了处理。
 * 手动处理成统一的结构后（包括成功响应拦截器中的业务异常）的原因是
 * mgop 在不同终端返回的结构不同，造成了开发时的心智负担。
 */
/**
 * mgop请求失败响应拦截器：
 * 这个拦截器返回的 Promise 永远是 reject，并且对返回的异常结构做了处理。
 * @param error 请求失败的错误信息
 * @param api 请求的API名称
 * @returns 返回一个永远为 reject 的 Promise，包含错误信息和API名称
 */
function handleOnMgopFailed(error: any, api: string) {
  return new Promise((resolve, reject) => {
    // 当错误信息包含 errorMessage 和 errorCode 时，构建自定义错误对象并 reject
    if (error.errorMessage && error.errorCode) {
      const { errorCode: code, errorMessage: message } = error;
      const myError = { code, message, api };
      return reject(myError);
    } else if (error?.ret[0]) {
      // 当错误信息包含 ret[0] 时，提取错误信息并构建自定义错误对象
      const message = error.ret[0];
      const code = parseInt(error.ret[0]);
      const myError = { code, message, api };
      return reject(myError);
    } else {
      // 当错误信息不足以构建自定义错误对象时，使用默认的错误信息
      const message = "未知系统异常";
      const code = 500;
      const myError = { code, message, api };
      return reject(myError);
    }
  });
}

export { mgopRequest, RequestResponse };

```



### 具体请求使用

```ts
import { mgopRequest, RequestResponse } from "./mgopRequest";

/**
 * 测试用接口，用于判断当前API访问环境。
 * @returns
 */
const getTestData = () => {
  return mgopRequest("mgop.xxx.test.doTest.COPY.SgVhTMTm");
};

/**
 * 测试用接口，用于判断当前API访问环境。
 * @returns
 */
const authTest = () => {
  return mgopRequest("mgop.xxx.test.authTest");
};

/**
 * 单点登录获取用户信息。
 * @param ticketId ticketId
 */
export const login = (data: {
  ticketId: string;
}): Promise<RequestResponse<UserInfoType>> => {
  return mgopRequest("mgop.xxx.xxx.login", data);
};

// 列表分页
export const getRecords = (
  data: queryClientType
): Promise<RequestResponse<AuthRecordType>> => {
  return mgopRequest("mgop.xxx.xxx.pageRecord", data);
};

// 信息
export const getInfo = (data: {
  authCode: string;
}): Promise<RequestResponse<InfoType>> => {
  return mgopRequest("mgop.xxx.xxx.info", data);
};

/**
 * 声明同意/拒绝（浙里办）
 * @param ifAgree 是否同意
 * @param callbackToken 安全token
 * @returns
 */
const AAA = (data: { ifAgree: boolean; callbackToken: string }) => {
  return mgopRequest("mgop.xxx.xxx.AAA", data);
};

/**
 * 取消（浙里办）
 * @param id 请求id
 * @returns
 */
const cancel = (data: {
  id: string;
}): Promise<RequestResponse<any>> => {
  return mgopRequest("mgop.xxx.xxx.cancel", data);
};

export { authTest, getTestData, getInfo, login, AAA, cancel };

```



### 获取票据 getTicket

> 封装获取 ticketId 的方法。

```ts
/**
 * 获取票据
 * @returns
 */
const ZWJSBridge = window.ZWJSBridge //记得declare一下

const getTicket = async () => {
  console.info('获取票据');
  let ticket = '';

  if (ZWJSBridge.ssoTicket) {
    const ssoFlag = await ZWJSBridge.ssoTicket({});
    if (ssoFlag && ssoFlag.result === true) {
      // 使用 浙里办统一单点登录组件
      console.info('使用 浙里办统一单点登录组件');
      if (ssoFlag.ticketId) {
        console.info('应用方服务端单点登录接口');
        ticket = ssoFlag.ticketId;
        return ticket;
      } else {
        console.warn(
          '当浙里办单点登录失败或登录态失效时调用 ZWJSBridge.openLink 方法重新获取 ticketId',
        );
        ZWJSBridge.openLink({
          type: 'reload',
        }).then((res: any) => {
          ticket = res.ticketId;
          return res.ticket;
        });
      }
    } else {
      console.error(
        '异常情况：当前环境不支持浙里办统一单点登录',
      );

      Toast.show({ content: '业务异常，请稍后再试' });
      // // 异常情况：当前环境不支持浙里办统一单点登录
    }
  } else {
    console.error(
      '异常情况：ZWJSBridge 加载异常',
    );
  }
  return ticket;
};
```



### 单点登录 ssoLogin

> 在 App.jsx 中获取 ticketId 并登录。

```tsx
// ...
const ssoLogin = useCallback(async () => {
    const ticketId = await getTicket()
    
    console.log(ticketId)
    login({ ticketId: ticketId })
      .then((res) => {
        const { saToken = '' } = res.data || {};
        window.localStorage.setItem('saToken', saToken);
        setUserInfo({ ...res.data, ticketId } as UserInfoType);
      })
      .catch(() => {
        window.localStorage.removeItem('saToken')
        clearUserInfo();
      });
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

 useEffect(() => {
    setGlobalTheme(); // 设置全局样式主题
    setIPhoneApp();
    needRecord()  // 打开埋点信息上报
    ssoLogin();
    return () => {
      window.localStorage.clear()
      document.cookie = ''
      queryClient.clear();
    };
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);
// ...
```

据说也不一定必须使用 mgop 来进行接口请求，在真机测试环境联调时，我们发现正常的 axios 也能调通，并且有听说其他老大哥的过去老项目就没走 mgop 。

