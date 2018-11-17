---
title: shell script
date: 2017-11-5 18:57:38
categories:
- Linux
- Shell
tags: 
- shell-script
---
总结shell脚本
<!--more-->
## shell VS shell script
shell 是一种应用程序，负责将指令传达至linux内核。`terminl` 是一种输入输出的虚拟终端，本身并不会解析输入的命令，真正处理输入命令的是shell。
`shell script` 是为shell编写的脚本，本质是命令的集合，相对terminal输入的方式，shell脚本可以批量操作，可以有组合逻辑，能做的事更多，但需要符合一定的代码规范。学习shell脚本本质是掌握shell命令以及shell script的语法规则。本文着重总结后者。

## 声明
脚本第一行需是约定的声明， 例如：`#!/bin/bash`，作用是告诉系统用那种解释器，全常用的是bash，很多系统自带，当然也可定义其他版本shell。

## 脚本执行
1. 添加执行权限 `chmod +x filepath`
2. 执行脚本     `filepath arg1 arg2 arg3 ...`

## 脚本参数
- `$0` :   当前脚本文件名
- $n  :  传递给脚本或函数的参数，n是一个数字，表示第几个参数，`$1`表示第一个参数，`$2`表示第二个参数 ...
- `$#` :   传递给脚本或函数的参数个数
- `$*` :   传递给脚本或函数的所有参数，当它被双引号（" "）包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 ... $n"的形式输出所有参数
- `$@`:    传递给脚本或函数的所有参数，当它被双引号（" "）包含时，与`$*`稍有不同，"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数, `$*` 和`$@`不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数
- `$$`: 脚本运行的当前进程ID号
- `$?`: 上一个命令的退出状态，`或函数的返回值`，如果正常退出则返回0，反之为非0值

## 变量
脚本里常用数据类型分为三类，字符串、数字类型，数组。
### 字符串
如果不声明类型，默认为字符串。
赋值时`=`附近不能有空格（`shell中使用空格会带来很多诸如此类的问题，能不用就不用！`）
使用变量是时需再变量名前加上`$`,同时使用可选的`{}`表示变量边界
```sh
#!/bin/bash
a=8
b=9
echo $a     # ${a}
echo $a+$b  # 8+9 默认字符串，并不是整型
echo ${#a}  # 获取字符串长度
```
### 字符串拼接
直接连接，中间不能有空格
```sh
#!/bin/bash
a=hello
b=$a" world"
echo $b
```
#### ${} 高级用法
```sh
#!/bin/bash
file=/dir1/dir2/dir3/my.file.txt
#可以用${ }分别替换获得不同的值：
echo ${file#*/}   #拿掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt
echo ${file##*/}  #拿掉最后一个 / 及其左边的字符串：my.file.txt
echo ${file#*.}   #拿掉第一个 . 及其左边的字符串：file.txt
echo ${file##*.}  #拿掉最后一个 . 及其左边的字符串：txt
echo ${file%/*}   #拿掉最后一个 / 及其右边的字符串：/dir1/dir2/dir3
echo ${file%%/*}  #拿掉第一个 / 及其右边的字符串：(空值)
echo ${file%.*}   #拿掉最后一个 . 及其右边的字符串：/dir1/dir2/dir3/my.file
echo ${file%%.*}  #拿掉第一个 . 及其右边的字符串：/dir1/dir2/dir3/my
# `#` 去掉左边
# `%` 去掉右边
# 单一符号是最小匹配，两个符号是最大匹配。
echo ${file:0:5}  #提取最左边的 5 个字节：/dir1
echo ${file:5:5}  #提取第 5 个字节右边的连续 5 个字节：/dir2
#也可以对变量值里的字符串作替换：
${file/dir/path}  #将第一个 dir 替换为 path：/path1/dir2/dir3/my.file.txt
${file//dir/path} #将全部 dir 替换为 path：/path1/path2/path3/my.file.txt
```

变量可用`declare`关键字声明类型，但也可不用。
### 数组
```sh
#!/bin/bash

#通过下标声明
array[0]=a
array[1]=b
#或声明并统一赋值
array=(a,b,c)
#取数组中的值：
echo ${data[1]}   # 输出第1个元素
echo ${data}      # 不写默认是第0个
echo ${data[*]}   # 输出数组中所有的变量
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```

## 单双引号及反引号
字符串可以用单引号，也可以用双引号，也可以`不用引号`（默认字符串类型）。
单双引号的区别：
- 单引号里`任何字符`都会原样输出，内部变量无效
- 单引号内部不能再出现单引号（转义输出也不行）
- 双引号里可以有变量（相当于模板字符串）
- 双引号可以出现转义字符

反引号"`",用于命令替换(command subsitution)，执行引号内命令，返回结果填充
```sh
#!/bin/bash
#显示上周日的日期
echo the last sunday is `date -d "last sunday" +%Y-%m-%d`
echo the last sunday is $(date -d "last sunday" +%Y-%m-%d)
```
反引号移植性好，新的shell可使用`$()`，
区别之一:
- 反引号本身就对\进行了转义，保留了它本身意思，如果我们想在反引号中起到\的特殊意义，我们必须使用2个\来进行表示。
所以我们可以简单的想象成反引号中: `\\ = \`
- $()中则不需要考虑\的问题，与我们平常使用的一样: `\ = \`
示例如下：
```sh
#!/bin/bash
#eg:my host name is 'sora'
echo  `echo $HOSTNAME`   # sora
echo $(echo $HOSTNAME)   # sora 此时二者无区别

echo  `echo \$HOSTNAME`   # sora
echo $(echo \$HOSTNAME)   # $HOSTNAME

echo  `echo \\$HOSTNAME`  # $HOSTNAME
echo $(echo \\$HOSTNAME)  # \sora
```

## echo与printf
`echo` 用于字符串的输出
`echo string`
```sh
#!/bin/bash
# 转义输出
echo "\"It is a test\""  
# 读取一行输出，read命令会去除首尾空格
read name 
echo "$name It is a test"

# -e 开启特殊符号转义
echo -e "OK! \n" # 显示换行
echo -e "OK! \c" # 不换行
echo "test"
# 显示结果定向至文件
echo "It is a test" > myfile
```
`printf` 类似于c的用法，用于格式化字符串，标准定义，移植性更好
`printf  format-string  [arguments...]`
```sh
#!/bin/bash
# format-string为双引号
printf "%d %s\n" 1 "abc"

# 单引号与双引号效果一样 
printf '%d %s\n' 1 "abc" 

# 没有引号也可以输出
printf %s abc def

# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf "%s\n" abc def
```

## 数值运算
```sh
#!/bin/bash
# declare -i 变量名声明为数值型进行运算
a=1
b=2
declare -i c=$a+$b

# expr或let数值运算工具
a=1
b=2
c=$(expr $a + $b)  # + 号左右两侧必须有空格

# $((运算式))或 $[运算式]
a=1 
b=2
c=$(($a + $b))
d=$[$a + $b]   
```

## 流程控制
### if else
```sh
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi

# 写成一行需要分号分隔
if condition1; then command; fi
```
### for
```sh
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
# 写成一行
for var in item1 item2 ... itemN; do command1; command2… done;

# example
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```
### while
```sh
while condition
do
    command
done

# example，当int小于5时输出显示
int=1
while [ $int -lt 5 ]
do
    echo $int
    int=$[$int+1]
done
```

## 条件判断
常用的两种方式，`test`命令或`[]`
其中用`[]`时，开头和结尾必须有空格
```sh
#!/bin/bash

bit=`getconf LONG_BIT`
# 判断系统位数是否为64
if [ $bit = 64 ] # 注意前后空格, = 在判断相等时前后可以有空格，赋值时不行
then
    echo 'yes'
else
    echo 'no'
fi

# test命令的方式则是
# if test $bit = 64
```
### 整型数值比较
- -eq	等于则为真
- -ne	不等于则为真
- -gt	大于则为真
- -ge	大于等于则为真
- -lt	小于则为真
- -le	小于等于则为真
注意， 这些符合判断仅适用于整型
```sh
#!/bin/bash
a=123
b=45
if test $[a] -gt $[b] # 也可以用 `=` 判断
then
    echo 'yes'
else
    echo 'no'
fi
```
### 字符串判断
`operator string`
字符串相等，用`=`判断, 整型的判断在这里不适用,`除非字符串是整型数字`
- =	    字符串等于则为真
- !=	字符串不相等则为真
- -z    字符串的长度为零则为真
- -n 	字符串的长度不为零则为真
```sh
#!/bin/bash
a=''
if test -z $a # [ -z $a ]
then
    echo 'yes'
else
    echo 'no'
fi
```

### 文件名判断
`operator filename`
- -a或-e 如果文件存在则为真
- -d     如果文件存在且为目录则为真
- -f     如果文件存在且为普通文件则为真
- -r     如果文件存在且可读则为真
- -s     如果文件存在且至少有一个字符则为真
- -w     如果文件存在且可写则为真
- -x     如果文件存在且可执行则为真

### 逻辑运算符
- ||    Or
- &&    And
### 布尔运算符
- -a   与
- -o   或
- !    非
```sh
#!/bin/bash
cd /bin
if test -e ./notFile -o -e ./bash
# 用逻辑运算符写如下：
# if test -e ./notFile || test -e ./bash
then
    echo '有一个文件存在!'
else
    echo '两个文件都不存在'
fi
```

## 函数
必须先声明再调用
### 函数声明与调用
```sh
# 声明 关键字function可省略
function funcName(){
    statement
}
# 调用
func arg1 arg2 ...
```
### 函数参数
函数内部$n 表示传入的第n个参数,这里和脚本参数类似

### 函数返回值
return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后跟`数值`(0-255)
```sh
#!/bin/bash
test(){
    return 123
}
test
a="$?"
echo $a
```

## 常用命令
### 得到当前脚本目录
```sh
#!/bin/bash
dir="cd `dirname $0`; pwd"
echo $dir
```
### 判断由当前脚本启动的实例个数
```sh
ret=`ps -ef | grep "yourprocessname" | grep -v grep | grep -v "$(basename $0)" | wc -l`
echo $ret
```
### 嵌入文本(here document)
将两个`delimiter`之间的内容作为输入传递给command
```sh
command << delimiter
    document
delimiter
```
tips：
- 结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。
- 开始的delimiter前后的空格会被忽略掉。
```sh
#!/bin/bash
#示例
cat <<EOF
content to print
EOF
```