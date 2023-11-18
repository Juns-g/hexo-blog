---
title: HTML面试知识点
tags:
  - HTML
  - 面试
  - 前端
categories: 面试
keywords: HTML 面试 八股
cover: "https://dogefs.s3.ladydaily.com/~/source/unsplash/photo-1676594038305-c0e855ca2d96?ixid=M3w0MjI2NjN8MHwxfHRvcGljfHxxUFlzRHp2Sk9ZY3x8fHx8Mnx8MTY4NDI5MjM2MHw&ixlib=rb-4.0.3&w=2560&h=1440&fmt=webp"
abbrlink: 6635
date: 2023-05-09 16:47:23
---

# HTML

## HTML5 新增的内容

- 媒体标签
  - audio，video，source
- 语义化标签
  - header，nav，section，footer，aside，article
- drag API，canvas，svg，websocket
- localStorage，sessionStorage
- querySelector，querySelectorAll
- 进度条 progress
- 表单
  - 表单类型：url，email，number，search，range，color 等
  - 表单属性：placeholder，required。。。
  - 表单事件：oninput 输入框内容变化就触发，oninvalid 验证通过触发

## src 和 href 的区别

- src 会阻塞，href 不会，就像一般最后才引入 js
- src 用于替换当前元素，例如 js，图片，会暂停其他资源的下载处理
- 所以 script 标签中就有 defer 和 async 来解决，async 是加载完就执行，defer 最后才执行![](https://cdn.nlark.com/yuque/0/2020/png/1500604/1603547262709-5029c4e4-42f5-4fd4-bcbb-c0e0e3a40f5a.png)

## drag API

- dragstart，被拖放元素，开始拖放时触发
- drag，被拖放元素，正在拖放时触发
- dragenter，目标元素，进入某元素时触发
- dragover，目标元素，某元素内移动
- dragleave，目标元素，移出目标元素
- drop，目标元素，完全接受拖放元素
- dragend，被拖放元素，结束触发
