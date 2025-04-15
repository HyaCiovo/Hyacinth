---
title: 能用就先别动😣，记一次奇怪的bug修复经历
published: 2024-01-29
updated: 2024-01-29
description: '记一次因为在开发时顺手做的样式缺陷优化，导致同事开发的新feature出现bug。能用就先别动代码，修改整体样式代码时，不仅要考虑已有页面，还要考虑到同事并行开发的页面。'
tags: [React]
category: 'Work'
draft: false 
---

## 发现问题

前段时间在迭代公司后台项目时，发现某几个页面滚动溢出了：


![597cfda6bd676dadd4ca246909c7e38.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/849530e1822c4b2294a74dc174415560~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1916&h=968&s=122563&e=jpg&b=fefdfd)


![b4e4d6cf25fd8f5443b1e18c0979a66.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0007bd3d81b54d2992cdc00992f9b0cd~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1916&h=972&s=118843&e=jpg&b=fdfcfc)

如图紫色方框中页面底部出现空白，且fixed定位的按钮也很乱。

## 顺手解决

打开控制台发现在 body 处就已经溢出了，当时没有多想这是什么原因造成的，自以为后台项目应该就是这些页面 table 列表展示，给 body 加个溢出样式就好了，不会影响整体页面展示。

```less
body {
	overflow: hidden;
}
```

于是我顺手就给这个bug“修复”了。

后面测试时，测试同学还专门问我什么时候把这个问题给修了，挺好的👍，因为这个问题之前就有，只是因为 “**后台项目而已，又不是不能用**” 没有提给开发同学🤣。

## 又有问题

但是没想到，这两天因为我加的这个`overflow: hidden`给另外的前端需求给搞出bug了。

原来和我当初迭代并行开发的一个 feature 做了一个页面预览功能，整个页面是要去溢出滚动的，但是因为是并行开发，当时没被我加上这个`hidden`，后面测试同学可能也没注意，两个 feature 都合并了之后出现了 bug，预览页面没法滚动了！😥

由于项目紧急，我当时修的这个 bug 也不影响使用，就把我修的 bug 作为新 bug 给修回去了😓。

## 问题排查

没办法，我就要去找找为啥页面会滚动了。

我先是认为部分元素的高度影响了页面，但挨个元素审查了一遍后并没有发现什么异常。

然后我发现项目刷新后并没有直接溢出滚动，且只有在部分页面向下滑动时才会出现这个现象，出现该现象后切换到其他页面，滚动不会消失。那会不会是某些页面给添加了某粒“老鼠屎”，后面又没有去干掉它呢，于是我又去仔细看了第一次产生溢出的页面的dom，发现这里添加了一个`iframe`元素。


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8513e1aada7743799c8c03c078742c5b~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1046&h=104&s=18763&e=png&b=f7f7f7)

切换页面后，这个`iframe`一直还在页面上。看来**罪魁祸首**就是他了！

在代码里查了一下，发现这个`iframe`是用来进行文件下载时引入的。

```js
/**
 * Use iframe to download file.
 * Usage:
 *  downloadFile('/abc.xml');
 */
import Modal from 'common/modal';

const iframe = document.createElement('iframe');
iframe.setAttribute('frameborder', 0);
iframe.setAttribute('style', 'width:0;height:0;opacity:0;');
document.getElementsByTagName('body')[0].appendChild(iframe);

iframe.onload = function() {
  let text;
  try {
    const body = (
      (iframe.contentWindow && iframe.contentWindow.document) ||
      iframe.contentDocument.document ||
      iframe.contentDocument
    ).getElementsByTagName('body');
    text = body.textContent || body.innerText;
  } catch (e) {
    window.location.href = iframe.src;
  }
  let error;
  try {
    const json = JSON.parse(text);
    error = json.error || json.message || json;
  } catch (e) {
    error = text;
  }
  if (text) {
    Modal.error({ content: error, title: '文件下载失败!' });
  }
};

export default function downloadFile(url) {
  iframe.src = url;
}
```

应该是为了避免频繁创建和销毁 dom，所以在第一次引入下载时创建`iframe`元素，后面不再进行销毁了。



## 最终解决

于是我试着给这个`iframe`添加了`display: none`，页面滚动溢出问题果然解决了！

但是明明这个方法里给`iframe`设置了`height: 0`，为什么还会把页面给搞溢出呢🫤？



查阅文章[《iframe 高度设置为0时还有占位_iframe占位》](https://blog.csdn.net/you227/article/details/120523359)后发现：

`iframe`是一个内联元素，默认是跟baseline对齐的，iframe后边有个看不见、摸不着的行内空白节点，空白节点占据着高度，iframe与空白节点的基线对齐，导致了div被撑开，从而出现滚动条，查看空白节点捣鬼。

解决方案：

1. 设置 iframe 的 vertical-align: top；
2. 设置父 div 的 font-size: 0，从而影响空白节点的 line-height 是0，从而不占据高度；
3. 改变 iframe 的内联元素性质，改为块级元素，display: block。



我又查阅了一下关于使用`iframe`进行文件下载的文章，发现基本都是设置`iframe`的样式为`display: none`。

便采用`display: none`来解决了因为`iframe`空白占位导致的页面异常溢出滚动的问题🫡。