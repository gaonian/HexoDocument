---
title : html
---

### html

### head

- meta

  ``` html
  <meta charset="UTF-8">
  ```

- title

  ``` html
  <title>这是title</title>
  ```

- style

  ```html
  <style>
    /* 此处写css代码 */
    dic {
      font-size: 10px
    }
  </style>
  ```

  

### body

### div

### span

### a

### img

### p

### input

### form

### button

### label

### strong



## 列表

### 有序列表 - ol、 li

- ol（ordered list）

  有序列表，直接子元素只能是li

- li (list item)

  列表中的每一项

```html
<ol>
  <li>第一个</li>
  <li>第二个</li>
  <li>第三个</li>
  <li>第四个</li>
</ol>
```

### 无序列表 - ul、li

- ul（unordered list）

  无序列表，直接子元素只能是li

- li (list item)

  列表中的每一项

```html
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
  <li>4</li>
</ul>

<style>
  ul {
    /* 去除ul前面的点 */
    list-style-type: none
  }
</style>
```

### 定义列表 - dl、dt、dd

- dl (definition list)

  定义列表，直接子元素只能是dt、dd

- dt (definition term)

  列表中每一项的项目名

- dd (definition description)

  列表中每一项的具体描述，是对 dt的描述、解释、补充

  一个dt后面一般紧跟着1个或者多个dd

- dt、dd常见的组合

  - 事物的名称、事物的描述
  - 问题、答案
  - 类别名、归属于这类的各种事物

```html
<dl>
  <dt>支付方式</dt>
  <dd>微信支付</dd>
  <dd>支付宝支付</dd>
  <dd>现金支付</dd>
</dl>
```

### 列表相关的css属性

列表相关的常见CSS属性有4个：`list-style-type`、`list-style-image`、`list-style-position`、`list-style`

- `list-style-type`：设置li元素前面标记的样式

  - disc（实心圆）、circle（空心圆）、square（实心方块）

  - decimal（阿拉伯数字）、lower-roman（小写罗马数字）、upper-roman（大写罗马数字）

  - lower-alpha（小写英文字母）、upper-alpha（大写英文字母）

  - none（什么也没有）

- `list-style-image`：设置某张图片为li元素前面的标记，会覆盖list-style-type的设置

- `list-style-position`：设置li元素前面标记的位置，可以取outside、inside2个值
- `list-style`：是list-style-type、list-style-image、list-style-position的缩写属性

一般最常用的还是设置为none，去掉li元素前面的默认标记 **list-style**:**none**;



## 表格

- `table`

  表格

- `tr`

  表格中的行

- `td`

  行中的单元格

- `tbody`

  表格的主体

- `caption`

  表格的标题

- `thead`

  表格的表头

- `tfoot`

  表格的页脚

- `th`

  表格的表头单元格

```html
<table>
  <tr>
    <td>姓名</td>
    <td>年龄</td>
    <td>性别</td>
  </tr>
  <tr>
    <td>张三</td>
    <td>18</td>
    <td>男</td>
  </tr>
  <tr>
    <td>李四</td>
    <td>20</td>
    <td>男</td>
  </tr>
</table>
```

**table的常用属性**

| border      | 边框的宽度                            |
| ----------- | ------------------------------------- |
| cellpadding | 单元格内部的间距                      |
| cellspacing | 单元格之间的间距                      |
| width       | 表格的宽度                            |
| align       | 表格的水平对齐方式left、center、right |

**tr的常用属性**

| valign | 单元格的垂直对齐方式top、middle、bottom、baseline |
| ------ | ------------------------------------------------- |
| align  | 单元格的水平对齐方式left、center、right           |

**th td的常用属性**

| valign  | 单元格的垂直对齐方式top、middle、bottom、baseline |
| ------- | ------------------------------------------------- |
| align   | 单元格的水平对齐方式left、center、right           |
| width   | 单元格的宽度                                      |
| height  | 单元格的高度                                      |
| rowspan | 单元格可横跨的行数                                |
| colspan | 单元格可横跨的列数                                |

### 细线表格

```html
<style>
  
  table {
    /* 合并单元格的边框 */
    border-collapse: collapse;
  }
  
  td {
    border: 1px solid #000;
  }
  
</style>
```

###单元格合并

合并方向是向右、向下，删掉被覆盖掉的td元素

``` html
<table>
  <tr>
    <td rowspan="2" colspan="2">姓名</td>
    <td>年龄</td>
    <td>性别</td>
  </tr>
  <tr>
    <td>张三</td>
    <td>18</td>
    <td>男</td>
  </tr>
  <tr>
    <td>李四</td>
    <td>20</td>
    <td>男</td>
  </tr>
</table>
```



## 表单

- `form`

  表单，一般情况下，其他表单相关元素都是它的后代元素

- `input`

  单行文本输入框、单选框、复选框、按钮等元素

- `textarea`

  多行文本框

- `select、option`

  下拉选择框

- `button`

  按钮

- `label`

  表单元素的标题

- `fieldset`

  表单元素组

- `legend`

  fieldset的标题

### form的常用属性

- **action**

  用于提交表单数据的请求URL

- **method**

  请求方法（get和post），默认是get

- **target**

  在什么地方打开URL _blank _self 参考a元素的target

- **enctype**

  规定了在向服务器发送表单数据之前如何对数据进行编码

  取值有3种

  - `application/x-www-form-urlencoded`：默认的编码方式
  - `multipart/form-data`：文件上传时必须为这个值，并且method必须是post
  - `text/plain`：普通文本传输

- **accept-charset**

  规定表单提交时使用的字符编码

### input常用属性

- **type**:  input类型

  - text : 文本输入框（明文输入）
  - password ：文本输入框（密文输入）
  - radio：单选框
  - checkbox：复选框
  - button：按钮
  - reset：重置
  - submit：提交表单数据给服务器
  - file：文件上传
  - hidden：隐藏域

- **maxlength**：允许输入的最大字数

- **placeholder**：占位文字

- **readonly**：只读

- **disabled**：禁用

- **checked**：默认被选中，只有当type为radio或checkbox时可用

- **autofocus**：当页面加载时，自动聚焦

- **name**：名字。在提交数据给服务器时，可用于区分数据类型

- **value**：取值

- **form**：设置所属的form元素（填写form元素的id）

  一旦使用了此属性，input元素即使不写在form元素内部，它的数据也能够提交给服务器

  