---
layout: post
title: "学习使用 HttpClient"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### HttpClient

`HttpClient`定义了`HTTP`请求和响应相关的类型实现，简化了网络请求的编码方式，完全支持异步调用（`async/await`），通过使用`HttpClient`及相关的一系列类型定义，可以很方便地构造请求和接收响应消息。

### GET | POST | PUT | DELETE

我们知道根据不同的请求动词，可以指定不同的请求操作。下面来看一下`HttpClient`中以上这四个请求动词是如何使用的。

#### GET

比如一个简单的获取这篇文章内容的`Http`请求可以这样写：

```csharp
public async Task GetStringAsync()
{
    using(var httpClient = new HttpClient())
    {
        string content = await httpClient.GetStringAsync("http://kanlei.github.io/2017/12/17/learn-to-use-httpclient");
        Console.WriteLine(content);
    }
}
```

由于`HttpClient`实现了`IDisposable`接口，所以可以使用`using`语句来释放不再使用的底层`Socket`连接。

> 通常，在项目中我们只会定义一个`HttpClient`实例，所有的请求都复用这个实例，这样做不仅避免了在每次请求时都指定请求头、`Cookie`、缓存等其它配置项，还复用了底层的`Socket`连接，避免不必要的重复创建和释放的过程，而且`HttpClient`是保证线程安全的，也不存在多线程竞争的问题；否则甚至可能会导致[`SocketException`](https://docs.microsoft.com/en-us/aspnet/web-api/overview/advanced/calling-a-web-api-from-a-net-client#create-and-initialize-httpclient)。

除了`GetStringAsync`方法，类库还提供了`GetStreamAsync`获取流，`GetByteArrayAsync`获取字节数组，以及`GetAsync`获取完整的响应消息，该方法还提供了`HttpCompletionOption`以及`CancellationToken`重载，用于指定响应的条件，比如在响应头可用时即识别出是否支持当前响应的处理，以及支持取消操作。

向下面这样，如果响应成功则读取消息内容：

```csharp
public async Task GetStringAsync()
{
    HttpResponseMessage responseMessage = await httpClient.GetAsync("http://kanlei.github.io/2017/12/17/learn-to-use-httpclient");
    if (responseMessage.IsSuccessStatusCode)  // [200-299]
    {
        string content = await responseMessage.Content.ReadAsStringAsync();
    }
}
```

#### POST

`HttpClient`提供了多种`POST`请求格式，比如使用`FormUrlEncodedContent`提交表单，通过对提交的内容进行`URL`编码后添加到请求体内，提交给服务器。

```csharp
// application/x-www-form-urlencoded
public async Task PostFormAsync()
{
    var keyValues = new List<KeyValuePair<string, string>>()
    {
        new KeyValuePair<string, string>("name", "value")
    };
    var formContent = new FormUrlEncodedContent(keyValues);
    HttpResponseMessage responseMessage = await httpClient.PostAsync("uri", formContent);
}
```

`PostAsync`方法接收一个`HttpContent`类型的参数，该类型是一个实现了`IDisposable`的抽象类。相关子类定义如下：

```
HttpContent -> ByteArrayContent -> FormUrlEncodedContent
                                -> StringContent
HttpContent -> StreamContent
HttpContent -> MultipartContent -> MultipartFormDataContent
```

使用StringContent提交`JSON`文本到服务器：

```csharp
// application/json
public async Task PostStringAsync()
{
    var stringContent = new StringContent("{ \"title\"=\"post\", \"content\"=\"string\" }", Encoding.UTF8, "application/json");
    HttpResponseMessage responseMessage = await httpClient.PostAsync("uri", stringContent);
}
```

使用`MultipartFormDataContent`上传文件：

```csharp
// multipart/form-data
public async Task PostFileAsync(Stream stream)
{
    stream.Seek(0, SeekOrigin.Begin);
    var streamContent = new StreamContent(stream);
    var formDataContent = new MultipartFormDataContent();
    formDataContent.Add(streamContent, "upload", "file.png");
    HttpResponseMessage responseMessage = await httpClient.PostAsync("uri", streamContent);
}
```

注意，这里是通过读取流的方式来上传文件，所以首先要确保**流的读取位置为起始位置**，其次流使用完毕后需要手动释放，该操作可以在请求响应后的上层逻辑中释放。

`HttpClient`大大简化了代码量，比如完整的上传代码实现需要添加像这样的参数：

```csharp
private async Task PostFileByStream(Stream stream)
{
    stream.Seek(0, SeekOrigin.Begin);
    var streamContent = new StreamContent(stream);
    streamContent.Headers.ContentLength = stream.Length;
    streamContent.Headers.ContentDisposition = ContentDispositionHeaderValue.Parse("form-data");
    streamContent.Headers.ContentDisposition.Parameters.Add(new NameValueHeaderValue("name", "upload"));
    streamContent.Headers.ContentDisposition.Parameters.Add(new NameValueHeaderValue("filename", "file.png"));
    streamContent.Headers.ContentType = MediaTypeHeaderValue.Parse("image/png");

    var multipartForm = new MultipartFormDataContent("--WebKit" + DateTime.Now.Ticks);
    multipartForm.Add(streamContent);
    HttpResponseMessage result = await httpClient.PostAsync("uri", multipartForm);
}
```

以上仍然是简化后的版本，如果完全手动拼接格式，则会向下面这样（`iOS`的实现）：

```csharp
var sessionDelegate = new SessionDelegate (progressCallback);
var defaultSession = NSUrlSession.FromConfiguration (NSUrlSessionConfiguration.DefaultSessionConfiguration, sessionDelegate, new NSOperationQueue ());

var header = $"Content-Disposition: form-data; name=\"file\"; filename=\"{fileName}\"\r\nContent-Type: {mineType}\r\n\r\n";
var boundary = "--WebKitFormBoundary" + DateTime.Now.Ticks;
var urlRequest = new NSMutableUrlRequest (NSUrl.FromString (url));
urlRequest.HttpMethod = "POST";
urlRequest ["Content-Type"] = "multipart/form-data; boundary=" + boundary;

stream.Seek(0, SeekOrigin.Begin);
NSData imageData = NSData.FromStream (stream);
NSMutableData mutableData = new NSMutableData ();
mutableData.AppendData (NSData.FromString ($"\r\n--{boundary}\r\n"));
mutableData.AppendData (NSData.FromString (header));
mutableData.AppendData (imageData);
mutableData.AppendData (NSData.FromString ($"\r\n--{boundary}\r\n"));

urlRequest ["Content-Length"] = mutableData.Length.ToString ();

NSUrlSessionDataTaskRequest dataTaskRequest = await defaultSession.CreateUploadTaskAsync (urlRequest, mutableData);
defaultSession.FinishTasksAndInvalidate ();
```

我们手动指定了请求头的`Content-Type`为`multipart/form-data`以及分割内容边界的`boundary`，该`boundary`不能在内容中出现；指定内容头`Content-Disposition`、`name`、`filename`和`Content-Type`，并谨慎的在合适的位置上插入换行符和边界标识符以及空行。以上的代码最终会转化为类似如下的请求格式发送给服务器：

```
POST image/upload HTTP/1.1
HOST:
Content-Type: multipart/form-data; boundary="--WebKitFormBoundary636491486584734080"
Content-Length: 1124

--WebKitFormBoundary636491486584734080
Content-Disposition: form-data; name="file"; filename="file.png"
Content-Type: image/png"
Content-Length: 1024

10101000101010100010101001...  // image data

--WebKitFormBoundary636491486584734080
```

以上遵循了 [rfc2388](https://www.ietf.org/rfc/rfc2388.txt) 的定义，如果需要上传多个内容，则需要更多的`boundary`用于标识边界，由此可见，`HttpClient`极大地提高了我们的编码效率，减少了手动拼接字符格式的错误次数，提升了开发效率。

#### PUT 与 DELETE

`PUT`与`POST`的使用方式相似；而`DELETE`则与`GET`的使用方式相似。

```csharp
public async Task PutAndDeleteAsync()
{
    HttpResponseMessage putResponse = await httpClient.PutAsync("uri", new StringContent(""));

    HttpResponseMessage deleteResponse = await httpClient.DeleteAsync("uri");
}
```

#### Send

其实上面所有便捷的请求方式，最终都会转化为调用`SendAsync`，该方法提供了一个`HttpRequestMessage`类型的参数重载，该类型可以指定所有与请求相关的内容，包括请求类型与请求体，如：

```csharp
public async Task SendAsync()
{
    var requestMessage = new HttpRequestMessage(HttpMethod.Get, "uri");
    requestMessage.Content = new StringContent("hello world");
    HttpResponseMessage deleteResponse = await httpClient.SendAsync(requestMessage);
}
```

### 请求处理 HttpMessageHandler

在构造`HttpClient`实例时，可以选择传递两个参数，一个是`HttpMessageHandler`类型的`handler`，还有一个是`bool`类型的`disposeHandler`。

```csharp
var client = new HttpClient(new HttpClientHandler(), false);
```

#### disposeHandler

`false`则表示在`HttpClient`上调用`Dispose`方法并不会释放当前`handler`，以及其所关联的`Socket`连接；`true`则相反。当我们想复用同一个`HttpMessageHandler`给多个`HttpClient`实例使用时，可以通过该参数控制释放的行为方式。

#### handler

其实`HttpClient`并不处理请求，而是将请求交付给`handler`来处理，`HttpMessageHandler`定义如下：

```csharp
public abstract class HttpMessageHandler : IDisposable
{
    protected HttpMessageHandler ();

    public void Dispose ();

    protected virtual void Dispose (bool disposing);

    protected internal abstract Task<HttpResponseMessage> SendAsync (HttpRequestMessage request, CancellationToken cancellationToken);
}
```

其`SendAsync`方法定义则说明了该类型的实际用途，`HttpClient`的`SendAsync`实际是调用了`handler`中的`SendAsync`方法定义。其相关的子类型定义如下：

```
HttpMessageHandler -> DelegatingHandler -> MessageProcessingHandler
HttpMessageHandler -> HttpClientHandler
HttpMessageHandler -> SocketsHttpHandler
```

`HttpClient`默认`handler`构造是`HttpClientHandler`，其定义了如 Cookie、Credentials、Proxy 等功能。`DelegatingHandler`则提供了一个`InnerHandler`属性，使得我们可以串联现有的`handler`，每个`handler`仅负责处理请求或响应过程中的一个阶段。`MessageProcessingHandler`则提供了更方便的`ProcessRequest`和`ProcessResponse`方法用于子类实现。

至此，`HttpClient`简单用法和构造已介绍完，[这里](https://blogs.msdn.microsoft.com/webdev/2012/08/26/list-of-asp-net-web-api-and-httpclient-samples/)可以查看更多的真实案例。

[*Call a Web API From a .NET Client*](https://docs.microsoft.com/en-us/aspnet/web-api/overview/advanced/calling-a-web-api-from-a-net-client#create-and-initialize-httpclient) / [*httpclientwrong*](https://aspnetmonsters.com/2016/08/2016-08-27-httpclientwrong/) / [*HttpClient*](http://chimera.labs.oreilly.com/books/1234000001708/ch14.html#_wrapper)