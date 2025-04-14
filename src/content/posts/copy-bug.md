---
title: 产品经理：“一个简单的复制功能也能写出bug？”
published: 2023-12-29
updated: 2023-12-29
description: '刚入职时，遇到了一个线上 bug，用户点击复制按钮没办法复制文本，产品经理震怒，“这么简单的一个功能也能出问题？当时是谁验收的？”，因为我刚来还闲着，就把我派去解决这个问题。'
tags: [React]
category: 'Work'
draft: false 
---

## 问题

刚入职时，遇到了一个线上 bug，用户点击复制按钮没办法复制文本，产品经理震怒，“这么简单的一个功能也能出问题？当时是谁验收的？”，因为我刚来还闲着，就把我派去解决这个问题。

我在排查问题时，发现该复制方法写在了一个自定义 hook 中（这里复制方法写在 hook 里没啥意义，但是乙方交付过来的代码好像特别喜欢把工具函数写成个 hook 来用），点进去查看就是简单的一个 `navigator.clipboard.writeText()`的方法，本地运行我又能复制成功。于是我怀疑是手机浏览器不支持这个 api 便去搜索了一下。



## Clipboard

MDN 上的解释：

剪贴板 [Clipboard API](https://developer.mozilla.org/zh-CN/docs/Web/API/Clipboard_API) 为 **[`Navigator`](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator)** 接口添加了只读属性 **`clipboard`**，该属性返回一个可以读写剪切板内容的 [`Clipboard`](https://developer.mozilla.org/zh-CN/docs/Web/API/Clipboard) 对象。在 Web 应用中，剪切板 API 可用于实现剪切、复制、粘贴的功能。

只有在用户事先授予网站或应用对剪切板的访问许可之后，才能使用异步剪切板读写方法。许可操作必须通过取得权限 [Permissions API (en-US)](https://developer.mozilla.org/en-US/docs/Web/API/Permissions_API) 的 `"clipboard-read"` 和/或 `"clipboard-write"` 项获得。

**浏览器兼容性**


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f38130c05a6d4653a0850944344aec00~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=960&h=399&s=42147&e=png&b=fefefe)



## 使用 document.execCommand() 降级处理

这里我也不清楚用户手机浏览器的版本是多少，那么这个 api 出现之前，是用的什么方法呢？总是可以 polyfill 降级处理的吧！于是我就查到了`document.execCommand()`这个方法：

- document.execCommand("copy") : 复制；
- document.execCommand("cut") : 剪切；
- document.execCommand("paste") : 粘贴。



## 对比

Clipboard 的所有方法都是异步的，返回 `Promise` 对象，复制较大数据时不会造成页面卡顿。但是其支持的浏览器版本较新，且只允许 https 和 localhost 这些安全网络环境可以使用，限制较多。

document.execCommand() 限制较少，使用起来相对麻烦。但是 MDN 上提到该 api 已经废弃：


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e86d3eca78b44fd5951c148ad08ed885~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=969&h=235&s=62908&e=png&b=f9e4e9)


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e8f2f29ec984ac683d7ecf6aba39c88~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=921&h=697&s=64659&e=png&b=fdfdfd)

浏览器很可能在某个版本弃用该 api ，不过当前 2023/12/29 ，该复制 api 还是可以正常使用的。



## 具体代码修改

于是我修改了一下原来的 hook：


```tsx
import Toast from "~@/components/Toast";

export const useCopy = () => { 
    
  const copy = async (text: string, toast?: string) => {
      
    const fallbackCopyTextToClipboard = (text: string, toast?: string) => {
      let textArea = document.createElement("textarea");
      textArea.value = text;

      // Avoid scrolling to bottom
      textArea.style.top = "-200";
      textArea.style.left = "0";
      textArea.style.position = "fixed";
      textArea.style.opacity = "0"

      document.body.appendChild(textArea);
      // textArea.focus();
      textArea.select();
      let msg;
      try {
        let successful = document.execCommand("copy");
        msg = successful ? toast ? toast : "复制成功" : "复制失败";
      } catch (err) {
        msg = "复制失败";
      }
      Toast.dispatch({
        content: msg,
      });
      document.body.removeChild(textArea);
    }; 
      
    const copyTextToClipboard = (text: string, toast?: string) => {
      if (!navigator.clipboard || !window.isSecureContext) {
        fallbackCopyTextToClipboard(text, toast);
        return;
      }
      navigator.clipboard
        .writeText(text)
        .then(() => {
          Toast.dispatch({
            content: toast ? toast : "复制成功",
          });
        })
        .catch(() => {
          fallbackCopyTextToClipboard(text, toast)
        });
    };
    copyTextToClipboard(text, toast);
  };

  return copy;
};
```

上线近一年，这个复制方法没出现异常问题。



