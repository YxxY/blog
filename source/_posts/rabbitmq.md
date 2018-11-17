---
title: 消息队列之RabbitMQ
date: 2017-06-30 20:11:38
categories:
- RabbitMQ
tags: 
- RabbitMQ
- Node.js
---

最近项目上开始部署了消息队列，手头的工作基本围绕着它展开。消息队列已经是比较成熟的技术了，例如使用中我们最终选择的rabbitMQ，已经诞生10年了。那这是我们选择它的原因吗？不全是，还因为它的库多……总结下这块内容以备忘！
<!--more-->

## 使用场景
我工作中的使用主要是使用MQ重构了原来的需求，改进的主要是
- 和其他各个子模块之间的通信接口更加清晰，不再是只有主动发和被动收，而是收发都是主动。
- 去掉了一些中间消息的存库，改为请求后直接监听响应，减少了MongoDB的读写压力。

仿佛和主流描述的使用场景刚好相反，他们说的是立即返回响应，节省请求响应时间，让MQ去处理异步事件，而我是在等异步结果回来再响应……不过工具终究是为解决实际问题的，MQ的引入确实解耦了系统，降低了复杂度，对于一些不需要二次处理结果的事件我也的确是提前给它返回了响应，这是最理想的情况，只是并非所有使用场景都是如此。

## 名词介绍
- `broker`: 按官网说法`RabbitMQ is the most widely deployed open source message broker`, 简单来说 `rabbitMQ <= broker`, 即我们通常说的broker指的是rabbitMQ服务器本身。
- `exchange`: 消息交换机，消息发送者消息都是发到交换机，由交换机根据规则转发到队列，如果没有对应队列消息会被丢弃
- `routing key`: 消息发送者指定的规则，告诉交换机如何转发消息, 有时未显性展示，routing key其实为队列名
- `binding key`: 队列和交换机之间建立的规则，只有当routing key的规则符合该规则，交换机才会把消息转发至该队列
- `queue`: 队列，消息接收载体，是交换机和消息接收者之间的中间件，和交换机按照binding key的规则连接，如未显性展示，一般是绑定默认交换机，binding key为队列名，一个队列可以有多个消息接收者，这个时候的消息分配是round-table形式循环的。
- `producer`：消息生产者，就是投递消息的程序。
- `consumer`：消息消费者，就是接受消息的程序。
- `vhost`: broker的最小组成粒度，一个broker可以有多个vhost
- `connection`: 程序和broker之间的TCP连接，最大连接数取决于系统能够接受的socket连接数
- `channel`: 基于connection的虚拟连接，断开重连不需要重新建立TCP连接，减少了开销。一个connection里能建的最大channel数可以设置，服务端默认128，设为0代表无限制。channel是消息真正的操作载体，绝大多数操作都是基于channel的。
- `heartbeat`: 用于检测客户端与服务端之间的tcp链接是否正常的发包间隔，服务端默认600秒，客户端如果设置，双方会连接后会协定以客户端为主，如果客户端设置为0表示不启用该功能。

## 结构图
![rabbitmq](http://opxo4bto2.bkt.clouddn.com/rabbitmq.png)

## 应用
rabbitMQ官网以6个例子当做入门教程，详见[RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html)，提供多种客户端接口，跟着走一遍就能入门了。

官方的六个实例基本概括了rabbitMQ的使用场景，熟悉以后我觉得总的来说可以分为两部分，即生产者和消费者。
生产者不用管自己发出的消息被谁处理，怎么处理，只需要发给按约定规则（routing key）发到指定exchange即可，消费者按照自己的需求去订阅消息，不用关心消息的来源。让双线联系变成和MQ的单线联系，从而达到解耦的目的。大部分场景只要分清当前程序是producer还是consumer，逻辑就能变得简单。
有时需要consumer返回处理完消息的结果，即consumer需要动态的切换成producer，这个就是RPC的应用场景了，重点还是分清自己的角色就好了。

### Producer
    如果当前程序是Producer，那么只需关心自己和`exchange`之间的交互即可。
exchange 有四种类型，分别对应不同的使用场景，
- `direct`: 一对一模式，一般以queue的名字作为key，向指定queue转发消息
- `fanout`: 广播模式，向所有绑定的queue转发消息
- `topic`:  匹配模式，binding key可以使用通配符去匹配routing key，从而获得转发消息。`*`匹配一个关键字，`#`匹配多个关键字
- `header`: 一种类似topic的扩展，完全可以用topic取代

需注意的是，发送时一般会先确认exchange存在，如果传递的exchange option和实际不符会引发错误，导致channel关闭

### Consumer
    当前程序是Consumer，监听`queue`即可。
queue的创建和绑定可以协商由谁来完成，二次确认时主要保持参数一致，否则也会产生错误，导致channel关闭。
RPC模式我认为更像是消费者模式的一种。
消息处理程序监听在一个协定好的队列(queue1)上，收到消息时处理完之后，发送结果到消息里指定的队列(queue2)。
对于消息的发送者，先将消息发出,保证消息能被转发至queue1，随后监听在队列queue2，等待处理结果返回。
这里涉及到的两个队列，queue1和queue2的创建和绑定由谁来做都可以，一般为了方便就直接采取direct的消息转发机制了。

## API使用
我选用的是javascript客户端，用的包是[amqplib](https://github.com/squaremo/amqp.node), API参考[AMQP 0-9-1 library and client for Node.JS](http://www.squaremobius.net/amqp.node/channel_api.html#channel_bindExchange)
总结使用中遇到的一些坑：
- 最容易出现是exchange和queue二次确认时，参数不一致导致channel关闭
- queue的exclusive参数，表示的是该队列是此次连接私有，一旦启用其他的channel消费该队列会出错。往该队列发送消息没有问题，因为发送是通过exchange的
- 主动删除队列时，监听队列会收到一条null消息，需特殊处理
- consumer其实就是一个注册回调函数，它是一直都在的，并不是一次消费就失效，除非channel关闭，连接断开。避免在不知情的情况下给队列绑定多个consumer
- 多个consumer绑定同一队列时，消息的分配是round-table形式的，当被分到不是自己的消息或处理失败时需要nack来决定重新分配还是丢弃
- 确定需要一个queue多个consumer时记得使用`prefetch`参数，确保消息不会在一个节点累积

## 总结
我对rabbitMQ 学习也是一个从0到1的过程，入门很快，这也得益于它的良好API设计，一切都十分合理，是一个天然的编程模型。但项目上涉及的使用都很基础，暂时没遇到什么性能问题，估计遇到也不一定能解决……所以没啥可深入写的了，只能对一些概念性的东西作总结，方便以后能快速捡起来。




