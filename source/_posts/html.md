---
title: HTML 
date: 2017-05-21 21:15:27
categories:
- HTML
tags: 
- html
---
到了这个阶段再来看html有一种说不出来的感觉，其实从未全面学习过，简单过一遍留个脚印。
<!--more-->

## 简介
- html是一个文本描述语言，用来描述网页文档(web pages/documents)，几乎是专用的。
- 该描述语言由一系列描述标签(tags)组成
- 不同的标签描述不同的描述内容
- 描述标签的集合组成了HTML文档

示例:
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page Title</title>
  </head>

  <body>
    <h1>My First Heading</h1>
    <p>My first paragraph.</p>
  </body>
</html>
```
样例说明：
- html对大小写不care，有过care的版本，但都是过去式，不必关心，但习惯上小写
- `DOCTYPE`声明定义文档类型为html，写html表示HTML5，是2014年提出的最新标准
- `<html></html>`标签内部的内容表示该文档的全部内容
- `<head></head>`表示头部,类似人体结构
- `<body></body>`表示主体部分
- html是写给机器读的，tags标签不会显示，仅用作标识，用来定义其内容的展示方式
- 仅有body部分的内容会展现在浏览器中
- html5标准中，`<html>`、`<body>`、`<head>`标签都是可以省略的，浏览器也能解析，但是不推荐这么做
## html元素
标签对以及其中的内容构成html元素。
- 元素内部可以包含元素，即元素是可嵌套的
- `<html>`元素定义了整个文档，其他元素都是它的子元素
- 不要忘记结束标签

每个元素都有一个默认的展示值从而决定它展示的方式，这个值通常是`block`或者是`inline`
### 块级元素（block）
一个块级元素通常起始于行首，并占据整行，后续元素不会与之并排。
常见的这类元素有：
- `<div>`
- `<h1>-<h6>`
- `<p>`
- `<form>`
### 行内元素 （inline）
不需要从起始于行首，并且只占据它需要的宽度。
常见的这类元素有：
- `<span>`
- `<img>`
- `<a>`

### 行内块元素 （inline-block）
强制改变块级元素的展现方式，让它们能同行并排，常见做法是改变该元素的CSS属性`display`为
`inline-block`
## html属性
html属性具体的说是元素的属性，用来描述元素额外的信息。
- 属性的作用是提供元素的额外信息
- 属性通常在起始tag中的定义
- 属性的定义规则是 `name="value"`形式, 多个属性声明时，以`空格`间隔, 一般使用双引号，内部存在双引号时也可使用单引号
eg:
```html
<!DOCTYPE html>
<html lang="en-US">
</html>
```
在html元素中声明`lang`属性，用来表明文档的语言类型。
常见的还有，
- 在`<p>`元素中增加`title`属性，当鼠标移动到该元素上时，title的内容会在提示框中显示。
- `<a>`元素的`href`属性用于超链接
- `<img>`元素`<img src="w3schools.jpg" alt="W3Schools.com" width="104" height="142">`,其中width和height单位为屏幕像素
### style
设置元素的显示风格.
定义规则为`style="property:value;"` property是一个css属性名，value是一个css属性值。
元素的展示形式属于视图层的范围，一般建议单独以css的形式定义，style属性提供内联的定义形式，这里不详细展开。只需区分style元素属性值的定义规则和元素属性定义规则的区别即可:
- name="value" html元素属性定义以空格分隔
- style="property:value;" style属性定义以`分号`分隔

## html标签
html标签太多，但常用的虽然不多
- `<hr>`水平分隔线，常与标题标签一起使用
- `<p>` 无论多少空格都会被算为一个，通常使用`&nbsp;`表示多个空格。换行同样不起作用，需独立使用`<br>`标签换行。与之相对的`<pre></pre>`标签的空格和换行都会保留，但
会使用固定宽度的字体
- ...

## html文本格式化
- Bold text - `<b></b>`, `<strong></strong>`
- Italic text - `<i></i>`, `<em></em>`(Emphasized )
- Marked text - `<mark></mark>`
- Small text - `<small></small>`
- Deleted text - `<del></del>`
- Inserted text - `<ins></ins>`
- Subscripts - `<sub></sub>`
- Superscripts - `<sup></sup>`
- Quotations - `<q></q>`, `<blockquote></blockquote>`, `<cite></cite>`
- Abbreviation - `<abbr title="World Health Organization">WHO</abbr>`
- Bi-Directional Override - 方向逆转， `<bdo dir="rtl">This text will be written from right to left</bdo>`
- comments - 注释 `<!-- comment lines -->`

## html布局
1. 通常使用`<div>`元素作为布局的工具，因为它能很方便的被css定位，从而定义它的样式。
2. html5提供了新的一套语义更清晰的元素来定义网页的不同部分。
- `<header>` 整个网页或某个局部区域的头部
- `<nav>` 导航链接的容器
- `<section>` 定义网页的各个不同区域
- `<article>` 定义一个独立的文章区域
- `<footer>` 整个网页或某个局部区域的底部
3. 还有一些很老的网页使用`table `元素来布局，不推荐。

## html响应式设计
1. 最简单的方式就是将整个页面使用块级元素分隔成固定大小的模块，然后使用`float`属性，让其随屏幕变化产生自适应效果。
2. 另一种方式就是使用响应式的样式表，例如`w3.css`等响应式css类库。


## html实体符号
由于很多字符被html使用或其他原因被设定为保留字，因此想表达原有的意思只能选择替代方案。
一般有两种表达方式：
1. &entity_name
2. &#entity_number
例如：
- 小于号的写法就为 `&lt`或`&#60`
- 常见的还有空格的表示，`&nbsp`
- copyright符号，`&copy`或`&#169`
- 人民币符号，`&ren`, `$#165`


