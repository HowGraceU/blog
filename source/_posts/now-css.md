---
title: 现代css
date: 2019-05-19
tags:
    - css
---

**近年来，js 的发展速度可谓是突飞猛进，一年一个版本，而 css 的进步也不容小觑。css 在最近也新增了不少 api。**

## css变量

**原生 css 支持使用变量，变量的内容可以是任何东西，变量也遵循 css 优先级的规则覆盖。**

``` css
:root {
    --fontSize: 1rem;
    --mainColor: #12345678;
    --highlightColor: red;
}
```

<p data-height="500" data-theme-id="dark" data-slug-hash="wbqgXR" data-default-tab="css,result" data-user="howgraceu" data-embed-version="2" data-pen-title="CSS sidebar toggle" class="codepen"></p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## :any-link 伪类

**代表伪类 :visited 和 :link 的集合**

<p data-height="300" data-theme-id="dark" data-slug-hash="XweXLV" data-default-tab="css,result" data-user="howgraceu" data-embed-version="2" data-pen-title="CSS sidebar toggle" class="codepen"></p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## :blank 伪类

**表单元素的 value 值为空时，伪类生效。因为 codepen 找不到对应的 postcss 插件，这里手动写了一下兼容。**

<p data-height="350" data-theme-id="dark" data-slug-hash="VOMayZ" data-default-tab="css,result" data-user="howgraceu" data-embed-version="2" data-pen-title="CSS sidebar toggle" class="codepen"></p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>