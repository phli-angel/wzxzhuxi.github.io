---
layout: post
title: "网络安全技术-抓包技术"
author: phli
date: 2025-11-20
categories: [网络安全]
tags: [网络安全, Charles, WireShark, BurpSuite, proxifier]
excerpt: >
  介绍主要的抓包软件与使用方法，涵盖 HTTP/HTTPS 协议抓包，包括 Fiddler、Charles、Burp Suite、Wireshark 等工具的特点与应用场景。
---

# 网络安全技术-抓包技术

目前对于抓包技术而言的通信协议主要为http/https，http/https在web应用使用最为广泛，本文将介绍一些主要的抓包软件与使用方法

### HTTP/HTTPS

* 准备工作
  * 证书安装（HTTPS）
    * 模拟器--->开发者模式安装证书（burp, charles, fidder）
    * 浏览器（设置内搜索证书后导入证书文件）
* 抓包工具
  * fidder(工具中勾选https选项)
  * charles
  * burpsuite(配置代理)
* 转发工具
  * proxifier
    * 建立代理服务器(如果没有代理选项)
    * 设置代理规则
      * 基于进程筛选
      * 基于IP域名筛选
      * 基于端口筛选
* 抓包类型
  * 浏览器WEB应用（burp需要配置代理）
  * APP应用（安装证书后配置代理）
  * 小程序/PC应用（burp 可配合charles/proxifier/系统代理）
* 其他协议（应用：APP/小程序/PC应用）
  * 科来(应用分类/应用进程/网络接口IP)
  * WireShark(只能通过语法筛选)
  * 封包监听（抓包通讯地址/封包发送测试） 

### Fiddler

https://www.telerik.com/fiddler

是一个http协议调试代理工具，它能够记录并检查所有你的电脑和互联网之间的http通讯，设置断点，查看所有的“进出”Fiddler的数据（指cookie,html,js,css等文件）。 Fiddler 要比其他的网络调试器要更加简单，因为它不仅仅暴露http通讯还提供了一个用户友好的格式。

### Charles

https://www.charlesproxy.com/

是一个HTTP代理服务器,HTTP监视器,反转代理服务器，当浏览器连接Charles的代理访问互联网时，Charles可以监控浏览器发送和接收的所有数据。它允许一个开发者查看所有连接互联网的HTTP通信，这些包括request, response和HTTP headers （包含cookies与caching信息）。

### TCPDump

是可以将网络中传送的数据包完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息。

### BurpSuite

是用于攻击web 应用程序的集成平台，包含了许多工具。Burp Suite为这些工具设计了许多接口，以加快攻击应用程序的过程。所有工具都共享一个请求，并能处理对应的HTTP 消息、持久性、认证、代理、日志、警报。

### Wireshark

https://www.wireshark.org/

是一个网络封包分析软件。网络封包分析软件的功能是截取网络封包，并尽可能显示出最为详细的网络封包资料。Wireshark使用WinPCAP作为接口，直接与网卡进行数据报文交换。

---

**作者**: Phli  
**日期**: 2025-11-20  
**版本**: 2.0