---
title: 用油猴脚本扒表单，又体会了一下就业“寒气”🥶
published: 2024-01-15
updated: 2024-01-15
description: '闲来无事，翻了翻公司官网，看到新一年的校招名单。就想着看看这群新打工人们的情况，扒了一下名单，不由得被这些“未来同事”们的 background 惊到了😧'
tags: [Go,Tampermonkey]
category: 'Learning'
draft: false 
---

## 前言
闲来无事，翻了翻公司官网，看到新的一年的校招名单。就想着看看新打工人们的情况，一眼过去全是各种名校硕士😧，想想两年前自己进来的时候还是一群双非本科🫤，真是感慨啊，现在的就业环境太差了。但是在网页上看不太直观，没法像 excel 那样筛选。于是想着能不能把数据扒下来放到 excel 里操作一下。

## 尝试扒接口

第一想法肯定是扒接口，直接从接口里把数据拿出来生成一个 csv 文件不就大功告成了！于是我 F12 看了一眼 network，“什么！空的！”，看了一眼地址栏，嚯，`jsp` ,原来不是 SPA 的页面，用 jsp 写的啊，有点在我知识盲区了，那么直接爬 html 吧！


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6060550a06c4c6eb4e91d4b52fbad84~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1006&h=816&s=48051&e=png&b=fdfdfd)

## Go Colly 小试牛刀

说干就干，去年学习 Golang 的时候，我了解了一下  Go 的 Colly 爬虫框架，于是试着做了一下。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e4b98e8dcf24723a1a0fe5f385e0b35~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1919&h=1079&s=197695&e=png&b=262b32)

运行后，确实可以拿到一个 csv 文件，格式还需要调整，还算满意。但是我感觉以后应该还会去扒😎，每次扒都要进去查 id ，再运行一遍程序好麻烦。后面还尝试了一下直接扒公告列表拿到所有公告 id ，全部扒一遍，但是由于公告格式都不太一样，没法很方便的一次性拿下来，而且每次运行都爬一遍也不太合理。有些公告我也不一定想爬下来，要是能像我们后台一样在页面上有个导出按钮就好了！

哎？那不就是往页面上插个有导出功能的脚本吗！以前刷学习通网课时，油猴脚本不就能干这个事情吗！

## 最终选择油猴脚本

油猴脚本很简单，就是写个 JavaScript 脚本运行在篡改猴当中，记得配置好对应的地址匹配就好了，毕竟不可能让咱们的脚本运行在每个页面上🐔。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69785fa8c6b8493ab0cacb04b3b388af~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=465&h=273&s=17706&e=png&b=fbfbfb)


写个简单的 JavaScript 脚本，对于我这个职业前端那就是手到擒来了。代码如下：

```javascript
// ==UserScript==
// @name         导出xxx公示数据
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  Hello Tampermonkey!
// @author       HyaCinth
// @match        http://xxx.cn/jsp/xxx/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
    console.log("脚本加载了")
    const title = document.getElementsByClassName("header")[0].innerText //文件标题
    const trs = document.getElementsByTagName("tr")
    // console.log(trs[1].innerText.split("\t"))
    let contentList = []
    for (let i = 0; i < trs.length; i++) {
        contentList.push(trs[i].innerText.split("\t"))
    }
    let tableData = []
    contentList.forEach((item) => {
        // 遍历 table 的每一行，长度为 1 的是标题，不写入 tableData
        if (item.length !== 1) 
        {tableData.push({
            order: item[0], // 序号
            name: item[1], // 姓名
            native: item[2], // 籍贯
            qualification: item[3], //学历
            school: item[4], // 学校
            subject: item[5], // 专业
            post: item[6] //岗位
        })}
    })
    const downLoadExcel = (data, fileName) => {
        //定义表头
        //let str = trs[1].innerText.split("\t").join(",") + "\n";
        let str = "";
        //增加\t为了不让表格显示科学计数法或者其他格式
        for (let i = 0; i < data.length; i++) {
            for (let item in data[i]) {
                str += `${data[i][item] + '\t'},`;
            }
            str += '\n';
        }
        //encodeURIComponent解决中文乱码
        let uri = 'data:text/csv;charset=utf-8,\ufeff' + encodeURIComponent(str);
        //通过创建a标签实现
        let link = document.createElement("a");
        link.href = uri;
        //对下载的文件命名
        link.download = `${fileName || '录取数据'}.csv`;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }

    let button = document.createElement("button"); 
    button.id = "id001";
    button.textContent = "导出当前Excel";
    button.style.width = "235px";
    button.style.color = "#ffffff";
    button.style.marginBottom = "20px";
    button.style.fontSize="24px";
    button.style.borderRadius="8px";
    button.style.border="3px solid #337ab7"
    button.style.backgroundColor="#337ab7"
    button.style.height = "48px";
    button.style.align = "center";

    button.onclick = () => downLoadExcel(tableData,title)
    const x = document.getElementsByTagName("table")[0];
    x.parentNode.insertBefore(button,x)
})();
```

找到公示页 table 标签，在上方插入一个“导出当前Excel”按钮，点击后就可以直接导出该 table 的内容。


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83565b8a43594a1289b78dab1950693c~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1864&h=988&s=657562&e=png&b=fefefe)

点击导出按钮，成功下载了公告 csv 文件。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/778ff2cad2514f4e9ff7338a26c9b385~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=541&h=411&s=21763&e=png&b=fcfcfc)


## 啊 ？

导出了文件，当然要看看效果了，让我来筛选看看， WPS 启动！

打开该 csv 文件，点击学历，点击筛选 ......

“啊？😧”


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3bd2c52958164b2597ad9ef284f77f5d~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=536&h=352&s=25371&e=png&b=f8f7f7)

