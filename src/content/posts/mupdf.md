---
title: Tauri+MuPDF 实现 pdf 文件裁剪，侄子再也不用等打印试卷了🤓
published: 2024-12-23
updated: 2024-12-23
description: '表哥最近经常找我给我侄子的试卷pdf 文件 A3 转 A4（因为他家打印机只有A4 纸）。为了省事，我用Tauri + MuPDF给他开发了一个pdf裁切小工具✨。'
tags: [Tauri,Assembly]
image: https://github.com/HyaCiovo/MuPdf-Crop-Kit/raw/main/src/assets/logo.png
hideCover: true
category: '开源'
draft: false 
---


> 基于`MuPDF.js`实现的 PDF 文件 A3 转 A4 小工具。（其实就是纵切分成2份🤓）

### 开发背景

表哥最近经常找我给我侄子的试卷 `pdf` 文件 A3 转 A4（因为他家只有 A4 纸，直接打印字太小了）。

> ![018963927d97b70ab63347bab8790d3.jpg](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/482372605be84c4685258418f5dc2691~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228339&x-orig-sign=lElNTTMwnNnzvVz4hJ0sI5Vvv6A%3D)

`WPS`提供了`pdf`的分割工具，不过这是会员功能，我也不是总能在电脑前操作。于是我想着直接写一个小工具，拿`Tauri`打包成桌面应用就好了。

> ![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/130e809987764038b536ef8b6703d2d7~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228339&x-orig-sign=3mu5CvjvT27zWJ4QZfaqir8dohk%3D)

在掘金里刷到了[柒八九](https://juejin.cn/user/1433418892322119/posts)大佬的文章：[Rust 赋能前端：PDF 分页/关键词标注/转图片/抽取文本/抽取图片/翻转...](https://juejin.cn/post/7410632177824366618) 。发现`MuPDF.js`这个包有截取`pdf`文件的`API`，并且提供了编译好的`wasm`文件，这意味着可以在浏览器中直接体验到更高的裁切性能，于是我果断选择了基于`MuPDF`开发我的小工具。

### 项目简介

`MuPDF-Crop-Kit`是一个基于`MuPDF.js`、`React`、`Vite`和`Tauri`开发的小工具，用于将 PDF 文件从 A3 纸张大小裁切为 A4 纸张大小。它具有以下特点：

*   **免费使用**：无需任何费用；
*   **无需后台服务**：可以直接在浏览器中运行，无需依赖服务器；
*   **高性能**：利用 WebAssembly (WASM) 技术，提供高效的文件裁切性能；
*   **轻量级桌面应用**：通过 Tauri 打包成桌面软件，安装包体积小，方便部署；
*   **开源项目**：欢迎社区贡献代码和建议，共同改进工具。

### 项目代码地址

*   `Github`：[MuPDF-Crop-Kit](https://github.com/HyaCiovo/MuPdf-Crop-Kit)

### 开发过程与踩坑

*   `MuPDF.js`只支持`ESM`，官网中给出的要么使用`.mjs`文件，要么需要项目的`type`改成`module`：

    ```shell
    npm pkg set type=module
    ```

    我在我的`Rsbuild`搭建的项目中都没有配置成功🤷‍♂️，最后发现用`Vite`搭建的项目直接就可以用...

*   因为没有直接提供我想要的功能，肯定是要基于现有的`API`手搓了。但是截取页面的`API`会修改原页面，那么自然想到是要复制一份出来，一个截左边一个截右边了。但是`MuPDF.js`的`copyPage`复制出来的`pdf`页修改之后，原`pdf`页居然也会被修改。
    于是我想到了，一开始就`new`两个`PDFDocument`对象，一个截左边一个截右边，最后再合并到一起，我很快实现了两个文档的分别截取，并通过转`png`图片之后再合并，完成了裁切后的文档的浏览器预览。
    然后我考虑直接使用`jspdf`把`png`图片转`pdf`文件，结果`2MB`的原文件转换后变成了`12MB`，并且如果原文件是使用`扫描全能王`扫描出来的，生成的`pdf`文件会很糊。
    最后，终于让我在[文档](https://mupdfjs.readthedocs.io/en/latest/classes/PDFDocument.html#)中发现`merge`方法：

    ![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/b4ed0b46a79a4f9fb96453ddce29d821~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228339&x-orig-sign=ABRLrpBs1QPjhzFWRXBpvO9sVaU%3D)

    不过依赖包提供的方法很复杂：

    ```js
    merge(sourcePDF, fromPage = 0, toPage = -1, startAt = -1, rotate = 0, copyLinks = true, copyAnnotations = true) {
      if (this.pointer === 0) {
          throw new Error("document closed");
      }
      if (sourcePDF.pointer === 0) {
          throw new Error("source document closed");
      }
      if (this === sourcePDF) {
          throw new Error("Cannot merge a document with itself");
      }
      const sourcePageCount = sourcePDF.countPages();
      const targetPageCount = this.countPages();
      // Normalize page numbers
      fromPage = Math.max(0, Math.min(fromPage, sourcePageCount - 1));
      toPage = toPage < 0 ? sourcePageCount - 1 : Math.min(toPage, sourcePageCount - 1);
      startAt = startAt < 0 ? targetPageCount : Math.min(startAt, targetPageCount);
      // Ensure fromPage <= toPage
      if (fromPage > toPage) {
          [fromPage, toPage] = [toPage, fromPage];
      }
      for (let i = fromPage; i <= toPage; i++) {
          const sourcePage = sourcePDF.loadPage(i);
          const pageObj = sourcePage.getObject();
          // Create a new page in the target document
          const newPageObj = this.addPage(sourcePage.getBounds(), rotate, this.newDictionary(), "");
          // Copy page contents
          const contents = pageObj.get("Contents");
          if (contents) {
              newPageObj.put("Contents", this.graftObject(contents));
          }
          // Copy page resources
          const resources = pageObj.get("Resources");
          if (resources) {
              newPageObj.put("Resources", this.graftObject(resources));
          }
          // Insert the new page at the specified position
          this.insertPage(startAt + (i - fromPage), newPageObj);
          if (copyLinks || copyAnnotations) {
              const targetPage = this.loadPage(startAt + (i - fromPage));
              if (copyLinks) {
                  this.copyPageLinks(sourcePage, targetPage);
              }
              if (copyAnnotations) {
                  this.copyPageAnnotations(sourcePage, targetPage);
              }
          }
      }
    }
    ```

    而且在循环调用这个`MuPDF.js`提供的`merge`方法时，`wasm`运行的内存被爆了🤣。
    仔细阅读代码发现其核心实现就是：

    *   `addPage`新增页面；
    *   `put("Resources")`复制原文档页面中的内容到新页面；
    *   `insertPage`将新增的页面插入到指定文档中。

    因为我并没有后续添加的`link`和`annotation`，所以经过设计后，决定使用一个空的`pdf`文档，逐页复制原文档两次到空白文档中。主要逻辑如下：

    *   加载 PDF 文件：读取并解析原始 A3 PDF 文件。
    *   复制页面：创建两个新的 PDF 文档，分别截取每页的左半部分和右半部分。
    *   合并页面：将两个新文档中的页面合并到一个新的 PDF 文档中。
    *   设置裁剪框：根据 A4 纸张尺寸设置裁剪框（CropBox）和修整框（TrimBox）。

    ```ts
    export function merge(
      targetPDF: mupdfjs.PDFDocument,
      sourcePage: mupdfjs.PDFPage
    ) {
      const pageObj = sourcePage.getObject();
      const [x, y, width, height] = sourcePage.getBounds();
      // Create a new page in the target document
      const newPageObj = targetPDF.addPage(
        [x, y, width, height],
        0,
        targetPDF.newDictionary(),
        ""
      );
      // Copy page contents
      const contents = pageObj.get("Contents");
      if (contents) newPageObj.put("Contents", targetPDF.graftObject(contents));
      // Copy page resources
      const resources = pageObj.get("Resources");
      if (resources) newPageObj.put("Resources", targetPDF.graftObject(resources));
      // Insert the new page at the specified position
      targetPDF.insertPage(-1, newPageObj);
    }
    ​
    export function generateNewDoc(PDF: mupdfjs.PDFDocument) {
      const count = PDF.countPages();
      const mergedPDF = new mupdfjs.PDFDocument();
      for (let i = 0; i < count; i++) {
        const page = PDF.loadPage(i);
        merge(mergedPDF, page);
        merge(mergedPDF, page);
      }
    ​
      for (let i = 0; i < count * 2; i++) {
        const page = mergedPDF.loadPage(i); // 使用 mergedPDF 的页码
        const [x, y, width, height] = page.getBounds();
        if (i % 2 === 0)
          page.setPageBox("CropBox", [x, y, x + width / 2, y + height]);
        else page.setPageBox("CropBox", [x + width / 2, y, x + width, y + height]);
    ​
        page.setPageBox("TrimBox", [0, 0, 595.28, 841.89]);
      }
      return mergedPDF;
    }
    ```

    完成以上核心方法后，便可以成功将我侄子的试卷裁切为`A4`大小进行打印了✅。

### 体验与安装使用

#### 浏览器版

*   直接访问网页链接（[MuPdf-Crop-Kit](https://mupdf-crop-kit.vercel.app/)）。

    ![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/aa507a8bb33549c38b321b7140f2b98e~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228339&x-orig-sign=A9s2U2t1TFAM9I%2B2wEqK8J5hR%2F0%3D)

#### 桌面版

*   下载并安装 `Tauri` 打包的桌面应用（[Release tauri-x64-exe-0.1.0 · HyaCiovo/MuPdf-Crop-Kit](https://github.com/HyaCiovo/MuPdf-Crop-Kit/releases/tag/tauri-x64-0.1.0)）；
*   使用源码自行打包需要的安装包。

### 使用教程

使用本工具非常简单，只需几个步骤即可完成 PDF 文件的裁切：

1.  选择需要裁切的 A3 PDF 文件；

    ![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/5a92fb61227e4a109f1944aa606108c3~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228339&x-orig-sign=EpmhcpZjbKrm8bR7GQ%2FAjXq%2FFLs%3D)
2.  点击裁切按钮；

    ![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/b36080c194744a0e909c20a20b345ddd~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228339&x-orig-sign=%2BSUdKMWiwqnMktXxJvX98kxyxiA%3D)
3.  下载裁切后的 A4 PDF 文件。

    ![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/711b9207894b4a25ae71f2bfa05b3a01~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg6auY6aG55LiN6L-H5LiN5pS55ZCN:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNDMzMjU0NTk3MDgyMDg2MSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1745228339&x-orig-sign=8ZwDLSUo8MwYkohD%2BLfK5RA6Hh4%3D)

### 不足

*   项目所使用的`wasm`文件大小有`10MB`，本工具真正用到的并没有那么多，但是优化需要修改原始文件并重新编译；
*   浏览器端的性能受限，并且`wasm`运行可以使用的内存也是有限的；
*   没有使用`Web 工作记录er`，理论上转换这种高延迟的任务应当放在`Woker`线程中进行来防止堵塞主线程。

### 替代方案

如果在使用过程中遇到问题或需要更多功能，可以尝试以下在线工具：

*   [Split PDF Down the Middle A3 to A4 Online](https://www.sejda.com/split-pdf-down-the-middle)：每小时可以免费转换3次，作者亲测好用👍。
