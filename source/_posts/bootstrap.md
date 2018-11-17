---
title: Bootstrap 
date: 2017-05-23 21:05:27
categories:
- CSS
- Bootstrap
tags: 
- bootstrap
- css
---
bootstrap已经流行很久了，流行到能被吐槽只有小白才会使用，老实说我认可这种吐槽，但同时觉得这也是对bootstrap一种很高的赞誉。我从未系统看过bootstrap，每次都是拿来主义，需要啥就去找，这次仅从使用上总结下。
<!--more-->

# 背景
`bootstrap`中文释义类似“引导程序”, 是一个基于HTML和css的库，提供一些风格独特的html元素模板，同时也提供一些可选的javascript插件，总体来说是，它是一个工具库，具体来说是一个css类库。
## 特点
- 使用简单，只需基本html和css常识就能使用
- 响应式特性，适应各做场景使用
- 优先移动端实现，从bootstrap3开始，优先移动端风格是核心设计之一
- 浏览器兼容，兼容现代浏览器

## 使用
简单的使用是下载目标文件或使用CDN，包括三个文件，一个css文件或js文件，外加一个依赖的jQuery的js文件

# 工具类
## Container
容器为两类
- `.container`， 提供一个响应式的占据固定宽度容器
- `.container-fluid`，提供一个完整宽度的容器，宽度取决于浏览器能有多宽

`.container`不能嵌套使用
## Grid
网格系统，将整个页面最大分成12列。因此， 如果span12代表整个页面宽度，那么span6即代表页面一半的宽度，一次类推，span4，span3，span1，都有代表其对应的宽度。
网格有四类单位，区分不同的使用场景
- xs - 手机
- sm - 平板
- md - 桌面
- lg - 大屏幕桌面

至此，网格类语法可总结为`.col-单位-宽度`,eg:
- .col-xs-6
- .col-md-6

使用时，先添加一个`.row`类定义行，再根据需求添加列
```html
<div class="row">
  <div class="col-md-6">.col-md-6</div>
  <div class="col-md-6">.col-md-6</div>
</div>
```
## Table
在原始table上增加修饰类，首先需在`<table>`元素上填加`.table`基础类
eg: `<table class="table">`
### .table-striped
添加斑马线效果，随后增加`.table-striped`即可
eg:`<table class="table table-striped">`
### .table-bordered
边框效果，默认无边框，添加此类增加边框效果
### .table-hover
鼠标悬停选中效果
### .table-condensed
内容压缩效果
### .table-responsive
- 将任何 .table 元素包裹在 .table-responsive 元素内，即可创建响应式表格，其会在小屏幕设备上（小于768px）水平滚动。当屏幕大于 768px 宽度时，水平滚动条消失
- 响应式表格使用了 overflow-y: hidden 属性，这样就能将超出表格底部和顶部的内容截断。特别是，也可以截断下拉菜单和其他第三方组件。

eg:
```html
<div class="table-responsive">
  <table class="table">
    ...
  </table>
</div>
```
### content status
给表格内容添加背景色
- `.active`	    鼠标悬停在行或单元格上时所设置的颜色
- `.success`	标识成功或积极的动作
- `.info`	    标识普通的提示信息或动作
- `.warning`	标识警告或需要用户注意
- `.danger`	    标识危险或潜在的带来负面影响的动作

```html
<!-- On rows -->
<tr class="active">...</tr>
<tr class="success">...</tr>

<!-- On cells (`td` or `th`) -->
<tr>
  <td class="warning">...</td>
  <td class="danger">...</td>
  <td class="info">...</td>
</tr>
```
## Images
图片处理
### .img-rounded
圆角处理，IE8不支持该属性
```html
<img src="xxx.jpg" class="img-rounded" alt="Cinque Terre" width="304" height="236">
```
### .img-circle
将图片转换成圆形，不支持IE8
### .img-thumbnail
缩略图处理，加了个外边框
### .img-responsive
原理是设置`display:block`, `max-width: 100%;`, `height: auto;`
### .embed-responsive
嵌套内容响应式显示。例如:
- `<iframe>`
- `<video>`
- `<embed>`

eg:
```html
<div class="embed-responsive embed-responsive-16by9">
  <iframe class="embed-responsive-item" src="..."></iframe>
</div>
```
### 照片墙
结合网格系统，可以实现照片墙的效果
```html
 <div class="row">
  <div class="col-md-4">
    <a href="Picture1.jpg" class="thumbnail">
      <p>Picture1</p> 
      <img src="Picture1.jpg" alt="x" style="width:150px;height:150px">
    </a>
  </div>
  <div class="col-md-4">
    <a href="Picture2.jpg" class="thumbnail">
      <p>Picture2</p>
      <img src="Picture2.jpg" alt="xx" style="width:150px;height:150px">
    </a>
  </div>
  <div class="col-md-4">
    <a href="Picture3.jpg" class="thumbnail">
      <p>Picture3</p> 
      <img src="cinqueterre.jpg" alt="xxx" style="width:150px;height:150px">
    </a>
  </div>
</div>
```

## Jumbotron
巨幕，bootstrap标志性的文本渲染。给文本加上半透明背景，同时增大字体达到强调的效果。
一般结合container使用，如果不是放在某个容器内，jumbotron则扩展至屏幕宽度
```html
<div class="container">
  <div class="jumbotron">
    <h1>Bootstrap Header</h1> 
    <p>Bootstrap is the most popular HTML, CSS, and JS framework for developing
    responsive, mobile-first projects on the web.</p> 
  </div>
  <p>This is some text.</p> 
  <p>This is another text.</p> 
</div>
```
### .page-header
类似巨幕里的大标题，`.page-header`实现一个大标题以及一条水平分隔线
```html
<div class="page-header">
  <h1>Example Page Header</h1>
</div>
```
### .well
类似巨幕里的背景渲染。
```html
<div class="well">Basic Well</div>
```
另外还可增加尺寸修饰
- .well-sm 小尺寸
- .well-lg 大尺寸

```html
<div class="well well-lg">Large Well</div>
```

## Alerts
消息提示框，基础类为`.alert`,后续修饰继续添加到后面。
### 显示提示
```html
<div class="alert alert-success">
  <strong>Success!</strong> Indicates a successful or positive action.
</div>
```
提示级别有四类:
- `.alert-success`, 绿色背景，表示成功操作
- `.alert-info`,    蓝色背景，表示提示内容
- `.alert-warning`, 橙色背景，表警示
- `.alert-danger`,  粉色背景，错误信息

### 关闭提示
添加一个包含`class="close"`, `data-dismiss="alert"` 的链接或者按钮接口, 点击关闭
```html
<div class="alert alert-success">
  <a href="#" class="close" data-dismiss="alert" aria-label="close">&times;</a>
  <strong>Success!</strong> Indicates a successful or positive action.
</div>
```
### 动态效果
`.fade`以及`.in`增加关闭时消失的动态效果(效果不是很明显)
```html
<div class="alert alert-success fade in">
```

## Buttons
按钮，提供多种形态的按钮选择。基础类是`.btn`。
### Button Styles
- `.btn-default` - 默认白底黑字
- `.btn-primary` - 蓝底白字
- `.btn-success` - 绿底白字
- `.btn-info`    - 蓝底白字
- `.btn-warning` - 橙底白字
- `.btn-danger`  - 红底白字
- `.btn-link`    - 蓝色链接，无按钮形态

按钮类可用在 `<a>`, `<button>`, `<input>`上，效果相同，某些情况下布局会有差异。用法如下
```html
<a href="#" class="btn btn-info" role="button">Link Button</a>
<button type="button" class="btn btn-info">Button</button>
<input type="button" class="btn btn-info" value="Input Button">
<input type="submit" class="btn btn-info" value="Submit Button">
```
### Button Sizes
和列分类类似，定义不同的按钮大小
- `.btn-xs`
- `.btn-sm`
- `.btn-md`
- `.btn-lg`
### Block Level Buttons
按钮默认是`inline`元素，可以转换成`block`级别，增加`.btn-block`类即可
### Active/Disabled Buttons
- `.active`类修饰按钮按下时的状态，形态会从蓝色转为深蓝色
- `.disabled`类修饰按钮禁用状态，鼠标不可点击

```html
<button type="button" class="btn btn-primary active">Active Primary</button>
<button type="button" class="btn btn-primary disabled">Disabled Primary</button>
```
## Button Groups
定义多个按钮时，按钮直接会有缝隙，但定义按钮组则不会, 基础类为`.btn-group`，表示水平按钮组，垂直按钮组则使用`.btn-group-vertical`基础类。
### Button Groups Styles
```html
<div class="btn-group">
  <button type="button" class="btn btn-primary">Apple</button>
  <button type="button" class="btn btn-primary">Samsung</button>
  <button type="button" class="btn btn-primary">Sony</button>
</div>
```
### Button Groups Sizes
`.btn-group-*` 修饰类决定所有按钮大小，eg: `.btn-group-lg`
```html
<div class="btn-group btn-group-lg">
  <button type="button" class="btn btn-primary">Apple</button>
  <button type="button" class="btn btn-primary">Samsung</button>
  <button type="button" class="btn btn-primary">Sony</button>
</div>
```
### Button Groups Justified
`.btn-group-justified` 修饰类让按钮扩展到整个屏幕宽度
此修饰类不能修饰`.btn-group-vertical`基础类
```html
<div class="btn-group btn-group-justified">
  <a href="#" class="btn btn-primary">Apple</a>
  <a href="#" class="btn btn-primary">Samsung</a>
  <a href="#" class="btn btn-primary">Sony</a>
</div>
```
如果使用`btn-group-justified`和`<button>`,则需包装一层按钮组，如下
```html
<div class="btn-group btn-group-justified">
  <div class="btn-group">
    <button type="button" class="btn btn-primary">Apple</button>
  </div>
  <div class="btn-group">
    <button type="button" class="btn btn-primary">Samsung</button>
  </div>
  <div class="btn-group">
    <button type="button" class="btn btn-primary">Sony</button>
  </div>
</div>
```
### Nesting Button Groups & Dropdown Menus
按钮组可以嵌套按钮组，实现下拉菜单。
```html
<div class="btn-group">
  <button type="button" class="btn btn-primary">Apple</button>
  <button type="button" class="btn btn-primary">Samsung</button>
  <div class="btn-group">
    <button type="button" class="btn btn-primary dropdown-toggle" data-toggle="dropdown">
    Sony <span class="caret"></span></button>
    <ul class="dropdown-menu" role="menu">
      <li><a href="#">Tablet</a></li>
      <li><a href="#">Smartphone</a></li>
    </ul>
  </div>
</div>
```
### Split Button Dropdowns
分裂的按钮下拉菜单
```html
<div class="btn-group">
  <button type="button" class="btn btn-primary">Sony</button>
  <button type="button" class="btn btn-primary dropdown-toggle" data-toggle="dropdown">
    <span class="caret"></span>
  </button>
  <ul class="dropdown-menu" role="menu">
    <li><a href="#">Tablet</a></li>
    <li><a href="#">Smartphone</a></li>
  </ul>
</div>
```

## Glyphicons
字体图标。Glyphicons提供250+免费的字体图标给bootstrap使用。
syntax:
```html
<span class="glyphicon glyphicon-name"></span>
```
## Badges and Labels
- `.badge`修饰类让背景显示为一个小圆圈。通常是显示一个数字，与其他内容配合使用。

```html
<a href="#">Updates <span class="badge">2</span></a>
<button type="button" class="btn btn-primary">Primary <span class="badge">7</span></button>
```
- `.label`修饰类，相当于将修饰内容打一个标签，标签大小自适应
- label也可以增加状态类，类似于alert

```html
<h1>Example <span class="label label-default">New</span></h1>
<span class="label label-default">Default Label</span>
<span class="label label-primary">Primary Label</span>
<span class="label label-success">Success Label</span>
<span class="label label-info">Info Label</span>
<span class="label label-warning">Warning Label</span>
<span class="label label-danger">Danger Label</span>
```

## Progress Bar
进度条,IE9及以下的浏览器不支持，因为他们不支持css3的转换和动画。
基础类为`.process-bar`。
### Basic Progress Bar
```html
<div class="progress">
  <div class="progress-bar" role="progressbar" aria-valuenow="70"
  aria-valuemin="0" aria-valuemax="100" style="width:70%">
    <span class="sr-only">70% Complete</span>
  </div>
</div>
```
`.sr-only`具体是否在进度条上显示内容

### Colored Progress Bars
进度条也有状态类，分别渲染了不同的颜色
- `.progress-bar-success`
- `.progress-bar-info`
- `.progress-bar-warning`
- `.progress-bar-danger`

颜色依次是绿蓝橙红。eg：
```html
<div class="progress">
  <div class="progress-bar progress-bar-success" role="progressbar" aria-valuenow="40"
  aria-valuemin="0" aria-valuemax="100" style="width:40%">
    40% Complete (success)
  </div>
</div>
```
### Striped Progress Bars
`.progress-bar-striped`类增加条纹效果，配个`.active`类，可以让条纹有动态效果
```html
<div class="progress">
  <div class="progress-bar progress-bar-striped active" role="progressbar"
  aria-valuenow="40" aria-valuemin="0" aria-valuemax="100" style="width:40%">
    40%
  </div>
</div>
```
### Stacked Progress Bars
堆叠的进度条，如果把多个进度条让在一个容器里，会有堆叠的效果

### js plugin
进度条的增减是修改`style`属性里的`width`属性值实现的。可使用js动态修改css的方法实现进度条的更新。jquery操作方式如下：
```js
$(selector).css("width", value + "%").text(value + "%"); 
```

## Pagination
分页导航
### Basic Pagination
通常是由一个`<ul>`元素和`.pagination`基础类组成。
- `.active`类修饰选中效果
- `.disabled`类修饰禁用效果

```html
<ul class="pagination">
  <li><a href="#">1</a></li>
  <li class="active"><a href="#">2</a></li>
  <li><a href="#">3</a></li>
  <li><a href="#">4</a></li>
  <li class="disabled"><a href="#">5</a></li>
</ul>
```
### Pagination Sizes
分页导航的大小
- `.pagination-lg` 大导航
- `.pagination-sm` 小导航

### Pager
提供上一页和下一页选项。分别由`.previous`和`.next`类修饰
```html
<ul class="pager">
  <li class="previous"><a href="#">Previous</a></li>
  <li class="next"><a href="#">Next</a></li>
</ul>
```
### Breadcrumbs
面包屑导航，即分层次导航
```html
<ul class="breadcrumb">
  <li><a href="#">Home</a></li>
  <li><a href="#">Private</a></li>
  <li><a href="#">Pictures</a></li>
  <li class="active">Vacation</li> 
</ul>
```

## List Groups
一个列表组通常由`<ul>`, `.list-group` 和 `<li>`, `.list-group-item`组成。
```html
<ul class="list-group">
  <li class="list-group-item">First item</li>
  <li class="list-group-item">Second item</li>
  <li class="list-group-item">Third item</li>
</ul>
```
- `<li>`元素也可替换为`<a>`
- `.active`类修饰选中效果
- `.disable`类修饰禁用效果
- `.list-group-item-*`类修饰状态，仍然是`success`,`info`, `warning`和`danger`

### Custom Content
修饰自定义内容风格
- `.list-group-item-heading` 自定义内容头
- `.list-group-item-text` 自定义内容主体

```html
<div class="list-group">
  <a href="#" class="list-group-item active">
    <h4 class="list-group-item-heading">First List Group Item Heading</h4>
    <p class="list-group-item-text">List Group Item Text</p>
  </a>
  <a href="#" class="list-group-item">
    <h4 class="list-group-item-heading">Second List Group Item Heading</h4>
    <p class="list-group-item-text">List Group Item Text</p>
  </a>
  <a href="#" class="list-group-item list-group-item-danger disable">
    <h4 class="list-group-item-heading">Third List Group Item Heading</h4>
    <p class="list-group-item-text">List Group Item Text</p>
  </a>
</div>
```

## Panels
面板。原理是一个带有边框的盒子，并且设置了一定的padding。
- 基础类为`.panel`,
- 标题修饰类为`.panel-heading`
- 内容主体修饰类为`.panel-body`
- 尾部修饰类为`.panel-footer`
- 自定义名称修饰类`.panel-title`
- 颜色状态类为`.panel-*`,分别有`default`,`primary`,`info`,`success`,`warning`,`danger`
- 面板组修饰类`.panel-group`,类似按钮组的逻辑

```html
<div class="container">
  <div class="panel panel-danger">
    <div class="panel-heading">
      header
    </div>
    <div class="panel-body">
      body
    </div>
    <div class="panel-footer">
      footer
    </div>
  </div>
</div>
```

## Dropdown
下拉菜单， 基础类为`.dropdown`
- 打开菜单一般是使用一个链接或按钮，该元素带有`.dropdown-toggle`类及属性`data-toggle="dropdown"`。
- `.caret`类修饰一个小的箭头图标。表示该按钮为下拉按钮
- 一个带有`.dropdown-menu`类的`<ul>`元素构成下拉菜单内容选项
- `.dropdown-header`类修饰菜单头
- `.divider`类修饰分隔符
- `..disabled`类修饰禁用菜单内容
- `.dropdown-menu-right`类可让菜单栏右侧对齐(strange behavior)
- `.dropup`类替换dropdown可让菜单栏向上拉

```html
<div class="dropdown">
    <button class="btn btn-default dropdown-toggle" type="button" data-toggle="dropdown">Dropdown Example
    <span class="caret"></span></button>
    <ul class="dropdown-menu dropdown-menu-right">
      <li class="dropdown-header">Dropdown header 1</li>
      <li><a href="#">CSS</a></li>
      <li><a href="#">JavaScript</a></li>
      <li class="divider"></li>
      <li class="dropdown-header">Dropdown header 2</li>
      <li ><a href="#">About Us</a></li>
    </ul>
  </div>
</div>
```

## Collapse
折叠显示。当需要隐藏或显示大量内容时非常实用。基础类为`.collapse`
- 折叠开关一般是使用一个链接或按钮，该元素带有属性`data-toggle="collapse"`以及`data-target="#id"`,该`id`指向基础类修饰的需要折叠的内容id。如果使用`<a>`元素，`data-target`可用`href`代替
- 折叠的内容默认不显示，但添加`.in`类可默认显示类容

```html
<a href="#demo" data-toggle="collapse in">Collapsible</a>

<div id="demo" class="collapse">
Lorem ipsum dolor text....
</div>
```

### Collapsible Panel
可折叠的panel，基础类为`panel-collapse`
```html
<div class="container">
  <div class="panel panel-default">
      <div class="panel-heading">
        <h4 class="panel-title">
          <a data-toggle="collapse" href="#collapse1">Collapsible panel</a>
        </h4>
      </div>
      <div id="collapse1" class="panel-collapse collapse panel-primary in">
        <div class="panel-heading">Panel Head</div>
        <div class="panel-body">Panel Body</div>
        <div class="panel-footer">Panel Footer</div>
      </div>
  </div>
</div>
```

### Collapsible List Group
借助`Collapsible Panel`的list group
```html
  <div class="container">
    <div class="panel-group">
      <div class="panel panel-default">
        <div class="panel-heading">
          <h4 class="panel-title">
            <a data-toggle="collapse" href="#collapse1">Collapsible list group</a>
          </h4>
        </div>
        <div id="collapse1" class="panel-collapse collapse">
          <ul class="list-group">
            <li class="list-group-item">One</li>
            <li class="list-group-item">Two</li>
            <li class="list-group-item">Three</li>
          </ul>
          <div class="panel-footer">Footer</div>
        </div>
      </div>
    </div>
  </div>
```

### Accordion
手风琴折叠，通过使用`data-parent`属性，让每个折叠子选项指向同一父元素，从而达到当其中子选项展开时，上一个展开的选项自动折叠，从而到达类似手风琴的折叠效果。
```html
<div class="panel-group" id="accordion">
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapse1">
        Collapsible Group 1</a>
      </h4>
    </div>
    <div id="collapse1" class="panel-collapse collapse in">
      <div class="panel-body">Lorem ipsum dolor sit amet, consectetur adipisicing elit,
      sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
      minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
      commodo consequat.</div>
    </div>
  </div>
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapse2">
        Collapsible Group 2</a>
      </h4>
    </div>
    <div id="collapse2" class="panel-collapse collapse">
      <div class="panel-body">Lorem ipsum dolor sit amet, consectetur adipisicing elit,
      sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
      minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
      commodo consequat.</div>
    </div>
  </div>
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapse3">
        Collapsible Group 3</a>
      </h4>
    </div>
    <div id="collapse3" class="panel-collapse collapse">
      <div class="panel-body">Lorem ipsum dolor sit amet, consectetur adipisicing elit,
      sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
      minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
      commodo consequat.</div>
    </div>
  </div>
</div>
```

## Tabs
- 网页中的标签通常定义在一个无序列表`<ul>`元素中，默认是垂直方向的
- 如果想创建一个水平方向的菜单，添加`.list-inline`类至`<ul>`元素即可
- 除了自定义方式之外，也可以使用bootstrap 的标签类，基础类为`.nav`,`.nav-tabs`

eg:
```html
<ul class="nav nav-tabs">
  <li class="active"><a href="#">Home</a></li>
  <li><a href="#">Menu 1</a></li>
  <li><a href="#">Menu 2</a></li>
  <li><a href="#">Menu 3</a></li>
</ul>
``` 

### Tabs With Dropdown Menu
```html
<ul class="nav nav-tabs">
  <li class="active"><a href="#">Home</a></li>
  <li class="dropdown">
    <a class="dropdown-toggle" data-toggle="dropdown" href="#">Menu 1
    <span class="caret"></span></a>
    <ul class="dropdown-menu">
      <li><a href="#">Submenu 1-1</a></li>
      <li><a href="#">Submenu 1-2</a></li>
      <li><a href="#">Submenu 1-3</a></li> 
    </ul>
  </li>
  <li><a href="#">Menu 2</a></li>
  <li><a href="#">Menu 3</a></li>
</ul>
```

### Pills style Tabs
- 胶囊风格的标签，基础类为`.nav-pills`
- 垂直风格的胶囊标签，增加修饰类`.nav-stacked`
- 下拉菜单的实现同普通菜单
- 居中分布的标签，使用修饰类`.nav-justified`

```html
<ul class="nav nav-pills nav-stacked">
  <li class="active"><a href="#">Home</a></li>
  <li><a href="#">Menu 1</a></li>
  <li><a href="#">Menu 2</a></li>
  <li><a href="#">Menu 3</a></li>
</ul>
```

### Toggleable / Dynamic Tabs
给标签加上自动切换的效果
- 给每一个链接增加`data-toggle="tab"`属性, 实现点击时动态切换`.active`类
- 可以指定切换后的内容展现，`href`属性为指向内容的`id`
- 指定内容需被包裹在一层`<div>`中，并且带有`.tab-pane`类
- 还可以添加`.fade`, `.in` 等修饰类增加动态效果
- 如果是pills，`data-toggle="pill"`, 其它与tabs一致

```html
<div class="container">
  <ul class="nav nav-pills">
    <li class="active"><a data-toggle="pill" href="#home">Home</a></li>
    <li><a data-toggle="pill" href="#menu1">Menu 1</a></li>
    <li><a data-toggle="pill" href="#menu2">Menu 2</a></li>
  </ul>
  <div class="tab-content">
    <div id="home" class="tab-pane fade in active">
      <h3>HOME</h3>
      <p>Some content.</p>
    </div>
    <div id="menu1" class="tab-pane fade">
      <h3>Menu 1</h3>
      <p>Some content in menu 1.</p>
    </div>
    <div id="menu2" class="tab-pane fade">
      <h3>Menu 2</h3>
      <p>Some content in menu 2.</p>
    </div>
  </div>
</div>
```

## Navigation Bars
导航栏，导航与标签的区别不是很大，我个人的理解是导航即放在特定位置的菜单。
- 基础类为`.navbar`
- 颜色修饰类有`.navbar-default`,`.navbar-primary`,`.navbar-inverse`
- 位置修饰类有`.navbar-fixed-top`, `.navbar-fixed-buttom`,固定导航栏的位置
- 内容修饰类`.navbar-header`,`.navbar-brand`,`.navbar-nav`
- 针对内容`.navbar-nav`的修饰，还可以追加`.navbar-right`类让导航条浮动在右边
- 也可以结合dropdown做下拉菜单导航
- 隐藏导航条，在`.collapse`基础类上增加`.navbar-collapse`类,一般用于在窗口小于一定宽度时折叠导航, 

```html
<nav class="navbar navbar-inverse">
  <div class="container-fluid">
    <div class="navbar-header">
      <button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#myNavbar">
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>                        
      </button>
      <a class="navbar-brand" href="#">WebSiteName</a>
    </div>
    <div class="collapse navbar-collapse" id="myNavbar">
      <ul class="nav navbar-nav">
        <li class="active"><a href="#">Home</a></li>
        <li class="dropdown">
          <a class="dropdown-toggle" data-toggle="dropdown" href="#">Page 1 <span class="caret"></span></a>
          <ul class="dropdown-menu">
            <li><a href="#">Page 1-1</a></li>
            <li><a href="#">Page 1-2</a></li>
            <li><a href="#">Page 1-3</a></li>
          </ul>
        </li>
        <li><a href="#">Page 2</a></li>
        <li><a href="#">Page 3</a></li>
      </ul>
      <ul class="nav navbar-nav navbar-right">
        <li><a href="#"><span class="glyphicon glyphicon-user"></span> Sign Up</a></li>
        <li><a href="#"><span class="glyphicon glyphicon-log-in"></span> Login</a></li>
      </ul>
    </div>
  </div>
</nav>
```

## Forms
bootstrap有三种类型的表单。
- Vertical form (default, `<label>`元素独占一行) 
- Horizontal form, `<form>`元素添加`.form-horizontal`类，`<label>`元素添加`.control-label`类
- Inline form, `<form>`元素添加`.form-inline`类

一些标准规则如下：
- 总是添加`<form role="form">`, 对非桌面环境友好
- 将`label`和`form`元素包裹在`<div>`元素中，如`<div class="form-group">`，从而保证合适的空白空间
- 给所有的文本输入元素如`<input>`, `<textarea>`, `<select>`添加`.form-control`类，保证它们宽度均为100%

```html
<form class="form-inline" role="form">
  <div class="form-group">
    <label for="email">Email address:</label>
    <input type="email" class="form-control" id="email">
  </div>
  <div class="form-group">
    <label for="pwd">Password:</label>
    <input type="password" class="form-control" id="pwd">
  </div>
  <div class="checkbox">
    <label><input type="checkbox"> Remember me</label>
  </div>
  <button type="submit" class="btn btn-default">Submit</button>
</form>
```

## Modal 
```html
<div class="container">
  <!-- Trigger the modal with a button -->
  <button type="button" class="btn btn-info btn-lg" data-toggle="modal" data-target="#myModal">Open Modal</button>

  <!-- Modal -->
  <div id="myModal" class="modal fade" role="dialog">
    <div class="modal-dialog">

      <!-- Modal content-->
      <div class="modal-content">
        <div class="modal-header">
          <button type="button" class="close" data-dismiss="modal">&times;</button>
          <h4 class="modal-title">Modal Header</h4>
        </div>
        <div class="modal-body">
          <p>Some text in the modal.</p>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
        </div>
      </div>

    </div>
  </div>
</div>
```
开关部分:
- `data-toggle="modal"`打开模态框
- `data-target="#myModal"` 指向模态框内容部分的id

框体部分:
- 模态框最外层的`<div>`必须有一个id，作为开关的指向
- 基础类为`.modal`, `.fade`修饰类增加消逝效果
- `.modal-dialog`类设置合适的宽度和margin

主体内容部分:
- 最外层是一个带有`.modal-content`类的`<div>`，包裹着模态框的header,body和footer
- `.close`类修饰关闭按钮，`&times;`是关闭图标的实体符号
- `.modal-title`类修饰header标题
- `.modal-body`,`.modal-footer`分别修饰内容和框底
- 给`<button>`按钮添加`.data-dismiss="modal"`属性可关闭模态框
- 模态框大小在`.modal-dialog`类上添加修饰类`modal-lg`或`.modal-sm`

## Tooltip
提示框。
- 给一个元素添加`data-toggle="tooltip"`属性即可创建一个提示框
- `title`属性的类型为具体的提示内容
- tooltip功能必须被jquery初始化，即选中具体的元素，调用它的`tooltip()`方法
- `data-placement`属性决定提示框的位置，属性值有上下左右

```js
//js初始化
<script>
$(document).ready(function(){
    $('[data-toggle="tooltip"]').tooltip(); 
});
</script>
```

## Popover
和提示框类似
- `data-toggle="popover"`创建，需要js初始化
- 提示内容在`data-content`中
- `data-placement`决定位置
- `data-trigger="hover"`,弹窗只在悬浮时呈现，如果值为`focus`,需要点击空白区域才能关闭

```js
<script>
$(document).ready(function(){
    $('[data-toggle="popover"]').popover(); 
});
</script>
```

## Scrollspy
滑动监听，切换导航。
- 给需要滚动部分的元素添加`data-spy="scroll"`属性，通常是`<body>`元素，该元素的`position`css值需为`relative`才能生效。
- 再添加`data-target`属性，属性值为导航条的id或者类，从而让滚动和导航连接起来
- 导航链接必须匹配滑动元素内部内容的id
- `data-offset`是个可选属性，用来微调滑动切换时的位移，默认10px

## Affix
允许一个元素变成某个区域或页面的固定部分。通常用在导航条，下拉时固定在顶端。
示例将一个水平导航下拉时固定在顶部：
- 给`<nav>`元素添加`data-spy="affix"`属性以及`data-offset-top`属性
- `data-offset-top`的属性值即为触发条件的下拉位移，同理还存在`data-offset-bottom`属性

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">

    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>主页</title>
    <link href="./css/bootstrap.min.css" rel="stylesheet">

      <style>
      body {
          position: relative; 
      }
      .affix {
          top:0;
          width: 100%;
          z-index: 9999 !important;
      }
      .navbar {
          margin-bottom: 0px;
      }

      .affix ~ .container-fluid {
         position: relative;
         top: 50px;
      }
      #section1 {padding-top:50px;height:500px;color: #fff; background-color: #1E88E5;}
      #section2 {padding-top:50px;height:500px;color: #fff; background-color: #673ab7;}
      #section3 {padding-top:50px;height:500px;color: #fff; background-color: #ff9800;}
      #section41 {padding-top:50px;height:500px;color: #fff; background-color: #00bcd4;}
      #section42 {padding-top:50px;height:500px;color: #fff; background-color: #009688;}
      </style>

    </head>
    <body data-spy="scroll" data-target=".navbar" data-offset="20">

    <div class="container-fluid" style="background-color:#F44336;color:#fff;height:200px;">
      <h1>Scrollspy & Affix Example</h1>
      <h3>Fixed navbar on scroll</h3>
      <p>Scroll this page to see how the navbar behaves with data-spy="affix" and data-spy="scrollspy".</p>
      <p>The navbar is attached to the top of the page after you have scrolled a specified amount of pixels, and the links in the navbar are automatically updated based on scroll position.</p>
    </div>

    <nav class="navbar navbar-inverse" data-spy="affix" data-offset-top="197">
      <div class="container-fluid">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#myNavbar">
              <span class="icon-bar"></span>
              <span class="icon-bar"></span>
              <span class="icon-bar"></span>                        
          </button>
          <a class="navbar-brand" href="#">WebSiteName</a>
        </div>
        <div>
          <div class="collapse navbar-collapse" id="myNavbar">
            <ul class="nav navbar-nav">
              <li><a href="#section1">Section 1</a></li>
              <li><a href="#section2">Section 2</a></li>
              <li><a href="#section3">Section 3</a></li>
              <li class="dropdown"><a class="dropdown-toggle" data-toggle="dropdown" href="#">Section 4 <span class="caret"></span></a>
                <ul class="dropdown-menu">
                  <li><a href="#section41">Section 4-1</a></li>
                  <li><a href="#section42">Section 4-2</a></li>
                </ul>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </nav>    

    <div id="section1" class="container-fluid">
      <h1>Section 1</h1>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
    </div>
    <div id="section2" class="container-fluid">
      <h1>Section 2</h1>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
    </div>
    <div id="section3" class="container-fluid">
      <h1>Section 3</h1>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
    </div>
    <div id="section41" class="container-fluid">
      <h1>Section 4 Submenu 1</h1>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
    </div>
    <div id="section42" class="container-fluid">
      <h1>Section 4 Submenu 2</h1>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
      <p>Try to scroll this section and look at the navigation bar while scrolling! Try to scroll this section and look at the navigation bar while scrolling!</p>
    </div>


    <script src="./js/jquery.min.js"></script>
    <script src="./js/bootstrap.min.js"></script>

</body>

</html>
```