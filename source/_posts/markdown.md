---
title: markdown 总结
date: 2018-6-17 20:30:38
categories:
- Markdown
tags: 
- markdown
---

md也写了很久了，最近想倒腾一个自定义渲染风格的md解析器，所以回过头写一篇基础总结，确实发现了一些之前不知道的细节，就当一篇markdown的语法备忘。
<!--more-->

## 语法导航

翻译自[官方标准V0.28](https://spec.commonmark.org/0.28)，
官网举例非常详细，我仅翻译一个我觉得够用的版本。

### Space
- 多个空格会被视为一个空格
- 多个空行也会被渲染为一个（因此换行需要两个回车）
    
#### Tabs
- 根据空格的规则，一般情况下的tab键只会被渲染为一个空格

    但在块结构中会被解析为4个空格，比如代码块
- 和`列表标签`组合使用时，
    - 换行+`tab`+列表标签，产生子列表效果
    - 换行+一个`tab`能产生`缩进`效果
    - 换行+两个`tab`能产生`引用块`的效果

            block quote context

    -       列表标签直接跟两个`tab`同样产生`引用块`的效果
- 和`>`符号组合使用，`>`+两个`tab`，能产生缩进两格的代码块效果

    >       an indented code block starting with two spaces.
- 和`*`符号组合，`*   *   *   `为`分割线`效果

    *   *   *   
- 和`#`符合组合产生标题

### Blocks-and-inlines
- 块元素的渲染优先级高于行元素
- 块元素分为两种，一种继续包含其他块元素，称为`container-blocks`

    另外一种不能，称为`leaf blocks`

#### Leaf-blocks
##### Thematic-breaks
一行包括`0-3`个缩进，跟着连续的三个或者更多的 `“*”`， `“_”`，`“-”` 其中任意一个字符，中间和结尾都可以有任意数量空格，后续不可再跟其他非空白字符。即组成了专题隔断（`Thematic breaks`），也就是通常所说的`分割线`
>       ***
>       ---
>       ___

- 必须是三个或更多的相同字符，不可任意组合
- 分割线前后不需要空白行
- 分割线能分割段落
- 当使用`破折号`分隔符`"-"`分割段落时，会将前面的内容渲染成[Setext heading](#setext-headings)
- 使用`"*"`符号组成分隔符，如果遇到`*`开头的列表符号，分隔符优先级更高

    列表中使用分隔符需要使用不一样的符号，例如

        - Foo
        - * * *

##### ATX-headings
标题行也是一种`leaf blocks`，组成规则如下：
- `0-3`个空格缩进
- 组成标题行的符号为`#`, 根据标题级数可由`1-6`个组成
- `#`符号之后`必须有至少一个空格或者换行符`，然后才跟标题内容
- 结尾可选跟任意数量的`#`符号作为结束标志
- `#`符号前加`\`会被转义
- 标题内容安装普通文本解析
- 前后空白字符在解析时会被忽略
- 标题前后不需要空白行分割，会自动分割段落
- 标题可以为空 `# #`

##### Setext-headings
setext(Structure Enhanced Text) heading，强调标题

- 由一行或多行文本组成,每一行至少包含一个非空字符
- 每行可有0-3个空格缩进
- 文本行的最后跟`setext heading underline`(强调标题下划线符号)：`“=”`或`“-”`
- 去掉`setext heading underline`符号时，文本应该翻译为一个段落，因此使用的场景`不包括`代码块、标题、引用块、分割线、列表和html段落等
- 如果使用`=`，文本会被渲染为一级标题的格式
- 如果使用`-`，文本会被渲染为二级标题的格式
- 可以使用任意数量的`setext heading underline`，但中间不能有空格
- 使用`-`时，需要和`列表`的使用区分开

example
```md
Setext-headings level 1
===
Setext-headings level 2
---

```
##### Indented-code-blocks
- 一个缩进的代码块由一个或多个缩进块（indented chunks）组成
- 缩进块（indented chunks）是由一组非空行组成，每一行至少有`4空格`缩进
- 缩进块的内容就是输入的文字内容。行尾需要换行，行首的4空格缩进不会被渲染
- 当`缩进块`和`列表`同时出现时，列表的渲染级别更高

##### Fenced-code-blocks
- 由至少三个连续的反引号字符包裹, 行首可选不超过3个空格的缩进
- 围栏头部可选一个信息字符串(info string)，第一个单词表明代码的类型

#### Container-blocks
容器块，就是可以将其他块当做它的内容，这里有两种基本的容器块
- 块引用（block quotes）
- 列表（list items）

##### Block-quotes
块引用的组成为：0-3个缩进，加上一个`">"`符号，再加可选的空格

- `>`可单独一行
- `>`和引用内容之间的内容可省略
- 多个`>`代表多级引用
- 引用可和其他渲染格式组合使用，比如`>`+两个`tab`，形成引用代码块

##### List-items
分为有序和项目标记列表

- 项目标记列表的组成符号为`"*"`, `"-"`或 `"+"`三种
- 有序列表的组成为`0-9`数字，跟着一个点号或者反括号
- 列表符号和内容之间的至少有一个空格
- 多级列表相对上一级需要增加`多个`缩进
- 项目标记符号不同是中间会新增一个空行

        - one
        - two
        + three
    相当于如下代码：

        - one
        - two
        <!-- -->
        - three
- 存在多级标题时，上下级之间增加空行能增加行距

#### Inlines
##### Backslash-escapes
任何的ASCII 符号字符都能被反斜杠转义，必要的时候使用

##### Entity-and-numeric-character-references
所有的合法HTML实体引用以及数字符号引用，只要不是出现在代码块或者代码域中，都能被识别为对应的unicode字符。

常见的引用有：
- `&nbsp;` 代表空格
+ &amp; &copy; &AElig; &Dcaron;
&frac34; &HilbertSpace; &DifferentialD;
&ClockwiseContourIntegral; &ngE; 的表示依次如下

        &amp; &copy; &AElig; &Dcaron;
        &frac34; &HilbertSpace; &DifferentialD;
        &ClockwiseContourIntegral; &ngE;

##### Code-spans
由反引号包裹的字符串称之为`code span`, 中文翻译成“代码域”似乎有点别扭，将就吧，知道是啥就行。这个用的很多，通常用作单词的高亮处理

##### Emphasis-and-strong-emphasis
强调和着重强调，在markdown中由`*`和`_`标识符引导

这里面的细节规则很多，但普通用户掌握最基本的用法就好

- 示例：*普通强调* 

        *普通强调*
- 示例：**着重强调**

        **着重强调**
- 示例：***着重强调***

        ***着重强调***

##### Links
链接的规则为
- `[link_text](url "title")`, title可省略

    eg：

        [link](#fragment) 页面内跳转（全小写字母）
        [link](http://example.com#fragment)
- 当不需要说明，仅展示url文本的链接,由尖括号包裹链接即可

    `<whatever-url-it-is>`

##### Images
图片的引用和链接非常像, 前面多一个感叹号
- `![foo](/url "title")`


## More 

markdown是一种类似html的标记语言，让使用者更加专注于内容的编写，而不关心样式。但它仍然是一门`语言`。

之前介绍的都是语法。语法虽然简单，但使用时依然会遇到一些问题。

### 语法渲染差异
常见的场景就是选择了不同的markdown编辑器，在这个里面打开视图没问题，去另外的就会出现渲染问题。

- 比如标题符号`#`后面要不要紧跟空格  
eg:  `## 我是二级标题`  
`haroopad`的语法可带可不带，而`vscode`必须带空格
- haroopad支持高亮语法
eg: `==highlight==`  
vscode没有

这些看起来是编辑器差异，其实是markdown规范实现的差异

### markdown规范
啰嗦几句规范的意义。

语言规范就是标准的定义。

只要符合这个标准，具体的实现不限，当然还可以自行添加新特性。

markdown的基本规范是[CommonMark ](http://commonmark.org/help/)

规范也是可以不断更新，版本迭代的，截止到本文编写的时间，CommonMark的最新版本是`v0.28, 2017-08-01`

历史版本见[历史版本列表](https://spec.commonmark.org/)

规范只有一个，但实现可以有多种，markdown的实现也是一样，根据`commonMark`的官方统计，目前在册的已经有几十种了。具体可参考
[规范实现列表](https://github.com/commonmark/CommonMark/wiki/Markdown-Flavors)

之前的语法渲染问题的原因就是，vscode支持的markdown语法只是最基本标准的实现，haroopad根据规范有一套自己的实现，也就是[Haroopad Flavored Markdown](http://pad.haroopress.com/page.html?f=haroopad-flavored-markdown)

## 总结
因此，编辑器的选择只是表象。
如果只是编写标准的markdown，那用什么编辑器都一样。
然而如果选择的是某一特定版本的实现，则需要参考相对应的使用说明，以及选择对应的渲染方式

当写markdown时，一定要清楚自己写的是哪一种实现，一般来说写标准的肯定没错，但一些有特色的特性也很有吸引力，如果使用时务必要使用对应的解析和渲染手段才能达到预期效果。
