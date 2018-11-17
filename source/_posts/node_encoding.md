---
title: Node.js 中的编码问题
date: 2017-09-3 20:25:14
categories:
- JavaScript
tags: 
- encoding
- Node.js
---

编码问题本不是一个问题，直到它主动找上你，它就是个大问题
<!--more-->

Node用到的默认编码格式是utf8，支持的编码格式在[Buffer API](https://nodejs.org/api/buffer.html#buffer_buffers_and_character_encodings)中可见，
默认支持的有![supported_encoding_type](http://opxo4bto2.bkt.clouddn.com/node_encoding_type.png)

很显然没有windows常用的GBK编码，那么这样就很容易产生两种不同的编码格式，只能一方迁就另一方。
之前的一个小项目就遇到这样一个问题，需要用户提供数据，但要求一个只会excel记录数据的普通人提供utf8编码的json格式数据实在太难为他了。最后折中的办法采用了CSV格式，原因无它，支持excel打开编辑。
但仍然存在一个问题，excel编辑完保存，会以默认非uf8编码格式保存，自带的保存转换也不生效，只能用一个土办法，记事本打开，另存为时指定utf8编码格式。但仅多这一个步骤也让用户叫苦不迭，偶尔忘记转码就会导致中文显示乱码。
因此，这个问题不得不解决。


# 浅谈编码
老实说，我对编码的了解无穷趋近于一无所知，所以真的是浅谈。我只知道编码不同是以不同的二进制数字表示字符，具体什么规则我真的是记不住，也无心去了解。从Node操作文件的API，我大胆来`猜测`一下它的工作流程。
读取文件时，API不管是`fs.readFile` 还是`fs.createReadStream` 
encoding默认均为null，即无编码，那就是不做任何处理，将码流原封不动的读取出来暂存。
但处理数据时，将buffer数据转为字符串处理时，默认是utf8, 并且node未提供GBK的转码方式（支持的转码格式见上图），那么问题就出现了，如果初始编码格式不是utf8，用utf8格式去解码，当然会出问题，就算我不懂编码我也知道，用一把锁的钥匙去开另一把不同的锁，打不开才是正常的。

# 从源头解决问题
当出现乱码问题时，要知道问题的源头是用了解码格式与编码格式不匹配造成的，所有解决的问题就要两种
- 改文件的编码方式，与解码格式匹配
- 该文件解析时的解码方式， 与编码格式匹配

第一种是让数据的生产者去改，前面已经说过了，代价太大，成本过高，不切实际。
第二种就更头疼了，我也没办法让node支持GBK的转码啊。
因此，只能曲线救国了，中间加一层转码，得到用户的数据后先把它从GBK的码流转成utf8的码流，再进行解析操作，完美！

# 实践
搜一下能将将gbk转成utf8格式的包，搜到一个`iconv-lite`, 测试可用。

```js
var iconv = require('iconv-lite')

var readableStream = fs.createReadStream(remote)
var writableStream = fs.createWriteStream(local)

readableStream.on('data',(chunk)=>{
    var unicodeString = iconv.decode(chunk, 'GBK')
    var buf = Buffer.from(unicodeString)
    if (!writableStream.write(buf)) {
        readableStream.pause()
    }
})

writableStream.on('drain', function() {
    readableStream.resume()
});

readableStream.on('end', function() {
    writableStream.end()
});
```

核心就两句，先将码流以GBK的解码方式解成字符串，这很符合逻辑，GBK编码，GBK解码。
`var unicodeString = iconv.decode(chunk, 'GBK')`
再将字符串再转换成utf8码流。
`var buf = Buffer.from(unicodeString)`
你没有猜错，Buffer.from默认utf8编码，node中的输出应该都是默认这样的操作。

# tips
这种写法必须保证数据来源必须始终是gbk编码的，如果提供了一个utf8编码的就会出问题，那怎么解决呢？说过了我也不懂。
但应该能很自然想到有没有一种方式判断文件的编码方式呢，尝试搜了一下，果然还是有的，测试可用。

```js
var fs=require('fs')

fs.readFile('code.txt',function(err,buffer){

    if(buffer[0]==0xff&&buffer[1]==0xfe){
        console.log('unicode')
    }else if(buffer[0]==0xfe&&buffer[1]==0xff){
   　　console.log('unicode big endian')
    }else if(buffer[0]==0xef&&buffer[1]==0xbb){
　　　 console.log('utf8')
    }else{
   　　console.log('else')
    }
})
```
差不多意思就是，文件开头的2个字节可以判断是utf8还是unicode，虽然没有GBK的判断方法，但排除法勉强能解决问题。
因此，之前的代码可以重构下，先判断文件编码格式，如果是utf8就不用转码了，如果不是那就认为是gbk，需要转码。
如果仍然出现了乱码问题，那就甩锅给用户，肯定是他们打开的方式不对，绝对不是我的bug！

# 总结
这次经历对编码、解码、转码的概念基本了解了。
- 编码：字符串-> 码流
- 解码：码流 -> 字符串
- 转码：码流 -> 字符串 -> 码流
也就是说目前还没有把一种编码格式的码流直接转成另一种的手段，只能用相应的解码方式先还原，再重新编码，这也是符合逻辑的。

