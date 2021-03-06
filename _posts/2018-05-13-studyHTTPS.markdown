---
layout:     post
title:      "HTTPS"
date:       2018-5-13
author:     "leeyixuan"
header-img: "img/background/post-bg-flex.jpg"
tags:
    - TCP/IP协议
---

HTTPS是在HTTP基础上多套了一层SSL安全套接层。
SSL位于HTTP和TCP之间。
## HTTPS的先进性
### HTTP的安全隐患：
1. 响应主体内容为明文传输。容易被抓包盗取。
2. 无法确定响应内容的完整性正确性（可能会有他人恶意篡改的风险）。
3. 中间人攻击，冒充服务器发送假的公钥和用户进行通信，盗取用户信息。


### HTTPS从以下三个方面弥补了HTTP的安全性：

#### 1. 为响应主体内容提供加密。
加密方式分为公开密钥加密和共享密钥加密。
- 公开密钥机制有公钥和私钥，公钥公开给用户保存，私钥保密給服务器保存。公钥加密私钥解密，私钥加密公钥解密。
- 共享密钥机制由用户和服务器共同维护一个密钥，该密钥既能加密也能解密。

由于公开密钥机制的加解密效率慢，一般使用公开密钥机制在通信的一开始建立连接，交换共享密钥，在通信的阶段还是使用公享密钥进行加密。

#### 2. **数字签名digital signature**保证响应主体的完整安全。

![](https://www.github.com/CoolRabbit520/photos/raw/master/小书匠/1532057336912.png)

#### 3. **数字证书digital certificate**保证获得真正的服务器公钥，杜绝的中间人攻击。
用户拥有第三方机构的公钥，
由第三方机构发布数字证书（数字证书是第三方的私钥对服务器的公钥进行加密的结果）
用户收到数字证书之后，使用第三方机构的公钥解密，获得服务器的公钥。



## SSL协议的握手过程
HTTPS建立安全通信之前，有五次安全握手的流程：

第一步，浏览器给出协议版本号、一个随机数（Client random），以及加密方法，比如RSA加密，此时是明文传输。

第二步，服务器确认双方使用的加密方法，并给出数字证书、以及一个随机数（Server random）。

第三步，浏览器确认数字证书有效，并取出数字证书中的公钥，然后生成一个新的随机数（Premaster secret），使用公钥以及指定的加密方加密这个随机数，发给服务器。利用Client random、Server random和Premaster secret通过一定的算法生成HTTP链接数据传输的对称加密key-`session key`。

第四步，服务器使用自己的私钥和已知的加密方法，获取浏览器发来的随机数（即Premaster secret）。和浏览器相同规则生成`session key`

第五步，浏览器和服务器利用Client random、Server random和Premaster secret通过一定的算法生成HTTP链接数据传输的对称加密key-`session key`，用来加密接下来的整个对话过程。


**注意**：
1. HTTPS5次安全握手的每一步都是应用层的通信，底层都有3次TCP3次握手4次挥手的过程。
2. 这5次握手只是通过公开密钥加密方式安全地产生一个共享密钥。HTTPS只是在安全通信建立之初使用公开密钥的加密方式，之后使用共享密钥的加密方式进行通信。因为公开密钥的过程繁杂效率低。



## Reference
1. [数字签名是什么](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)
2. [图解SSL/TLS协议](http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html)
3. 《图解HTTP》