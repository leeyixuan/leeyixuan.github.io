---
layout:     post
title:      "HTTP"
date:       2018-5-14
author:     "leeyixuan"
header-img: "img/background/post-bg-flex.jpg"
tags:
    - TCP/IP协议
---


## get方法 v.s. post方法
HTTP方法是为了告知服务器本次请求的目的，用于不同的场景。比如，get是为了获取资源，post是为了传输资源。不同方法的HTTP数据包并无差别，底层的实现并无差别，都是是TCP链接。虽然GET方法也能传输实体的主体，但是一般不使用GET方法。

但是，由于浏览器/服务器的限制，导致GET和POST在应用过程中体现出一些不同。
区别的根源在于：
**get会将请求的参数跟在URL后面进行传递；post将请求的参数添加到HTTP消息的实体内部发送。**


1. POST发送的数据量比GET更大   
实际上，HTTP协议规范没有对URL长度进行限制，URL不存在参数上限的问题。这个限制是特定的浏览器及服务器对它的限制。IE对URL长度的限制是2083字节(2K+35)。对于其他浏览器，如Netscape、FireFox等，理论上没有长度限制，其限制取决于操作系统的支持。    
理论上讲，HTTP协议规范对POST数据是没有限制的，起限制作用的是服务器的处理程序的处理能力。

2. POST的安全性要比GET的安全性高   
POST传输的数据不会作为url的一部分，不会被缓存、保存在服务器日志、以及浏览器浏览记录中。


3. POST比GET慢   
从性能的角度上统计，GET请求的速度最多可达到POST的两倍。简单的说：GET产生一个TCP数据包；POST产生两个TCP数据包。对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

4. GET请求能缓存，POST不能。     
POST相对GET安全一点点，因为GET请求都包含在URL里，且会被浏览器保存历史纪录POST不会，但是在抓包的情况下都是一样的。


## 重定向
 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1532350961102.png" />
重定向由服务器发送特殊的响应而触发。浏览器接收重定向响应之后会使用响应提供的URL重发一次请求。

重定向的目的：
- 正确处理请求（资源已经被分配到了新的URL上，网页扩展名改变）
- 最小化时延
- 节约网络带宽

不同类型的重定向映射可以划分为三个类别：永久重定向、临时重定向和特殊重定向。

1. 永久重定向
- 301(Moved Permanently)：GET方法不会发生变更，其他方法**有可能**会变更为GET方法。
- 308(Permanent Redirect)：方法和消息主体都**不发生变化**。

2. 临时重定向
- 302(Found	GET)：方法不会发生变更，其他方法**有可能**会变更为GET方法。
- 303(See Other)：GET方法不会发生变更，其他方法会变更为GET方法（消息主体会丢失）。用于PUT或POST请求完成之后进行页面跳转来防止由于页面刷新导致的操作的重复触发。
- 307(Temporary Redirect)：方法和消息主体都**不发生变化**。

3. 特殊重定向
- 300(Multiple Choice)：所有的选项在消息主体的HTML页面中列出。也可以返回 200 OK 状态码。
- 304(Not Modified)：该状态码表示缓存值依然有效，可以使用。


注意：
- HTTP响应除了返回3XX的状态码，还会返回一个`location`字段，告知浏览器随后向该URL发起请求。
- 除了307（临时重定向）和308（永久重定向），浏览器接随后会发送get方法的HTTP请求。

301和302的区别：
- 相同点：
301和302状态码都表示重定向，就是说浏览器在拿到服务器返回的这个状态码后会自动跳转到一个新的URL地址，这个地址可以从响应的Location首部中获取（用户看到的效果就是他输入的地址A瞬间变成了另一个地址B）。

- 不同点：
301是永久重定向，302是临时重定向。
301表示旧地址A的资源已经被永久地移除了（这个资源不可访问了），搜索引擎在抓取新内容的同时也将旧的网址交换为重定向之后的网址；
302表示旧地址A的资源还在（仍然可以访问），这个重定向只是临时地从旧地址A跳转到地址B，搜索引擎会抓取新的内容而保存旧的网址。 SEO302好于301。


除了HTTP重定向，还有其他重定向方法。
- HTML meta元素

```html
<head> 
  <meta http-equiv="refresh" content="0;URL=http://www.example.com/" />
</head>
```
- js  `window.location`

```javascript
window.location = "http://www.example.com/";

```
## 长连接
HTTP/1.0的时候，每进行一次HTTP通信就断开一次TCP连接。    
当浏览器浏览一个包含多张照片的HTML页面的场景下，每次TCP都会造成许多无谓的TCP连接建立和端口，增加通信量的开销。



 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1532360522078.png" />


HTTP/1.1提出**持久连接 keep-alive**的方法。   
持久连接的特点就是：只要任意一端没有明确提出断开连接，就一直保持TCP连接。在HTTP/1.1中是默认打开持久连接的。

 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1532360521318.png" />


持久连接对应的HTTP头部字段时`connection`。   
HTTP/1.1默认是持久连接，因此会自动填充`connection：keep-alive`。   
当服务器想明确断开连接时，则指定`connection：close`。   


**注意**：

1. 长连接只是在一次HTTP请求响应后，不断开TCP连接，继续下一个HTTP请求响应。并不是多个HTTP请求并行！！同一域名下的多个HTTP请求并行是HTTP/2.0的特性。
2. keep-alive不会永远保持，它有一个持续时间，一般在服务器中配置（如apache），另外长连接需要客户端和服务器都支持时才有效。


## HTTP/2.0
HTTP/2.0在与HTTP/1.1完全语义兼容的基础上，进一步减少了网络延迟，大幅度的提升了web性能。

HTTP/2.0具有以下特性：
### 1. 多路复用 (Multiplexing)：实现多流并行
在 HTTP/1.1协议中，浏览器客户端在同一时间，针对同一域名下的请求有一定数量限制。超过限制数目的请求会被阻塞。

 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1532358933354.png" />

多路复用允许同时通过单一的HTTP/2连接发起多重的请求-响应消息。HTTP/2.0通信都在一个连接上完成，这个连接可以承载任意数量的双向数据流。

因此，不需要合并HTTP请求，比如精灵图。合并HTTP请求之后，反而效果变差。

### 2. 二进制分帧：改进传输性能，实现低延迟和高吞吐量
应用层(HTTP/2.0)和传输层(TCP或UDP)之间增加一个二进制分帧层。

 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1532358933355.png" />

 在二进制分帧层上， HTTP/2.0会将所有传输的信息分割为更小的消息（包含头部和主体）。并进一步将消息拆解为帧（消息的头部拆为 header frame，消息的主体拆为data frame）。最后对它们采用二进制格式的编码。
 
 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1532358933017.png" />


客户端和服务器可以把HTTP消息分解为互不依赖的帧，然后乱序发送，最后再在另一端把它们重新组合起来。注意，同一链接上有多个不同方向的数据流在传输。客户端可以一边乱序发送stream，也可以一边接收者服务器的响应，而服务器那端同理。

 <img class="shadow" width="450" src="https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1532358933018.png" />


**总结**：
- 同一域名的请求只建立一个TCP连接
- 一个TCP连接可以承载多个并行的双向数据流stream(逻辑流)
- 每个数据流以消息的形式发送，而消息由一或多个帧组成，这些帧可以乱序发送，然后再根据每个帧首部的流标识符重新组装。


### 3. 首部压缩
HTTP/1.x的header带有大量信息，而且每次都要重复发送，HTTP/2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
### 4. 服务端推送：在客户端请求之前发送数据
在HTTP/2.0中，服务器可以对客户端的一个请求发送多个响应。除了对最初请求的响应外，服务器还可以额外向客户端推送资源，而无需客户端明确地请求。
### 5. 请求优先级
多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY允许给每个request设置优先级，这样重要的请求就会优先得到响应。比如浏览器加载首页，首页的html内容应该优先展示，之后才是各种静态资源文件，脚本文件等加载，这样可以保证用户能第一时间看到网页内容。


### HTTP/2有五大优势

1. 每个服务器只用一个连接。HTTP/2对每个服务器只使用一个连接，而不是每个文件一个连接。这样，就省掉了多次建立连接的时间，这个时间对TLS尤其明显，因为TLS连接费时间。
2. 加速TLS交付。HTTP/2只需一次耗时的TLS握手，并且通过一个连接上的多路利用实现最佳性能。HTTP/2还会压缩首部数据，省掉HTTP/1.x时代所需的一些优化工作，比如拼接文件，从而提高缓存利用率。
3. 简化Web应用。使用HTTP/2可以让Web开发者省很多事，因为不用再做那些针对HTTP/1.x的优化工作了。
4. 适合内容混杂的页面。HTTP/2特别适合混合了HTML、CSS、JavaScript、图片和有限多媒体的传统页面。浏览器可以优先安排那些重要的文件请求，让页面的关键部分先出现，快出现。
5. 更安全。通过减少TLS的性能损失，可以让更多应用使用TLS，从而让用户信息更安全。


## Reference
1. [浅谈HTTP中Get与Post的区别 ][1]    
2. [GET和POST有什么区别？及为什么网上多数答案都是错的 ][2]
3. [使用HTTP/2提升性能的7个建议](https://www.w3ctech.com/topic/1563#tip7sharding)
4. [HTTP2.0的奇妙日常](http://www.alloyteam.com/2015/03/http2-0-di-qi-miao-ri-chang/#prettyPhoto)
5. [HTTP 的重定向](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Redirections)
6. 《图解HTTP》

  [1]: https://www.cnblogs.com/hyddd/archive/2009/03/31/1426026.html
  [2]: http://web.jobbole.com/86298/