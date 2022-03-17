---
layout: post
title:  "socket-websocket-soap一次讲通"
summary: "网络协议、socket/websocket/soap/webservice讲解！"
author: hana
date: '2022-03-01'
category: ['network']
tags: jekyll
thumbnail: /assets/img/posts/web.jpg
keywords: learn
usemathjax: false
permalink: /blog/socket/
---


## 网络体系机构的通信协议
### 基本认识
![img](/assets/img/posts/w.png)


1. 以上都是协议，每个协议有自己的默认端口。
2. HTTP：无状态，只能由客户端发起。
3. Socket：是操作系统提供的对于传输层（TCP/UDP）抽象的接口，是一个编程概念，是应用层与TCP/IP协议族通信的中间软件抽象层，他是一组接口。Socket是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后见，对用户来说，一组简单的接口就是全部。让Socket去组织数据，以符合指定的协议。当两台主机通信时，必须通过Socket连接，也就是：应用层-->Socket--->传输层。
4. WebSocket：双向通信协议。无回溯限制；跟HTTP一样需要握手进行建立连接，websocket在建立连接时，数据通过HTTP传输，建立之后，在真正传输时不需要HTTP协议。

### webservice
> 问题：有a、b两台机器，a的程序如何被b的主机获取？拿就要跨编程语言，跨操作系统才能实现，也就是用webservice可以实现。

webservice是一种跨编程语言和跨操作系统平台的远程调用技术。从表面上看，webservice时一个应用程序向外界暴露出一个能通过web进行调用的API，从深层次看，webservice是建立可互操作的分布式应用程序的新平台，时一个平台，一套标准。

那么，既然他是平台，那怎么实现分布式应用的创建？你想想，任何平台都有他的数据表示方法和类型系统，要实现互操作性，webservice就要提供一种标准来描述seb service。好了，有了描述，怎么实现远程调用，这个方法是一种远程调用协议（RPC），RPC还必须与平台编程语言无关。

#### webservice平台技术：XML+XSD、SOAP、WSDL
`概念`
* SOAP = HTTP协议+XML数据格式
* XML是webservice平台中表示数据的格式
* XSD是来规范XML中数据类型的。
* SOAP (Simple Object Access Protocol），是个协议
* WSDL（Web Services Description Lauguage）一个基于XML的语言，用于描述Web Service及其函数、参数和返回值。

`细节`
webservice通过HTTP协议发送请求和接受结果时，发送的请求内容和结果都采用XML格式，并增加了一些特定的HTTP消息头，以说明HTTP消息的内容格式，这些特定的HTTP消息头和XML内容格式就是SOAP协议。SOAP提供了标准的RPC方法来调用web service。

SOAP请求是HTTP POST的一个专用版本，遵循一种特殊的XML消息格式，Content-Type设置为：text/xml。

WSDL是WebService客户都安和服务器都能理解的标准格式。WSDL文件保存在Web服务器上，通过一个url地址就可以访问到它，WebService服务提供商可通过两种方式来暴露他的WSDL文件地址：
1. 注册到UDDI服务器，以便被查找
2. 直接告诉客户端调用者


