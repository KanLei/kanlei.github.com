---
layout: post
title: "双因素认证的原理与实践"
description: ""
category: 
tags: []
---
{% include JB/setup %}


### 多因素认证

多因素认证是常用于身份认证的一种方式，通过验证多个因素来确保用户身份的合法性。

认证的因素通常分为以下 3 类：

- 只有你自己知道的知识
- 只有你自己拥有的物品
- 只有你自己拥有的特征

以门禁为例，如果你需要输入密码，那么密码就是你所知道的知识；如果你需要使用卡片，那么卡片就是你所拥有的物品；如果你需要验证指纹，那么指纹就是你的特征。由于门禁的安全性要求并不是很高，所以通常只采用其中一种因素来认证用户的身份。

单因素认证存在一些弊端，比如你的密码被盗了。目前银行采用的是双因素认证，即知识+物品，除了你知道你的密码，你还必须持有你的卡片，只有两者同时满足时，才能证明你是这个账户的主人；即便两者中的任意一项被盗取，依然可以保障你账户的安全。由此可见，需要认证的因素越多，安全性则越高，操作的复杂度也随之增加。

### 用户身份认证

随着网络越来越普及，网络账户安全也逐渐受到用户的重视，曾经使用单个密码的安全认证方式也逐渐被完善和改进。根据上面提到的三种常用的认证因素，可以很容易想到，除了知识（密码）因素，我们还可以添加另外两种因素，但出于对第三种因素（特征）验证，不仅需要设备支持，同时也要避免被服务商滥用的考虑，物品因素便成为了改进认证机制的最佳选择。在当下，能时时伴随在我们身边的物品中，相信手机绝对是其中最重要的一件，因此手机则转变成了最佳认证工具。

#### 如何让手机完成认证？

以最简单的短信验证为例，每当用户登录账户时，除了输入密码，还要额外输入一个短信验证码，以确保该账户信息安全。但随着技术的不断更新迭代，短信验证码以及 SIM 卡盗窃和伪造的技术也逐渐被不法分子掌握和利用，因此新的安全认证方案策略亟待解决。

由上面的验证流程可知，短信验证因素满足以下 2 点：

- 手机由用户本人持有
- 验证码只有用户和服务商知道


#### 可行性方案假设

##### 验证码传送安全性

假如用户和服务商各自提前拥有一个验证码，而这个验证码与用户账号进行绑定，那是不是就可以避免验证码因通过短信发送，或网络传送而造成的安全隐患。

##### 验证码可用时效性

验证码需要在一定时间段内可用，超过时间段后需要重新申请验证码，以避免类似重播攻击等带来的安全隐患，因此用户和服务商所持有的验证码也必须是具有时效性的。而这种时效性可以通过引入基于时间的度量策略来解决，用户和服务商达成一致的约定，即基于当前的[时间+验证码]计算得到最终的验证码。

##### 时间偏差

由于上面引入时间单位作为验证码的一部分，则不可避免的存在用户与服务商之间由于网络延迟导致时间偏差的现象，为了解决时间偏差，我们需要添加最大时间范围的容错机制，比如 30s 以内的偏差认为是合理的时间偏差。

满足以上三个条件的验证方式其实已经存在，并且已被业界定义为一项标准来实现，即基于时间的一次性密码(TOTP)。

#### TOTP

> Time-based One Time Password

TOTP 的主要思想是，用户注册账户后，服务商会根据当前账户提供一个唯一的 SecretKey，用户通过专用的软件记录下这个 SecretKey，软件将 SecretKey 与当前时间通过算法计算得出一个验证码，接着将验证码提交给服务商，完成账号与 SecretKey 的绑定。当用户再次登录时，服务商会同时校验密码和 SecretKey，以验证用户的身份。

公式如下：

> Key = Hash(SecretKey, (Time / Tolerance))

代码实现

[*Wikipedia*](https://en.wikipedia.org/wiki/Multi-factor_authentication) / 
[*TOTP*](https://tools.ietf.org/html/rfc6238) / 
[*Otp.NET*](https://github.com/kspearrin/Otp.NET)
