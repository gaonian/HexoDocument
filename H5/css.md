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

  > *

- 元素选择器（type selectors）

  > 标签选择器 div input span

- 类选择器（class selectors）

  > .name

- id选择器（id selectors）#title

  

- 属性选择器（attribute selectors）

  > [href="#top"]

- 组合（combinators）

- 伪类（pseudo-classes）

  - 动态伪类
    - a:link 未访问的链接
    - a:visited 已访问的链接
    - a:hover 当鼠标移动到链接上
    - a:active 当鼠标左键点击选中时
    - a:focus 当链接被tab键选中激活时
    - :hover :active :focus还可以应用到其他元素上

- 伪元素（pseudo-elements）



## 继承

一个元素如果没有设置某属性的值，就会跟随父元素的值。当然，一个元素如果有设置某属性的值，就使用自己设置的值。比如color、font-size等属性都是可以继承的。如果不能继承的属性还想要使用继承，则使用`inherit`强制继承

```css
div {
  border: 2px
}

p {
  /* inherit 强制继承 */
  border: inherit;
}
```



