---
title: 在网页中使用Material-Design图标
date: 2020/03/28 16:24:14
abbrlink: @@abbrlink
categories:
- Web前端
tags:
- 前端
- 图标
- 网页
- 使用
---
## 什么是Material Design图标
`MD`大家都听过的吧，就是谷歌的Material Design设计，Material Design图标就是符合
MD设计语言的图标。
本文通过字体的方法在网页中显示Material Design图标。

## What are material icons
>Material design system icons are simple, modern, friendly, and sometimes quirky. Each icon is created using our design guidelines to depict in simple and minimal forms the universal concepts used commonly throughout a UI. Ensuring readability and clarity at both large and small sizes, these icons have been optimized for beautiful display on all common platforms and display resolutions.

See the full set of material design icons in the [Material Icons Library](https://www.google.com/design/icons/).

## 准备
### 1. 使用谷歌在线字体
这是最方便的方式：

引入在线CSS文件
```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons"
      rel="stylesheet">
```

### 2. 使用本地字体
由于某些你懂的原因，谷歌服务在大陆是无法使用的，所以我们可以把这些字体下载下来，然后在网页中引用即可。

下载地址：https://github.com/google/material-design-icons/tree/master/iconfont

新建CSS文件：
```css
@font-face {
  font-family: 'Material Icons';
  font-style: normal;
  font-weight: 400;
  src: url(https://example.com/MaterialIcons-Regular.eot); /* For IE6-8 */
  src: local('Material Icons'),
    local('MaterialIcons-Regular'),
    url(https://example.com/MaterialIcons-Regular.woff2) format('woff2'),
    url(https://example.com/MaterialIcons-Regular.woff) format('woff'),
    url(https://example.com/MaterialIcons-Regular.ttf) format('truetype');
}

.material-icons {
  font-family: 'Material Icons';
  font-weight: normal;
  font-style: normal;
  font-size: 24px;  /* Preferred icon size */
  display: inline-block;
  line-height: 1;
  text-transform: none;
  letter-spacing: normal;
  word-wrap: normal;
  white-space: nowrap;
  direction: ltr;

  /* Support for all WebKit browsers. */
  -webkit-font-smoothing: antialiased;
  /* Support for Safari and Chrome. */
  text-rendering: optimizeLegibility;

  /* Support for Firefox. */
  -moz-osx-font-smoothing: grayscale;

  /* Support for IE. */
  font-feature-settings: 'liga';
}
```


## 在网页中使用
在Material Design图标库中寻找你想要的图标，然后通过名字就可以把图标正确显示在网页里咯。

图标库：https://material.io/tools/icons/

使用举例：
```html
<i class="material-icons">face</i>
```

{% asset_img 8869373-72d8103d343ca04d.png %}

## 欢迎交流
交流问题请在微信公众号后台留言，每一条信息我都会回复哈~
- 微信公众号：画星星高手
- 打代码直播间：[https://live.bilibili.com/11883038](https://live.bilibili.com/11883038)
- 知乎：[https://www.zhihu.com/people/dealiaxy](https://www.zhihu.com/people/dealiaxy)
- 专栏：[https://zhuanlan.zhihu.com/deali](https://zhuanlan.zhihu.com/deali)
- 简书：[https://www.jianshu.com/u/965b95853b9f](https://www.jianshu.com/u/965b95853b9f)
