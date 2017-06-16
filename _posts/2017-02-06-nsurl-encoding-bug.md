---
layout: post
title: "记一次 NSUrl 中文字符导致的 bug"
description: ""
category:
tags: [Xamarin.iOS]
---
{% include JB/setup %}

### NSURL 字符格式异常

App 中有一个提交页面，当点击提交按钮操作后导致崩溃。

经测试，出现条件如下：

1. 只有特定账户才会复现
2. 相同账号和环境，只有在 adhoc 和 appstore 模式下才会复现

由条件1可推断大致由于账户不同导致数据不同，最终导致部分数据空异常；由条件2可推断跟数据无关，与条件1互斥。这就尴尬了-_-。因此只能通过反复注释代码和重新打包进行测试，却依然没能找出原因。
最后，通过对提交前后的操作进行梳理和排查，发现在提交后服务端立即向该设备推送了一条消息，而且是只有在发布模式才推送，因此，暂且断定消息推送才是罪魁祸首，立即注释后端推送代码，果然不再崩溃。但为什么有些账号不会崩溃呢？经过进一步查看手机中的崩溃日志，发现导致崩溃的原因是构造 `NSUrl` 失败，原来推送中带有额外的参数，而部分账户中参数包含中文字符，在接收到推送后，默认会提取附带参数并用来构造 `NSUrl` ，由于 `NSUrl` 接收到不符合规范的字符而抛出异常导致 app 崩溃。

> The URL string with which to initialize the NSURL object. This URL string must conform to URL format as described in RFC 2396, and must not be nil. This method parses URLString according to RFCs 1738 and 1808. [initWithString:](https://developer.apple.com/documentation/foundation/nsurl/1413146-initwithstring?language=objc)

经过测试发现，在 `Xamarin.iOS` 中，`new NSUrl(...)` 包含中文字符时默认会抛出异常；而 `NSUrl.FromString(...)` 包含中文字符时，默认会返回空，并不抛出异常。`Swift` 中 `URL` 构造失败默认返回 `nil`。

解决方式，构造 `NSString` 实例，通过实例方法 `CreateStringByAddingPercentEscapes (NSStringEncoding.UTF8)` 转义中文字符为 `UTF8` 编码，构造 `NSUrl`，恢复中文字符使用实例方法 `CreateStringByReplacingPercentEscapes (NSStringEncoding.UTF8)`。

### 头部参数字符格式异常

由于无法查看 `HttpClient` 源码实现，我们可以参考 `WebClient` 实现，打开 [WebHeaderCollection](http://referencesource.microsoft.com/#System/net/System/Net/WebHeaderCollection.cs,69a6a97a52b7b09a) 定义，会看到在添加头部参数时会调用如下方法：

```csharp
public override void Add(string name, string value)
```

方法内部分别对 `name` 和 `value` 字符进行检测，实现逻辑如下：

```csharp
//
// CheckBadChars - throws on invalid chars to be not found in header name/value
//
internal static string CheckBadChars(string name, bool isHeaderValue) {
    // ...
    if (isHeaderValue) 
    {
        // VALUE check
        //Trim spaces from both ends
        name = name.Trim(HttpTrimCharacters);
     
        //First, check for correctly formed multi-line value
        //Second, check for absenece of CTL characters
        int crlf = 0;
        for(int i = 0; i < name.Length; ++i) {
            char c = (char) (0x000000ff & (uint) name[i]);
            switch (crlf)
            {
                case 0:
                    if (c == '\r')
                    {
                        crlf = 1;
                    }
                    else if (c == '\n')
                    {
                        // Technically this is bad HTTP.  But it would be a breaking change to throw here.
                        // Is there an exploit?
                        crlf = 2;
                    }
                    else if (c == 127 || (c < ' ' && c != '\t'))
                    {
                        throw new ArgumentException(SR.GetString(SR.net_WebHeaderInvalidControlChars), "value");
                    }
                    break;
     
                case 1:
                    if (c == '\n')
                    {
                        crlf = 2;
                        break;
                    }
                    throw new ArgumentException(SR.GetString(SR.net_WebHeaderInvalidCRLFChars), "value");
     
                case 2:
                    if (c == ' ' || c == '\t')
                    {
                        crlf = 0;
                        break;
                    }
                    throw new ArgumentException(SR.GetString(SR.net_WebHeaderInvalidCRLFChars), "value");
            }
        }
        if (crlf != 0)
        {
            throw new ArgumentException(SR.GetString(SR.net_WebHeaderInvalidCRLFChars), "value");
        }
    }
    else
    {
        // NAME check
        //First, check for absence of separators and spaces
        if (name.IndexOfAny(ValidationHelper.InvalidParamChars) != -1) 
        {
            throw new ArgumentException(SR.GetString(SR.net_WebHeaderInvalidHeaderChars), "name");
        }
     
        //Second, check for non CTL ASCII-7 characters (32-126)
        if (ContainsNonAsciiChars(name)) {
            throw new ArgumentException(SR.GetString(SR.net_WebHeaderInvalidNonAsciiChars), "name");
        }
    }
    // ...
}
```

上面代码主要对头参数中的 name 和 value 进行验证。

当参数是 name 时，即 isHeaderValue = false，验证 name 中是否包含如下不合法的字符。

> ( ) < > @ , ; : \\ " \ / [ ] ? = { }  \t \r \n

以及是否包含非 ASCII 字符。 

当参数是 value 是，即 isHeaderValue = true，验证 value 中是否包含不合法的控制字符。规则是，\r 后的字符必须是 \n，\n 后的字符必须是空格或 \t，否则则抛出异常；ASCII 值为 127 或小于空格且不等于 \t 时抛出异常。合法的字符如下。
> 9(TAB) 制表符 + 空格以及空格之后的字符 - 127(DEL) 删除符 

### .NET 编/解码 API

#### UrlEncode

在 .NET 中我们可以使用 `HttpUtility` 类进行编解码，该类定义在 `System.Web` 命名空间下，如 [HttpUtility.UrlEncode](https://msdn.microsoft.com/en-us/library/4fkewx0t(v=vs.110).aspx) 方法用于编码操作，默认采用 UTF-8。

`UrlEncode` 默认会采用 UTF-8 编码，通过 `Encoding.UTF8.GetBytes(string)` 获取 UTF-8 格式的字节数组，然后将字节数组中[不安全的字符转化为 %XX 格式](http://referencesource.microsoft.com/#System.Web/Util/HttpEncoder.cs,aa1079312bd28b8c)，最后使用 `Encoding.ASCII.GetString(byte[])` 将字节数组转化为最终的字符串格式。

> string > UTF8 [byte] > 替换 [byte] > ASCII string


#### Base64

[Base64](https://en.wikipedia.org/wiki/Base64#Examples) 算法的主要实现是通过将字符串内容转化为二进制的字节数组，接着通过按 6 位为一组进行划分，根据 Base64 索引表，将 6 位表示的值转化为对应的字符，最终得到转化后的字符串。在 .NET 中可以使用 `Convert.ToBase64String(byte[])` 和 `Convert.FromBase64String(string)` 来操作。



[*RFC 3986*](https://tools.ietf.org/html/rfc3986)  
[*NSURL*](https://developer.apple.com/reference/foundation/nsurl)