---
layout: post
title: HTTP Keep-Alive
category: blog
description: HTTP Keep-Alive
---

当一个客户端向服务器发送http请求时，两者之间会建立一个tcp连接，然后服务器发回响应信息同时关闭连接。如果请求的的页面中含有别的资源连接，比如图片、flsah等，就会再次创建连接。Keep-Alive的作用就是在第一次创建连接时，服务器会把这个tcp连接保持一段时间（服务器端会有一个keepaliveTime的最大时间，超过时间就断开连接）。这样就不会频繁的去建立tcp连接，同一次请求中的信息传递都可以使用同一个tcp连接。

http 1.0中默认是关闭的，需要在http头加入"Connection: Keep-Alive"，才能启用Keep-Alive；http 1.1中默认启用Keep-Alive，如果加入"Connection: close "，才关闭。

目前大部分浏览器都是用http1.1协议，也就是说默认都会发起Keep-Alive的连接请求了，所以是否能完成一个完整的Keep- Alive连接就看服务器设置情况。现在的一些服务器都可以设置KeepAlive是否开启，以及KeepAlive的超时时间，服务器支持的KeepAlive数量（数量一般不会很大，否则会对服务器产生很大的压力）。

由于http协议基于tcp协议，当然http协议也需要tcp的3次握手的过程，而对于一个完整的HTTP/1.0的请求和响应它有以下的一个请求过程

	建立tcp连接 (syn; ack, syn2; ack2; 三个分组握手完成)
	请求
	响应
	关闭连接 (fin; ack; fin2; ack2 四个分组关闭连接)
	
而对于一个完整的HTTP/1.1的请求和响应:

	建立tcp连接 (syn; ack, syn2; ack2; 三个分组握手完成)
	请求
	响应
	…
	…
	请求
	响应
	关闭连接 (fin; ack; fin2; ack2 四个分组关闭连接)
	
### HTTP长连接
[HTTP长连接说明](https://www.qcloud.com/document/product/214/4149)