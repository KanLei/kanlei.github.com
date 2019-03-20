---
layout: post
title: "Recursive Length Prefix"
description: ""
category: blockchain
tags: [ethereum]
---
{% include JB/setup %}

### 什么是 RLP

RLP(Recursive Length Prefix) 主要用于 Ethereum 中对象序列化传输和存储，比如 Word State Trie 中的账号信息存储。RLP 旨在通过字节序列来简单高效地编码数据信息，至于字节序列中表示的具体数据类型则交由上层协议实现。

### 为什么选择 RLP

如果我们想通过一串字节序列来表示数据信息，需要满足以下条件：

1. 希望单个字节能尽可能表示出要获取数据的完整信息
2. 至少需要表示两种数据格式，第一种是字符串元素，第二种是由元素嵌套组成的列表，因为列表本身也是元素

有了以上2条标准，就可以考虑具体的实现方案了，单个byte能表示的数据范围是[0x00,0xff]，如果单个字符在 ASCII [0,127]范围之内，即[0x00,0x7f]，则可以将该字符用单个字节存储其本身；超出 ASCII 范围的元素需要用剩余范围[0x80,0xff]来表示，而数据又划分成两种类型：元素和列表，因此把[0x80,0xff]划分成两部分，[0x80,0xbf]用来表示元素，[0xc0,0xff]用来表示列表；如果元素字节长度超出ASCII范围，可以采用0x80+字节长度的方式来定位真实数据的存储位置和范围，如果字节长度过长，则可以进一步增加字节长度的长度前缀表示，即0x80+字节长度的长度+字节长度，来增加[0x80,0xbf]的表示范围；同理，列表亦是如此。

### RLP 规则定义

#### [0x00,0x7f]

表示ASCII字符集

#### [0x80]

表示空字符串

#### [0x81,0xbf]

##### [0x81,0xb7]

如果要表示的元素超出ASCII范围，且字节长度在[2,55]之间，则整个字节序列有两部分组成：0x80+元素字节长度和元素字节内容

##### [0xb8,0xbf]

如果要表示的元素超出ASCII范围，且字节长度大于55，则整个字节序列由三部分组成：0xb7+元素字节长度的长度+元素字节的长度+元素字节内容

#### [0xc0]

表示空列表

#### [0xc1,0xff]

##### [0xc1,0xf7]

如果要表示列表字节长度在[2,55]之间，则整个字节序列有两部分组成：0xc0+列表元素字节长度总和和列表中元素字节存储表示

##### [0xf8,0xff]

如果要表示的列表字节长度大于55，则整个字节序列由三部分组成：0xf7+列表字节长度总和的字节长度+列表字节长度总和+列表中各元素字节存储表示

#### Tips:

RLP 编码整形时需要采用大端序，并且移除前缀0


[*RLP-CSharp*](https://github.com/KanLei/RLP-CSharp)  
[*RLP Wiki*](https://github.com/ethereum/wiki/wiki/RLP)  
[*RLP EncodingDecoding*](https://www.bacoor-vietnam.co/single-post/2018/03/07/Data-structure-in-Ethereum-Episode-1-Recursive-Length-Prefix-RLP-EncodingDecoding)  
[*Ethereum Building Blocks RLP*](http://hidskes.com/blog/2014/04/02/ethereum-building-blocks-part-1-rlp/)