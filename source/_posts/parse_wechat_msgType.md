---
title: 对微信消息内容类型的判断
date: 2017-05-15 18:35:38
categories:
- Lua
tags: 
- wechat-message
- lua
---
最近试着做些工作总结，这也是我开始写博客的初衷，少bb，just do it！

这是我最早进公司做的一个小项目，需求是判断手机收发微信消息内容类型。就是手机发了一条微信消息之后，程序要能解析出发的是文本、语音还是图片等类型，并且统计时延。接收也是如此，主要需求是对发送类型的判断。
<!--more-->

## 实现步骤
	1. 抓包
	2. 解包
	3. 结果展示

- - -
### 特殊说明
	该工作主要依赖部门内部高度定制的解析工具，可复现度几乎为0，因此全文只谈思路、方法与收获。

- - -
### 抓包
抓取终端发生微信消息时的消息包，根据3GPP协议可以获取该消息的id为0x11EB

- - -
### 解包
这个过程就是把二进制的码流按照协议解析成有意义的字段。再从中筛选实际需要的内容做二次解析。
这部分按照当时使用的工具，解析语言用的lua。
这里主要说明下数据的过滤和解析
#### 过滤数据
如何判断数据是否有效？答案是`根据协议数据结构和自身业务需求`。
根据TCP/IP协议，数据包的格式是`Header + Data`
这里具体来说就是，`ipHeader + tcpHeader + data`，其中data部分才是我们真正需要的解析目标。
- - -
贴一下数据包的数据结构,分别对应IP数据包格式和TCP数据包格式，图片来源于网络
。更多内容可参考这篇文章[TCP协议](http://www.ruanyifeng.com/blog/2017/06/tcp-protocol.html)
![ip](http://opxo4bto2.bkt.clouddn.com/pic/lua/wechat/IP_Package.png)

![tcp](http://opxo4bto2.bkt.clouddn.com/pic/lua/wechat/TCP_Package.png)

根据协议，很清楚可以看到ipHeader和tcpHeader的固定部分都是20个字节，因此协议规则如下：
_ _ _
- 	ipHeader 至少20个字节
- 	tcpHeader 至少20个字节
加上和业务定制的规则如下：
- ipHeader 中的版本（0-4位对应的字段值）为4，即0100，用来表明IP协议实现的版本号，当前规定为IPv4，因此如果是连wifi发微信消息，那么就直接被过滤掉了
- ipHeader中的协议（72-80位）为6，指明IP层所封装的上层协议类型，如ICMP（1）、IGMP（2） 、TCP（6）、UDP（17）等，当前为TCP
- data部分至少16个字节，这个是查资料获得，和微信消息类型相关的特征字信息，只在前16个字节中，后续字节内容应该表示的消息内容
   
总的来说过滤数据，就是把不符合这五条规则的数据直接丢掉不处理，直接进入下一条log的处理。
_ _ _

#### 解析数据

假设我们已经获取数据，并过滤，得到了我们需要的data，下面开始处理，这里贴出当时写的很难看的处理函数。
```lua
function getDataTypeFromDataContent(mo, buf)
    local ls = stat[mo]
    local str, dataSN = '', ''
    for i = 1, 12, 1 do
    	str = str..buf[i]..' '
    end
    for i = 13, 16, 1 do
    	dataSN = dataSN..buf[i]
    end
    ls.dataSNText = dataSN
 	
    if string.match(str, "00 00 00 82 00 10 00 01 %x%x %x%x %x%x ED") then
    	ls.dataType = 'Text'..'-'..'Send OK'
    elseif string.match(str, "00 00 %x%x %x%x 00 10 00 01 %x%x %x%x %x%x ED") then
    	ls.dataType = 'Text'..'-'..'Send'
    elseif string.match(str, "00 00 00 7E 00 10 00 01 %x%x %x%x %x%x 13") then
    	ls.dataType = 'Voice'..'-'..'Send OK'
    elseif string.match(str, "00 00 00 81 00 10 00 01 %x%x %x%x %x%x 13") then
    	ls.dataType = 'Voice'..'-'..'Send OK'
    elseif string.match(str, "00 00 00 83 00 10 00 01 %x%x %x%x %x%x 13") then
    	ls.dataType = 'Voice'..'-'..'Send OK'
    elseif string.match(str, "00 00 %x%x %x%x 00 10 00 01 %x%x %x%x %x%x 13") then
    	ls.dataType = 'Voice'..'-'..'Send'
    elseif string.match(str, "AB 00 %x%x %x%x %x%x 27 14 00") then
        ls.dataType = 'Img'..'-'..'Request'
    elseif string.match(str, "AB 00 %x%x %x%x %x%x 2A FC 00") then
        ls.dataType = 'Img'..'-'..'Request OK'
    elseif string.match(str, "AB 00 %x%x %x%x %x%x 27 10 00") then
        ls.dataType = 'Img'..'-'..'Send'
    elseif string.match(str, "AB 00 %x%x %x%x %x%x 2A F8 00") then
        ls.dataType = 'Img'..'-'..'Send OK'
    elseif string.match(str, "AB 00 %x%x %x%x %x%x 75 30 00") then
        ls.dataType = 'Video'..'-'..'Send'    
    elseif string.match(str, "AB 00 %x%x %x%x %x%x 79 18 00") then
        ls.dataType = 'Video'..'-'..'Send OK' 
    elseif string.match(str, "00 00 %x%x %x%x 00 10 00 01") then
        ls.dataType = 'WeChat'..'-'..'unknown'
    end
end
```
这种全是if/else的代码着实难堪的，嗯，很符合当时的水平,说明如下：

	ls 为状态对象，是一个状态机的实现，用来记录每个手机实时的消息状态。
	将data数组拼接成字符串，以空格为间隔。
    可以看出，前12个字节用于判断类型，后四个字节用来记录序列号。
	然后匹配对应各自的消息类型。
    这些规则也是查阅资料论文和测试获得，微信也暂未公开这些信息。
		
_ _ _
至此，已经可以获得消息类型了。当然只是常用的几种，文本，图片、语音及视频类型。但这里只完成了40%的任务，因为还需要统计发送消息的时延啊，例如发了一条消息，会马触发一条`Text-Send`的log，但直到有`Text-Send OK`的log上报，才会认定消息发送成功，二者之间的时间差就是需要记录的时延。

但理想很丰满，上面的消息判断依旧是很粗糙的，总有一些意外情况，例如`Send-OK`消息的格式总是有意外情况，图片会有分包的情况，如何判断是一次而不是多次发送等等问。

所以后来第二个迭代时就完全结束了这种不完全靠谱的做法，直接自定义两条log，在发送之前发一次，里面写好发送类型手机id等信息，发送完毕再发一条结束消息确认。这样来，一切就简单多了，相当于固定场景测试了。

- - -


### 总结
总结下此次开发任务的收获和心得体会

- 熟练掌握了lua的基本语法和使用，体验到了动态语言的优越性。

- 对IP数据包的结构有了基本认识，一切都是站在协议的肩膀上完成的

- 学会了如何分拆将一个稍微大点的任务分拆成小块，快速挖掘关注核心部分

- 收集资料的也是一种能力，懂业务比写代码实现更重要









