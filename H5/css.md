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

### outline

### border

### border-radius

### border-spacing

```css
table {
  /* 合并单元格边框 */
  border-collapse: collapse;
  
  /* 设置单元格水平 垂直边距 */
  border-spacing: 10px, 20px
}
```

### display

```css
span {
  /* 让元素显示为块级元素 */
  display: block;
}

div {
  /* 让元素展示为行内元素 */
  display: inline;
}

p {
  /* 隐藏 */
  display: none;
}

span {
  /* 行内。随意设置宽高, 宽高包裹内容 */
  /* 跟其他行内级元素展示在同一行，而且能随意更换宽高
  让行内级非替换元素随意更换宽高（a、span）
  让块级元素能够和其他元素一行显示（div、p）*/
  display: inline-block;
}
```

### visibility

```css
div {
  /* 可见 隐藏 
  	 和display：none的区别是 是否改变位置*/
  visibility: visible;
  visibility: hidden;	
}
```

### overflow

``` css
div {
  /* 默认 一直显示 */
  overflow: visible;
  /* 超出隐藏 */
  overflow: hidden;
  /* 一直添加一个滚动条 */
  overflow: scroll;
  /* 超出范围添加滚动条 */
  overflow: auto;
  overflow-x: auto;
  overflow-y: auto;
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

##  优先级

按照经验，为了方便比较CSS属性的优先级，可以给CSS属性所处的环境定义一个权值（权重）

- !important: 10000
- 内联样式：1000
- id选择器：100
- 类选择器、属性选择器、伪类：10
- 元素选择器、伪元素：1
- 通配符*： 0

**比较优先级的严谨方法**

从权值最大的开始比较每一种权值的数量多少，数量多的则优先级高，即可结束比较

如果数量相同，比较下一个较小的权值，以此类推

如果所有权值比较完毕后，发现数量都相同，就采取“就近原则”

> 总结：选择器的针对性越强，优先级越高

