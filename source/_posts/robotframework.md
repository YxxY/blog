---
title: 自动化测试环境搭建
date: 2017-7-13 21:15:27
categories:
- Python
- RobotFramework
tags: 
- robotframework
- jenkins
---

在我有限的自动化测试经历中，我接触了基于robotframework测试框架+jenkins管理平台的一整套CI测试流程，受益良多。
验收测试驱动(ATDD)的思想确实有助于保证产品质量，提高开发效率。总结下这块的学习经验，以后应该还用得上。
<!--more-->
## robotframework
简称`RF`,
详细介绍可以参考[官网](http://robotframework.org) 

代码托管在github [robotframework](https://github.com/robotframework/robotframework)

总的来说RF是一个为验收测试和验收测试驱动开发的自动化测试框架，技术上以关键字驱动实现测试用例的编写。

特点是易于使用，易扩展，结构层次分明，整个生态异常庞大。


### 环境搭建
[官方安装文档](https://github.com/robotframework/robotframework/blob/master/INSTALL.rst#introduction)

RF是用python实现的，同时也支持jython和IronPython，大部分项目应该是直接使用python

此处当然又会遇到python的世纪难题，到底是py2还是py3。

先说结论，还是py2吧

原因有两点
- RF 从v3.0开始才支持py3，但依旧支持py2
- 虽然官网推荐生态圈中的库和工具也应该开始支持py3，然而支持度还不够全面，例如官方编辑器`RIDE`依然仅支持py2，目前没看到有支持py3的计划。虽然可以选择其他编辑器，但对于新手来说不够友好，还是建议先使用RIDE的GUI界面，方便快速入门

#### 安装RF
- 安装python v2.7及以上版本
- 安装robotframework, v3.0及以上版本. `pip install robotframework`
- 确认已配置环境变量
- 验证安装是否成功
```shell
robot --version
rebot --version
```

命令行工具介绍
- `robot` 执行测试用例
- `rebot` log和report文件生成器。将生成的`output.xml`记录输出文件二次解析成可读性更强的`log.html`和`report.html`结果文件。作用相当于XML解析器

详细信息可通过
`robot--help` 和 `rebot --help` 查看

#### 安装编辑器
测试用例一般编写完成保存在文件中。

支持多种格式的文件，例如`txt`,`csv`等。
更推荐带有标识的后缀名，例如`tests.robot`

具体编写时，推荐使用官方编辑工具 [RIDE](https://github.com/robotframework/RIDE/wiki)

官方安装文档见 [RIDE Installation](https://github.com/robotframework/RIDE/wiki/Installation-Instructions)

总的来说就是
- 安装wxPython依赖
- 安装ride

启动编辑器
- 运行ride.py
- 指定参数 ride.py path/to/tests
- windows上双击桌面快捷方式

#### 执行测试用例
执行测试用例命令
```shell
robot [options] data_source
python -m robot [options] data_source
python path/to/robot [options] data_source
```
所有的optsions参数可通过`robot --help`命令查看

常用的有：
- `-t --test name *`, 选取指定用例名来执行测试，默认为`*`,匹配所有用例
- `-s --suite name *`, 同上，通过测试套名指定
- `-i --include tag *`, 通过标签指定
- `-e --exclude tag *`, 排除特定标签的用例
- `-d --outputdir dir`, 输出目录，默认为当前测试用例执行的路径
- `-o --output file`, 输出文件名，默认为`output.xml`, 设置为`NONE`可禁用输出。其他log、report等输出都是基于该文件，
- `-r --report file`, 测试报告名，默认`report.html`, 设置为`NONE`不输出，也可以通过`rebot`工具后续再输出
- `-l --log file`, 测试log文件名， 同上


`python -m robot`到RF3.0版本才支持

旧版本需要使用`python -m robot.run`

解析xml也同理
```shell
rebot output.xml
python -m robot.rebot output.xml
python -m robot.rebot -d ./output output.xml
```
## Jenkins
[jenkins](https://jenkins.io/)的logo很好的说明的它的作用，简单来说它就是一个“管家”，可以定制各种任务的构建，测试，部署等各种工作的开源自动化持续集成服务器。

### 安装
- 安装Java8
- 下载最新jenkins资源包`jenkins.war`
- 启动`java -jar jenkins.war --httpPort=8080`
- 打开浏览器网址，根据提示完成安装
- 插件下载地址  [plugins](http://updates.jenkins-ci.org/download/plugins/)

## 总结
RF 实现了测试主体，并提供了便捷的命令行启动接口。

jenkins是任务执行者，定制任务然后执行启动RF测试的命令，二者相结合实现了端到端的完整测试流程

RF创建的测试可以说是验收测试，也可以算是单元测试和系统测试。总之，测试保证了开发质量，提升了开发效率


