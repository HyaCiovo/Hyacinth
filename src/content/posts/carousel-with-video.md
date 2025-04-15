---
title: AntD Carousel 组件支持更加符合直觉的视频轮播图方案
published: 2025-03-21
updated: 2025-04-10
description: '想想当初我还嘲笑那位同事被安排了这么个活，现在回旋镖打到自己身上了🫤...... Carousel 轮播图组件支持视频、图片混用，支持自动轮播、移入暂停，支持视频播放完毕后再切换页面 ......'
tags: [React]
category: '工作记录'
draft: false 
---

## 更新日志
> 2025/4/10 
> 
> 发现了`bug`，当轮播图最后一项为视频时，视频不会自动播放，但是依然触发`onEnded`事件。检查后发现是因为`Carousel`组件复制了`dom`，`dom`列表中会把最后一项`Slider`复制到最前面，而使用的`docuemnt.getElementById()`方法默认获取的是第一个，但是展示在页面上的是第二个，所以操作错了`dom`。后续更新了获取`dom`的方式解决了这个问题。

## 背景

产品经理前段时间给官网首页轮播图加上了视频播放的`feature`，不过因为还要支持自动轮播，所以每次视频都放不完就切走了。因为之前急着上线，当时的解决方案是首页关掉了自动轮播，只能用户手动切换。最近新的迭代需求，产品又把这个提了出来，需要优化：

> 1. 视频文件播放完，切换至下一个`Slider`展示；
> 2. 静态图片固定展示`3s`后切至下一个`Slider`；
> 3. 依然支持鼠标移入后的暂停轮播功能。

而且还急着上线，排在了`T0`优先级。结果当初做支持视频播放功能的前端同学这两周请假了，这活落在了我的头上😅。想想当初我还嘲笑那位同学被安排了这么个活，现在回旋镖打到自己身上了🫤。

## 调研

首先我想着能不能找到现成的完美支持我们需求的轮播图组件，不过我简单搜索了一下，并没有找到合适的开源组件；考虑了一下，要么装个`Swiper.js`手搓，要么在现有正在使用的`Ant Design`组件库的`Carousel`组件上手搓了。既然都手搓，还是用现成的吧🤣。

## 分析

对需求进行了一下分析 ：

1. **视频文件播放完，切换至下一个 Slider 展示**。这个实现起来不难，主要有以下几种思路：

   - 后端返回视频资源路径的同时，返回该视频资源的时长，前端手动给对应的`Slider`实现相应的播放时长，需要后端接口改动；
   - 前端拿到视频资源，手动加载获取视频的时长，接着和之前一样实现对应的播放时长，需要加载整个视频才能拿到；
   - 给视频加上`onEnded`事件监听，准确获取视频播放结束时刻，进行后续操作。

   > 隐性功能：切换到下一页之前，这一页的视频要停止播放；下次展示该视频时，需要重新播放。

2. **静态图片固定展示 3s 后切至下一个 Slider**。这个比较简单；

3. **依然支持鼠标移入后的暂停轮播功能**。看起来比较简单，`Carousel`组件本身就支持`pauseOnfocused`。

   > 隐性功能：视频页在`pauseOnfocused`时，需要轮播，而不是播放完一次就暂停不播了。

## 开发

1. **视频文件播放完，切换至下一个 Slider 展示**。因为优先考虑前端单独实现且重视性能，所以直接采用`video`标签的`onEnded`事件属性方案。

   - 首先确定手动播放视频的方法：

     ```ts
     // 视频播放方法
     const playVideo = (ele: HTMLVideoElement) => {
       if (ele?.paused)
         try {
           ele?.play()
         } catch (_err) {
           ele?.play()
         }
     }
     ```

   - 在视频播放时，将`Carousel`组件的`autoplay`属性改为`false`，播放结束后再改为`true`，且暂停视频播放：

     ```ts
     // 视频播放
     const playVideo = (ele: HTMLVideoElement) => {
       if (ele.paused){
         setAutoplay(true)
         try {
           ele.play()
         } catch (_err) {
           ele.play()
         }
       }
     }
     // 视频结束
     const handleEnded = (e: any) => {
       e.target.pause();
       setAutoplay(true);
     }
     ```

   - 发现修改`Carousel`组件的`autoplay`属性，会导致整个组件重新渲染，内部的视频标签也会重新渲染导致闪烁。所以给内部的媒体组件加上`React.memo()` 。

     ```tsx
     const MedaNode = memo(({
       video,
       imageUrl,
       index,
       handleEnded
     }: MedaProps) => {
       // 视频
       if (video) {
         return <video
           src={imageUrl} autoPlay muted preload="auto" disablePictureInPicture
           id={`video-${index}`}
           onEnded={handleEnded}
         />
       }
       // 静图
       return <img src={imgUrl} />
     })
     
     export default Banner ({ bannerList }: BannerProps) {
       // ...
       const handleEnded = useCallback((e: any) => {
         e.target.pause();
         setAutoplay(true)
       }, [])
       // ...
       return (
         <Carousel>
            {list?.length &&
              list.map((item, index) => {
                const { id, imageUrl, linkUrl, video } = item;
                return (
                  <div
                     key={id}
                     onClick={() => linkUrl && Router.push(linkUrl)}
                   >
                     <MedaNode 
                        video={video} 
                        imageUrl={imageUrl} 
                        index={index} 
                        handleEnded={handleEnded} 
                      />
                   </div>)
               })}
         </Carousel>)
     }
     ```

2. **静态图片固定展示 3s 后切至下一个 Slider**。这个功能，只要开启了`Carousel`组件的`autoplay`就可以直接实现；

3. **依然支持鼠标移入后的暂停轮播功能**。分析一下并不难，因为`Slider`是图片时，该功能是自动支持，且`Slider`是视频时，自动轮播是关闭的，播放完需要前端手动切换到下一页。

   - 只要对视频播放结束方法进行改造，添加手动切换：

     ```ts
     const handleEnded = useCallback((e: any) => {
         e.target.pause();
         carouselRef.current?.next();
         setAutoplay(true)
       }), [])
     ```

   - 此外要注意功能分析时发现的`隐性功能`：视频页在`pauseOnfocused`时，需要轮播，而不是播放完一次就暂停不播了。一开始认为只要`handleEnded`方法在`mouseEnter`为`true`时，不做任何操作：

     ```ts
     const handleEnded = useCallback((e: any) => {
       if(mouseEnter)
         return;
       e.target.pause();
       carouselRef.current?.next();
       setAutoplay(true);
     }), [mouseEnter])
     ```

     就可以实现，因为`video`标签加上了`autoPlay、 loop`属性，不去`pause`，自然就会一直重复播放下去。结果发现，加上了`loop`属性的`video`标签无法触发`onEnded`事件，加上了`autoPlay`属性后去`pause`时，视频总是无法回到播放时间为`0`的时刻，导致重新`play`时，会闪烁。经过重复尝试最终修改：

     ```ts
     // 视频播放
     const playVideo = (ele: HTMLVideoElement) => {
       if (ele?.paused){
         ele.currentTime = 0;
         try {
           ele?.play()
         } catch (_err) {
           ele?.play()
         }
       }
     }
     
     // 视频结束
     const handleEnded = useCallback((e: any) => {
       if (mouseEnter)
         playVideo(e.target)
       else {
         e.target.pause();
         carouselRef.current?.next();
         setAutoplay(true)
       }
     }, [mouseEnter])
     ```

   ## 最终代码

   ```tsx
   import { Carousel } from "antd";
   import { CarouselRef } from "antd/es/carousel";
   import { memo, useCallback, useEffect, useMemo, useRef, useState } from "react";
   import { BannerProps, MedaProps } from "src/interfaces/type";
   import { getFileExt } from "src/utils";
   
   // 视频播放方法
   const playVideo = (ele: HTMLVideoElement) => {
     if (ele?.paused)
       try {
         ele?.play();
       } catch (_err) {
         ele?.play();
       }
   }
   
   // 视频暂停方法
   const pauseVideo = (ele: HTMLVideoElement) => {
     ele.currentTime = 0;
     if (!ele?.paused){
       ele.pause();
     }
   }
   
   const MedaNode = memo(({
     video,
     imageUrl,
     index,
     handleEnded
   }: MedaProps) => {
     // 视频
     if (video) {
       return <video
         src={imageUrl} autoPlay muted preload="auto" disablePictureInPicture
         className={`video-${index}`}
         onEnded={handleEnded}
       />
     }
     // 静图
     return <img src={imgUrl} />
   })
   
   export default Banner ({ bannerList }: BannerProps) {
     const [autoplay, setAutoplay] = useState<boolean>(false);
     const [current, setCurrent] = useState<number>(0);
     const carouselRef = useRef<any>(null);
       
     const list = useMemo(() => {
       return bannerList.map((item: any) => {
         const ext = getFileExt(item.imageUrl); // 截取文件后缀
         return { ...item, video: ext === 'mp4' }; // 返回新对象
       });
     }, [bannerList]);
     
     /* 因为 Carousel 组件复制了 dom，所以原本使用 id 来获取 dom 的方法有问题 */
     const getVideoElement = (index: number): HTMLVideoElement => {
       const i = index !== list.length - 1 ? 0 : 1; // 最后一个 item 需要取第2个，因为最前面有一个复制项
       return document.getElementsByClassName(`video-${index}`)[i] as HTMLVideoElement;
     }
      
     const beforeChange = (cur: number) => {
       if (cur === 0)
         setAnimate(false);
       const currentItem = list[cur];
       if (!currentItem) return;
       if (currentItem?.video) {
         const currentVideoElem = getVideoElement(cur)
         pauseVideo(currentVideoElem)
       }
     }
   
     useEffect(() => {
       const currentItem = list[0]
       if (!currentItem) return;
       if (currentItem?.video) {
         setAutoplay(false)
         const videoElem = getVideoElement(0);
         playVideo(videoElem)
       }
     },[])

     const afterChange = (cur: number) => {
       const currentItem = list[cur];
       if (currentItem?.video) {
         setAutoplay(false)
         const videoElem = getVideoElement(cur);
         playVideo(videoElem)
       } else {
         setAutoplay(true)
       }
     }
       
     const handleEnded = useCallback((e: any) => {
       if (mouseEnter)
         playVideo(e.target);
       else {
         carouselRef.current?.next();
         pauseVideo(e.target);
       }
     }, [mouseEnter])
     
     return (
       <Carousel 
          afterChange={afterChange} 
          beforeChange={beforeChange}
          autoplay={autoplay}
          ref={carouselRef} 
          speed={800}
        >{list?.length &&
            list.map((item, index) => {
              const { id, imageUrl, linkUrl, video } = item;
              return (
                <div
                   key={id}
                   onClick={() => linkUrl && Router.push(linkUrl)}
                 >
                   <MedaNode 
                      video={video} 
                      imageUrl={imageUrl} 
                      index={index} 
                      handleEnded={handleEnded} 
                    />
                 </div>)
             })
         }
       </Carousel>)
   }
   ```


### 结果

功能完美实现，通过产品审核，“暂时”还没出现问题🫡。
