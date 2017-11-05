---
layout: post
title: "HTTPS With SSL Pinning"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### 为什么使用 HTTPS ？

随着网络逐渐融入到我们的日常生活，越来越多的人开始关注个人信息的安全性。由于现有的网络是基于 HTTP 协议进行通信的，因此 HTTP 协议的安全与否直接决定了我们个人信息的安全性。

HTTP 协议在通信过程中的三个缺点：

* 信息采用明文传输，无法保证隐私性
* 信息易被篡改，无法保证其完整性
* 无法验证通信双方的身份

为了解决 HTTP 自身的不足，推出了 HTTPS

> HTTPS = HTTP + 加密 + 认证 + 完整性保护
 
#### 加密算法

加密算法分为对称加密算法和非对称加密算法。对称加密只有一个密钥，同时可用来加密和解密使用；非对称加密则有两个密钥：公钥和私钥，当公钥用于加密，私钥用于解密时，可以确保与通信方的安全；当私钥用于加密，公钥用于解密时，可以确保消息来自可信任的指定通信方 [Public Key Cryptography](https://www.youtube.com/watch?v=GSIDS_lvRv4)。

为了保证在通信前，双方能够安全的获取密钥，HTTPS 首先采用非对称加密算法，服务端发送其公钥给客户端，自己保留私钥，客户端使用服务端的公钥加密信息，并将信息发送给服务端，服务端接收到信息后使用私钥进行解密，从而保证了信息在网络中传输的安全性。但非对称加密相比于对称加密会消耗更多的时间，占用更多地资源，因此为了提高数据传输的效率，只在连接建立时时采用非对称加密算法，数据发送时采用对称加密算法，即客户端获取服务端发来的公钥之后，使用公钥加密一个随机密钥，并发送给服务端，接着双方使用该随机密钥进行通信。

#### 认证

认证过程主要通过可信任的第三方结构颁发的证书来确认身份，如 CA 颁发证书给指定的服务器，客户端(如，浏览器或手机)中提前内置了可信任的 CA 颁发的公钥，当客户端访问支持 HTTPS 的服务端时，会首先根据公钥验证服务端证书的合法性，如果验证通过，则证明该服务器是合法的服务器。(2011.7 DigiNotar 曾被黑客入侵，并发布了伪造的可信任证书)

#### 信息完整性

通过对消息摘要进行哈希计算，将得到的哈希值存储到消息体中发送给服务器，服务器接收到消息后，对消息摘要再次进行哈希运算，将计算结果与消息体中的哈希值对比，检验消息是否在传输过程中被篡改。

### 为什么使用 SSL Pinning ？

虽然 HTTPS 采用了公钥和证书的加密和验证方式，但依然存在 MIM(中间人攻击)的可能性，因为证书颁发机构可能被黑客入侵，而 HTTPS 只验证了证书的合法性，**并没有验证当前服务器是否是我们要访问的服务器**。

为了保证我们当前访问的服务器就是我们需要访问的服务器，需要通过 SSL Pinning 的方式来实现，其包括 Certificate Pinning 和 Public Key Pinning 两种。

证书锁定需要把服务器的证书提前下载并内置到客户端中，当请求发起时，通过比对证书内容来确定连接的合法性，但由于证书存在过期时间，因此当服务器端证书更换时，需同时更换客户端证书。

公钥锁定则需提取证书中的公钥内置到客户端中，通过比对公钥值来验证连接的合法性，由于证书更换依然可以保证公钥一致，所以公钥锁定不存在客户端频繁更换证书的问题。

### 总结

* 加密保证通信过程是安全的，但无法确认通信方是否可信任
* 证书验证了通信方可信任，当无法确认是指定的通信方
* Pinning 保证通信方即是希望通信的通信方


### 应用实践

以 Xamarin 结合 HttpClient，验证 www.baidu.com 为例:

首先在终端中输入命令获取 www.baidu.com 的公钥

```
openssl s_client -connect www.baidu.com:443 | openssl x509 -pubkey -noout
```
 
得到

> -----BEGIN PUBLIC KEY-----  
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
MIIBCgKCAQEAyJkPC0Lev6L0shNY3OTO  
......  
-----END PUBLIC KEY-----

拷贝 Begin/End 之间的字符串并保存到 `PublicKey` 变量中，**MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A** 除外，由于该串字符表示加密算法(**1.2.840.113549.1.1.1**)可通过 `GetKeyAlgorithm()`可获取，而 `GetPublicKey()` 获取除算法标识外的公钥。

接着在 `AppDelegate` 的 `FinishedLaunching` 方法中设置如下

```csharp
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
ServicePointManager.ServerCertificateValidationCallback = (sender, certificate, chain, sslPolicyErrors) =>
{
    byte[] keyBytes = certificate.GetPublicKey();
    string remotePublicKey = Convert.ToBase64String(keyBytes);

    return PublicKey == remotePublicKey;
};
```


[*Pinning Cheat Sheet*](https://www.owasp.org/index.php/Pinning_Cheat_Sheet)  
[*SSL Certificate Pinning in mobile applications*](https://www.bugsee.com/blog/ssl-certificate-pinning-in-mobile-applications/)  
[*Certificate and Public Key Pinning with Xamarin*](https://thomasbandt.com/certificate-and-public-key-pinning-with-xamarin)   
[*Certificate Pinning*](https://talk.objc.io/episodes/S01E57-certificate-pinning)  
[*Increasing your trust certificate pinning on iOS*](https://fastchicken.co.nz/2016/03/21/increasing-your-trust-certificate-pinning-on-ios/)  
[*RSA Key Formats*](https://www.cryptosys.net/pki/rsakeyformats.html)  
[*Transform Public Key Format*](https://stackoverflow.com/questions/18039401/how-can-i-transform-between-the-two-styles-of-public-key-format-one-begin-rsa)  
[*Creating Certificates for TLS Testing*](https://developer.apple.com/library/content/technotes/tn2326/_index.html#//apple_ref/doc/uid/DTS40014136)  
[*openssl*](https://www.madboa.com/geek/openssl/)
