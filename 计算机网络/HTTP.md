& 超文本传输协议（英语：HyperText Transfer Protocol，缩写：HTTP）

  HTTP是一个客户端（用户）和服务端（网站）之间请求和应答的标准，通常使用[TCP协议](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)

    [https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)

# HTTP的特点

  简单快速 灵活   无连接 、无状态

    [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview#http_%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%80%A7%E8%B4%A8](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview#http_%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%80%A7%E8%B4%A8)

# HTTP请求报文

 <[https://www.cnblogs.com/biyeymyhjob/archive/2012/07/28/2612910.html](https://www.cnblogs.com/biyeymyhjob/archive/2012/07/28/2612910.html)>

`＜request-line＞`

`＜headers＞`

`＜blank line＞`

`＜request-body＞`

  `请求行`

  `请求行由请求方法字段( 有8种 )、URL字段和HTTP协议版本字段3个字段组成，它们用空格分隔。`

  `例如，GET /index.html HTTP/1.1。`

`GET`

`？之后为请求数据，之间用&合开`

`不适合传送私密数据`

`浏览器对地址的字符有限制，一般为1024个字符(http没有规定URL长度，但部分浏览器有)`

`POST`

`POST方式请求行中不包含数据字符串，这些数据保存在”请求内容”部分，各数据之间也是使用”&”符号隔开`

可以传送私密数据？？？（相比较于get而言，本质上都不适合，要使用HTTPS）

不在URL中，理论提交的数据量大（部分服务器会对post提交的大小做限制）

POST请求相比GET请求要占用更多资源。从性能方面说，发送相同数量的数据，GET请求比POST请求要快两倍。

区别：

get请求会被浏览器主动缓存，而post不会

get请求的参数，会报保留在浏览器的历史记录里，而post不会

get重新执行请求没有副作用，而post请求重新执行会有副作用

[https://www.zhihu.com/question/28586791](https://www.zhihu.com/question/28586791)

put：更新资源

DELETE：删除资源

`HEAD`

        `和GET类似，没有相应内容，高效（查看某个页面的状态）`

`请求头`

  `请求头部由关键字/值对组成，每行一对，关键字和值用英文冒号“:”分隔。请求头部通知服务器有关于客户端请求的信息，典型的请求头有：`

`User-Agent：产生请求的浏览器类型。`

`Accept：客户端可识别的内容类型列表。`

`Host：请求的主机名，允许多个域名同处一个IP地址，即虚拟主机。??`

`更多参考`

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers

[https://weread.qq.com/web/reader/751326d0720befab7514782ka4a32da02aba4a042cf4e81](https://weread.qq.com/web/reader/751326d0720befab7514782ka4a32da02aba4a042cf4e81)

`空行`

  `最后一个请求头之后是一个空行，发送回车符和换行符，通知服务器以下不再有请求头。`

`请求数据`

  `请求数据不在GET方法中使用，而是在POST方法中使用。POST方法适用于需要客户填写表单的场合。与请求数据相关的最常使用的请求头是Content-Type和Content-Length。`

# HTTP相应报文

`＜status-line＞`

`＜headers＞`

`＜blank line＞`

`[＜response-body＞]`

`与请求的区别在第一行，用状态信息代替了请求信息。`

`状态行（status line）通过提供一个状态码来说明所请求的资源情况`

格式：HTTP-Version（http协议版本号） Status-Code（服务器发回的响应状态代码） Reason-Phrase CRLF（状态代码文本描述）

1xx：指示信息--表示请求已接收，继续处理。

2xx：成功--表示请求已被成功接收、理解、接受。

200 OK：客户端请求成功。

3xx：重定向--要完成请求必须进行更进一步的操作。

304：则表示资源未修改过，是从浏览器缓存中直接拿取的。

4xx：客户端错误--请求有语法错误或请求无法实现。

400 Bad Request：客户端请求有语法错误，不能被服务器所理解。

403 Forbidden：服务器收到请求，但是拒绝提供服务。

404 Not Found：请求资源不存在，举个例子：输入了错误的URL。

5xx：服务器端错误--服务器未能实现合法的请求。

500 Internal Server Error：服务器发生不可预期的错误。

503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复正常，举个例子：HTTP/1.1 200 OK（CRLF）。

`Eg:`

`HTTP/1.1 200 OK`

`Date: Sat, 31 Dec 2005 23:59:59 GMT`

`Content-Type: text/html;charset=ISO-8859-1`

`Content-Length: 122`

`＜html＞`

`＜head＞`

`＜title＞Wrox Homepage＜/title＞`

`＜/head＞`

`＜body＞`

`＜!-- body goes here --＞`

`＜/body＞`

`＜/html＞`