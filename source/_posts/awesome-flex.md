---
title: flex 小技巧
date: 2019-09-17
tags:
    - css
    - flex
---

<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

一、flex 实现左中右布局

**中间布局格子的 dom 需要在最上面，宽度填满剩下的空间。左和右边框需要固定宽度（不会扩大也不会缩小）**

**只要保证左右框的 flex-grow，flex-shrink 属性为 0， 中间框的 flex-grow，flex-shrink 属性为 1 即可。（下面则使用 flex 简写属性）**

<p data-height="500" data-theme-id="dark" data-slug-hash="VwZEWWW" data-default-tab="css,result" data-user="howgraceu" data-embed-version="2" data-pen-title="CSS sidebar toggle" class="codepen"></p>