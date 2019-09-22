---
title : css
---

### text-transform

``` css
div {
  text-transform: 
}
```

### @font-face

``` css
/* 加载字体文件 */
@font-face {
  /* 位置 */
  src: url("xxx.ttf");
  /* 名称 */
  font-family: "名称"
}
```

### font-weight

``` css
/* 用于设置文字的粗细 */
div {
  font-weight: normal;
  font-weight: bold;
  font-weight: 100;
  font-weight: 200;
}
```

### font-style

``` css
div {
  font-style: normal;
  /* 字体的倾斜功能 */
  font-style: italic;
  /*  文本倾斜展示 */
  font-style: oblique;
}
```

### font-variant

``` css
div {
	font-variant: normal;
  font-variant: small-
}
```

## CSS选择器

开发中需要找到特定的网页元素进行设置样式。按照一定规则选出符合条件的元素，为之添加css样式

- 通用选择器（universal selector）
- 元素选择器（type selectors）
- 类选择器（class selectors）
- id选择器（id selectors）
- 属性选择器（attribute selectors）
- 组合（combinators）
- 伪类（pseudo-classes）
- 伪元素（pseudo-elements）