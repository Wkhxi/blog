---
title: Data URL
tags:
  - 前端
  - Data URL
categories:
  - 前端
  - Data URL
abbrlink: ec77340d
date: 2022-07-14 20:57:41
---


##  Data URL 是什么

```
以 data：模式为前缀的url，将一些比较小的数据直接嵌入网页中，不用再从外部载入。 例如：data:image/jpeg;base64,/9j/4......
```

## 优点和缺点分析


```html
<!-- src里写入 base64编码 -->
<img src="data:image/jpeg;base64,/9... "  alt="">

```

```css
// url里写入 base64编码 
.box1 {
  width: 100px;
  height: 100px;
  background-image: url(base64编码);
}
```

```
此时，当你加载页面的时候会发现，在network中的图片请求的 请求网址 就是 base64编码，而不是http开头的。

状态代码: 200 OK （来自内存缓存）。意思是，不请求网络资源，资源在内存中。

```

1. 会减少一个图片的http请求数（类似的有精灵图，通过减少http请求提高性能）。

2. 因为一般转成base64之后大小会增大，再将base64写在html或css中，会使html或css代码膨胀体积增大影响渲染。
所以一般会选择将一些比较小的图片资源以base64这种方式嵌入网页。

3. 还有一个缺点：无法缓存。

所以使用这种方式并不代表性能的优化，要具体情况具体分析。
