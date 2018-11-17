---
title: CSS
date: 2017-05-22 21:32:27
categories:
- CSS
tags: 
- css
---
总结过了html，自然不能少了css，同样我几乎没系统看过，虽然我一直知道css不简单，手写css是需要过硬基础的，本文重点记录学习过程中的基础内容，可供需要时查阅。
<!--more-->

## 作用及使用
### 作用
css作为视图层决定了网页的展现方式，html决定了内容，css则决定了内容的呈现方式。
### 使用
css有三种使用方式
- inline 通过元素的`style`属性定义
- internal 通过页面头部中的`<style>`元素定义
- external 通过页面头部中的`<link>`元素链接一个外部的样式表

## 语法规则
`selector {property: value; ...}`
eg:
```css
h1 {
    color: blue;
    font-size: 12px;
}
```
- selector - 选择器，指向要渲染的元素
- property - css属性
- value - css属性值
- 定义多个属性时用`分号`分隔

## css selectors
css的选择器是基于目标元素的元素名、id、类、元素属性等特征，找出所有符合条件的元素。

- id selector `#id {property:value;}`
- class selector `.class {property:value;}`
- element selector `elementName {property:value;}`
- 当然也能混合使用定位到具体的的元素，例如`p.center {text-align: center}`。即定位到类为center的p元素

- group selectors 当某些元素的样式相同是可以一起定义，去除重复代码，例如
```css
h1, h2, p {
    text-align: center;
    color: red;
}
```
## css 属性
css属性如同html元素，数量较多，但各司其职。
### color
颜色有三种表示方式：
1. 颜色名， 类似于`red`, `green`等，html和css支持140种标准颜色名
2. RGB， eg:`rgb(255,0,0)`
3. 十六进制表示， eg:`#FF0000`

css3则新增了更多的表示方法，常用的有
- RGBA, eg:`rgba(255, 0, 0, 0.2);`最后一个值的范围是`0.0-1.0`,代表透明度，0代表完全透明，1表示纯色

### backgrounds
```css
body {
    background: #ffffff url("img_tree.png") no-repeat fixed right top;
}
```
示例为简写的方式，`background`的属性值的属性名依次是：
- `background-color`
- `background-image`  (url('URL))
- `background-repeat` (repeat/repeat-x/repeat-y/no-repeat/initial/inherit default repeat)
- `background-attachment` (fixed/scroll, default scroll)
- `background-position`

### borders
```css
p {
    border: 5px solid red;
}
```
示例为简写的方式，`border`的属性值的属性名依次是：
- `border-width`
- `border-style` (required)
- `border-color`

### margins
在盒模型中指包围在元素border外层的空间
- 简写赋值依次是`顺时针上右下左`
- `auto` 的赋值可以让元素水平居中
```css
div {
    width:300px;
    margin: auto;
    border: 1px solid red;
}
```
此时的`auto`等同于`0 auto`,让水平方向的margin相等从而实现居中的效果

### padding
在盒模型中指元素内容和元素border之间的填充
- 赋值方式和margin类似

### height/width
设置元素的宽高，默认值是`auto`,即让浏览器自行计算占据的宽高，但也可以自定义
```css
div {
    width: 500px;
    height: 100px;
    border: 3px solid #73AD21;
}
```
还有其他一些设置方法，以`width`举例
- `max-width` 设置最大宽度，默认值是`none`,即不存在最大宽度。
- `max-width` 覆盖`width`的值
- 使用`width`时当窗口小于设置宽度时会自动添加滚动条，用`max-width`则不会产生
- `min-width` 设置元素最小宽度，覆盖`max-width`、`width`的值

#### 盒模型
低版本的IE浏览器中，盒模型的`width` = `content_width` + `padding-left` + `padding-right` + `border`*2。`height`也类似包括了padding和border

而标准的w3c盒模型`width` = `content_width`， `height` = `content_height`

在html中添加`DOCTYPE`的值，告诉浏览器以w3c的标准去解析盒模型

### text
定义文本样式。
- 网页的默认文本样式定义在`body`选择器上。例如，设置body的color为red，那页面所有文本颜色均为红色了。

#### text-align
设置文本的水平对其方式，可选值有
- center
- left
- right
- justify - 使得每一行宽度相同，如同报纸里的内容展示那样。

#### text-decoration
装饰文本。一般设置值为`none`，用来去除`<a>`链接的下划线。
一般可选值还有：
- overline
- underline
- line-through

#### text-transform
文本转换，一般用来转换大小写。
- uppercase
- lowercase
- capitalize (每个单词首字母大写)

#### text-indent
首行缩进。
eg:
```css
p {
    text-indent: 50px;
}
```

#### letter-spacing
词间隔。
eg:
```css
h1 {
    letter-spacing: 3px;
}

h2 {
    letter-spacing: -3px;
}
```

#### word-spacing
字间隔。
eg:
```css
h1 {
    word-spacing: 10px;
}

h2 {
    word-spacing: -5px;
}
```

### Fonts
定义字体的字体族，宽度, 以及字体风格。

#### font-family
字体族分两类
- Generic family：一组风格差不多的字体组成, 如`Serif`
- font family：单个拥有具体名字的字体，如`Times New Roman`

赋值时一般设置多个字体名，用作回退。一般最后一个会设置为`Generic family`类字体族,防止前面的一个均为匹配上，系统会从一组中选一个近似的选择。eg:
```css
p {
    /* 有空格时需加引号 */
    font-family: "Times New Roman", Times, Serif;
}
```

#### font-style
字体风格，一般为正常字体或斜体
- normal
- italic

#### font-size
设置字体大小
#### font-weight
设置字体的粗细，一般为正常字体或粗体
- normal
- bold

#### shorthand font property
依次顺序为
- font-style,
- font-variant ,
- font-weight ,
- font-size/line-height ,
- font-family

```css
p {
    font: italic bold 12px/30px Georgia, serif;
}
```

### Links
#### links state
- a:link - a normal, unvisited link
- a:visited - a link the user has visited
- a:hover - a link when the user mouses over it
- a:active - a link the moment it is clicked

tips:
- a:hover MUST come after a:link and a:visited
- a:active MUST come after a:hover

#### links buttons
链接设置成按钮是多个属性的组合
```css
a:link, a:visited {
    background-color: #f44336;
    color: white;
    padding: 14px 25px;
    text-align: center; 
    text-decoration: none;
    display: inline-block;
}

a:hover, a:active {
    background-color: red;
}
```
### Lists
#### list-style-type
列表风格，可选值有
- circle
- square
- upper-roman
- lower-roman
- lower-alpha
- upper-alpha

也可以使用自定义图片
```css
ul {
    list-style-image: url('sqpurple.gif');
}
```

#### list-style-position
定义列表项的位置, 展示缩进效果，可选值有
- inside
- outside （default）

#### shorthand property
`list-style` 代表一下属性的顺序值
- list-style-type
- list-style-position
- list-style-image
```css
ul {
    list-style: square inside url("sqpurple.gif");
}
```
### Display
控制元素的布局。`display`默认值一般为`block`或`inline`。

Block-level Elements example:
- `<div>`
- `<h1> - <h6>`
- `<p>`
- `<li>`
- `<form>`
- `<header>`
- `<footer>`
- `<section>`

Inline Elements example:
- `<span>`
- `<a>`
- `<img>`

#### Override The Default Display Value
每个元素都有默认的显示值，但为了到达某种效果，也可以改写。
eg: 将分行的列表改为水平菜单
```css
li {
    display: inline;
}
```
同理也可以将`inline`元素改写为`block`显示

#### display:none
不显示某元素有两种方式
- display:none
- visibility:hidden

区别在于前者不占据空间，后者仍然占据着位置只是不显示

#### Inline-block
`display: inline-block`让块级元素inline展示。

### Position
元素定位。`position`可选值有
- static (default)
- relative
- fixed
- absolute

先设置position，再配合`top`, `right`, `bottom`, `left`等定位方法的值，可实现元素的定位

#### static
网页元素默认都是`static`的。
它的位置不受`top`, `right`, `bottom`, `left`定位方法的影响。
#### relative
位置设为`relative`后，它的位置可相对它原来的位置变化。
```css
div.relative {
    position: relative;
    top: 5px;
    border: 3px solid #73AD21;
}
```
此时，div的位置会相对原来`下移`5px。
#### fixed
位置设为`fixed`后，元素相对页面的位置将不再改变即使页面发生滚动，常见的应用场景有页面下拉时产生一个“回到顶部”的按钮。
此时的`top`, `right`, `bottom`, `left`都是用来辅助定位的， 一般使用其中的两个即可完成定位。
```css
div.fixed {
    position: fixed;
    bottom: 0;
    right: 0;
    width: 300px;
    border: 3px solid #73AD21;
}
```
例子中的div元素会被定位到右下角位置。
#### absolute
位置设为`absolute`后，元素的定位会相对它最近的已被定位(not static)的父元素，而不是像`fixed`那样相对于整个页面。然而如果不存在已被定位的父元素，就是使用文档的body，它就将随着页面滚动而变化。此时的`top` 等方法也是协助定位的。
```css
div.absolute {
    position: absolute;
    top: 80px;
    right: 0;
    width: 200px;
    height: 100px;
    border: 3px solid #73AD21;
}
```
此时相对父元素下移80px,右侧与父元素重合。
#### Overlapping Elements
重叠元素的位置设置使用`z-index`属性。值越大优先显示在上层。
如果重叠元素没有设置`z-index`属性，那么最后定位的元素将被显示在最顶部。
```css
img {
    position: absolute;
    left: 0px;
    top: 0px;
    z-index: -1;
}
```

#### overflow
设置当元素内容超过元素边界的行为，一般可选值为
- hidden 不显示
- scroll 横竖均显示滚动条
- auto 自动显示滚动条

### Float
`float`决定元素是否应该浮动，与之对应的有`clear`,用来清除浮动。
tip： 
- float 只是水平方向的控制。
- 使用float通常设置body的margin和padding为0以避免浏览器视图差异。
- IE8及更低版本在使用float时会在右侧预留17px的margin滚动条空间，所以需设置`!DOCTYPE` 防止差异，原因和盒模型的差异相同。
#### float
可选值有`left`或`right`
```css
img {
    float: right;
    margin: 0 0 10px 10px;
}
```
#### clear
浮动会产生内容重叠，使用clear可产生换行避免该问题。

#### clearfix
当一个元素高于它的容器，且该元素浮动，则显示时会溢出容器，此时给容器添加`overflow: auto`，可解决该问题。

### Align
- 块元素水平居中，使用`margin: auto;`
- 块元素内容居中，使用`text-align: center`
- 居左或居右，使用`postition`以及`left`/`right`或`float`

### Combinators
连接符，用来描述选择器之间的关系。
css3中有四种连接符：
- descendant selector (space)
- child selector (>)
- adjacent sibling selector (+) 
- general sibling selector (~)

eg：
```css
/* div下所有的p元素 */
div p {
    background-color: yellow;
}
/* div下所有的p子元素 */
div > p {
    background-color: yellow;
}
/* div后第一个p元素 */
div + p {
    background-color: yellow;
}
/* div后所有的p元素 */
div ~ p {
    background-color: yellow;
}
```

### Pseudo-classes
伪类，定义元素的不同状态。
语法为
```css
selector:pseudo-class {
    property:value;
}
```
- hover，常用于`<a>`,也可用于其他元素
- first-child, 第一个元素
- last-child, 最后一个元素
- nth-child(n), 第n个元素

### Pseudo-elements
伪元素，通常用作修饰某一元素的特定部分。
语法为,区别于伪类，使用双引号
```css
selector::pseudo-element {
    property:value;
}
```
- first-line, 修饰第一行

```css
p::first-line {
    color: #ff0000;
    font-variant: small-caps;
}
```
- first-letter, 修饰第一个字母

```css
p::first-letter {
    color: #ff0000;
    font-size: xx-large;
}
```
- before, 在元素前插入一些内容

```css
h1::before {
    content: url(smiley.gif);
}
```
- after, 在元素后插入一些内容
- selection, 改变选中文本的风格
```css
/* 选中文本为颜色为红色，背景为黄色 */
::selection {
    color: red; 
    background: yellow;
}
/* Code for Firefox */
::-moz-selection { 
    color: red;
    background: yellow;
}
/* IE8及以下版本不支持该属性 */
```

### Attribute Selectors
- CSS [attribute] Selector

```css
/* 仅选中包含target属性的a元素 */
a[target] {
    background-color: yellow;
}
```
- CSS [attribute="value"] Selector

```css
a[target="_blank"] { 
    background-color: yellow;
}
```
- CSS [attribute~="value"] Selector, 属性值包含某个具体值，The value has to be `a whole word`

```css
/* 选中title中包含"flower"的元素 */
[title~="flower"] {
    border: 5px solid yellow;
}
```
- CSS [attribute|="value"] Selector, 起始值匹配，The value has to be `a whole word`


```css
/* 选中class起始值为top的元素 */
/* 不会匹配class="topclass"的元素*/
[class|="top"] {
    background: yellow;
}
```
- CSS [attribute^="value"] Selector，起始值匹配，起始值包含value值即可。
- CSS [attribute$="value"] Selector，结束值匹配。
- CSS [attribute*="value"] Selector，全位置匹配，包含value值即可