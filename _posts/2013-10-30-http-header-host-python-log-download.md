---
layout: post
title: "下载不同服务器相同域名下日志文件"
description: "Intellij IDEA Eclipse Document"
category: python 
tags: [python, http]
---

### 前言
最近遇到一个问题：我们的应用部署在多台服务器上，偶尔需要把日志下载下来分析就需要手动更改本机hosts文件来连接到不同的服务器一遍遍的下载日志。本着懒人原则，如何更高效的从不同服务器上的相同域名上下载日志文件已经成为必须解决的问题。

### 分析
**过程：**下载日志文件需要发出一次http请求，如url: *http://jervyshi.me/logs/debug.log* ，每次http请求都会根据url中的domain去做DNS解析获取到对应的服务器ip，如果我们指定hosts，则在做DNS解析前优先读取hosts中配置的ip，获取到ip之后与对应服务器建立连接然后获取数据。

**方案一：**如果能根据domain对应的已知服务器列表在每次请求日志文件时候更改hosts文件中此domain的配置可以实现此功能，之前手动操作完全是按照此方法来，缺点很明显下载一个文件改一次hosts。如果用程序实现，在程序下载日志时hosts频繁更改，影响正常浏览器访问，此方案不佳。

**方案二：**如果能在读取本地hosts文件之前就已经指定ip，显然会方便很多。其实在一次http请求发出时，请求头中会包含 *Host* 信息，如果url中domain不是ip那么会把domain截取出来放进http请求头 *Host* 中，如：*http://192.168.1.100/logs/debug.log* 然后将本次请求的header中增加 *Host:jervyshi.me* ，这样就可以使程序通过切换ip来访问不同服务器上的domain。

### 实现
根据方案二，我用python做了一个简单的实现，可以通过配置来达到从不同服务器上获取相同域名下的日志文件并且分类保存。具体代码参见： *https://github.com/JervyShi/python-utils*
