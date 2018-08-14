---
title: Web的工作方式
date: 2018-06-14 16:05:24
tags: [Web,工作方式]
---

* 对于普通的上网，系统是这样做的：浏览器本身就是一个客户端，当你输入URL的时候，首先浏览器会去请求DNS服务器，通过DNS获取相应域名的对应的Ip地址，通过IP地址找到对应IP对应的服务器，要求建立TCP连接，等浏览器发送完HTTP Request包后，服务器接受到请求包之后才开始处理请求包，服务器调用自身服务，返回Http Response （响应包）：客户端收到来自服务器的响应后开始渲染这个Response包里的主体（body）,等收到全部的内容随后断开与该服务器之间的TCP连接。

![用户访问一个Web站点的过程](https://upload-images.jianshu.io/upload_images/5363507-8f5601da0c982aed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
* Web服务器也被称为HTTP服务器，它通过HTTP协议与客户端通信（客户端通常指的是Web浏览器，手机的客户端也是浏览器实现的）

* TCP/IP协议（来源于百度百科）：Transmission Control Protocol/Internet Protocol的简写，中译名为传输控制协议/[因特网](https://baike.baidu.com/item/%E5%9B%A0%E7%89%B9%E7%BD%91)互联协议，又名网络[通讯协议](https://baike.baidu.com/item/%E9%80%9A%E8%AE%AF%E5%8D%8F%E8%AE%AE)，是Internet最基本的协议、Internet国际[互联网](https://baike.baidu.com/item/%E4%BA%92%E8%81%94%E7%BD%91)络的基础，由[网络层](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C%E5%B1%82/4329439)的IP协议和[传输层](https://baike.baidu.com/item/%E4%BC%A0%E8%BE%93%E5%B1%82/4329536)的TCP协议组成。TCP/IP 定义了电子设备如何连入[因特网](https://baike.baidu.com/item/%E5%9B%A0%E7%89%B9%E7%BD%91/114119)，以及数据如何在它们之间传输的标准。协议采用了4层的层级结构，每一层都呼叫它的下一层所提供的协议来完成自己的需求。[通俗](https://baike.baidu.com/item/%E9%80%9A%E4%BF%97/10621346)而言：TCP负责发现[传输](https://baike.baidu.com/item/%E4%BC%A0%E8%BE%93/7078195)的问题，一有问题就发出信号，要求重新传输，直到所有[数据安全](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%AE%89%E5%85%A8/3204964)正确地传输到目的地。而IP是给[因特网](https://baike.baidu.com/item/%E5%9B%A0%E7%89%B9%E7%BD%91/114119)的每一台联网设备规定一个地址。
* HTTP协议（来源于百度百科）：[超文本传输协议](https://baike.baidu.com/item/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE/8535513)（HTTP，HyperText Transfer Protocol)是[互联网](https://baike.baidu.com/item/%E4%BA%92%E8%81%94%E7%BD%91)上应用最为广泛的一种[网络协议](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE/328636)。所有的[WWW](https://baike.baidu.com/item/WWW)文件都必须遵守这个标准。设计HTTP最初的目的是为了提供一种发布和接收[HTML](https://baike.baidu.com/item/HTML)页面的方法。1960年美国人[Ted Nelson](https://baike.baidu.com/item/Ted%20Nelson)构思了一种通过[计算机](https://baike.baidu.com/item/%E8%AE%A1%E7%AE%97%E6%9C%BA/140338)处理文本信息的方法，并称之为超文本（hypertext）,这成为了HTTP超文本传输协议标准架构的发展根基。Ted Nelson组织协调万维网协会（World Wide Web Consortium）和[互联网工程工作小组](https://baike.baidu.com/item/%E4%BA%92%E8%81%94%E7%BD%91%E5%B7%A5%E7%A8%8B%E5%B7%A5%E4%BD%9C%E5%B0%8F%E7%BB%84/6992977)（Internet Engineering Task Force ）共同合作研究，最终发布了一系列的[RFC](https://baike.baidu.com/item/RFC)，其中著名的RFC 2616定义了HTTP 1.1。

* Web服务器的工作的原理：
  * 客户端通过TCP/IP协议建立到服务器TCP连接
  *  客户端想服务器发送HTTP协议请求包，请求服务器里的资源文档
   *   服务器向客户端发送发送HTTP协议应答包，如果请求的资源包含有动态语言的内容，那么服务器会调用动态语言的解析引擎负责处理动态内容，并将处理得到的数据返回给客户端
   *  客户端与服务端断开，由客户端解释HTML文档，在客户端屏幕上渲染图形结果
###需要注意的是：客户端和服务器之间的通信是非持久连接的，也就是当服务器发送了应答后就与客户端断开连接了，等待下一次请求。

##URL和DNS解析
 * URL(Uniform Resource Locator)是“统一资源定位符”的英文缩写，用于描述一个网络上的资源
 *  URL由三部分组成：资源类型、存放资源的主机域名、资源文件名。
*   URL的一般语法格式为：(带方括号[]的为可选项)：scheme://host[:port#]/path/.../[?query-string][#anchor]
```
scheme://host[:port#]/path/.../[?query-string][#anchor]
scheme         指定底层使用的协议(例如：http, https, ftp)
host           HTTP服务器的IP地址或者域名
port#          HTTP服务器的默认端口是80，这种情况下端口号可以省略。如果使用了别的端口，必须指明，例如 http://www.cnblogs.com:8080/
path           访问资源的路径
query-string   发送给http服务器的数据
anchor         锚
```
* DNS解析
*  DNS(Domain Name System)是域名系统的英文缩写，是一种组织成域层次结构的计算机和网络服务命名的系统，它用于TCP/IP网络，它从事将主机名或者域名转换成为实际的IP地址的工作。NDS就是一位翻译官

![ DNS工作原理](https://upload-images.jianshu.io/upload_images/5363507-965eb5144d95b3fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 更加详细的工作流程如下：
   *  1、在浏览器中输入'www.jianshu.com'域名，操作系统会检查自己本地的hosts文件是否有这个网址映射的关系，如果有机会调用这个IP地址映射，完成域名解析（这里有个骚操作，在双十一的时候，把本地的hosts的文件指向你自己的静态页面，随便写个404的页面，告诉你女朋友说，服务器挂了，买不了，哈哈哈哈哈哈☺）
![hosts](https://upload-images.jianshu.io/upload_images/5363507-c95e1b79e1c55956.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    *  2、如果`hosts`没有这个域名的映射，就会查找本地的DNS解析器缓存，是否有这个网址映射的关系，如果有，直接返回，完成域名解析。
     *  3、如果`hosts`于本地DNS解析器缓存都没有相应的网址映射关系，首先会找`TCP/IP`参数中设置的首选的DNS服务器（有时候我们翻墙就要改动这里），本地DNS服务器，此服务器收到查询时候，如果要查询的域名，它包含本地配置区域资源中，则返回解析结果给客户机，完成域名解析，这个解析具有权威性
![DNS设置](https://upload-images.jianshu.io/upload_images/5363507-24b3b29997ab7445.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    *  4、如果要查询的域名，不由本地DNS服务器区域解析，但该服务器已经缓存了此网址的映射的关系，则调用这个IP地址映射，完成域名解析，此解析不具有权威性
    *  5、如果本地DNS服务器本地区域文件与缓存解析都失效，则根据本地DNS服务器设置（是否设置了转发器）进行查询，如果没有使用转发模式，本地的DNS就把请求转发到根DNS服务器，根服务器收到请求了会去判断这个域名（.com）是谁来授权管理，并会返回一个负责该顶级域名服务器的一个IP。本地服务收到服务器的IP信息了，将会联系负责`.com`的这台服务器。这台服务器负责`.com`域的服务器收到请求了，如果自己无法解析，它就会找一个管理`.com`域下一级DNS服务器地址(jianshu.com)给本地的DNS服务器。当本地的DNS服务器收到这个地址后，就会找`jianshu.com`,重复上面的动作，进行查询，直到找到`www.jianshu.com`这个地址。
    *  6、如果用的转发的模式，此DNS服务器就会把这个请求转发至上一级DNS服务器，由上一级服务器进行解析，如果上一层服务器不能解析，或者是根DNS服务器吧请求转至上上级。不管本地DNS服务器用的是转发，还是根提示，左后都是把结果返回给本地DNS服务器，由此DNS服务器在返回给客户机
![ DNS解析的整个流程](https://upload-images.jianshu.io/upload_images/5363507-75332fcf7f5f1f38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  所谓递归查询过程：就是 “查询的递交者” 更替,查询提交者不断的更变,
*  而迭代查询过程： 则是 “查询的递交者”不变。

*  举个例子来说，你想知道某个一起上法律课的女孩的电话，并且你偷偷拍了她的照片，回到寝室告诉一个很仗义的哥们儿，这个哥们儿二话没说，拍着胸脯告诉你，甭急，我替你查(此处完成了一次递归查询，即，问询者的角色更替)。然后他拿着照片问了学院大四学长，学长告诉他，这姑娘是xx系的；然后这哥们儿马不停蹄又问了xx系的办公室主任助理同学，助理同学说是xx系yy班的，然后很仗义的哥们儿去xx系yy班的班长那里取到了该女孩儿电话。(此处完成若干次迭代查询，即，问询者角色不变，但反复更替问询对象)最后，他把号码交到了你手里。完成整个查询过程。

*  通过上面的步骤，最终获取IP地址，也就是浏览器最后发起请求的时候基于IP来和服务器做信息交换的

####HTTP协议详解 
*  HTTP是一种让`Web服务器`与浏览器(客户端)通过`Internet发送与接收数据的协议`,它建立在`TCP协议`之上，一般采用`TCP的80端口`。
*  它是一个请求、响应协议--客户端发出一个请求，服务器响应这个请求。
*  在HTTP中，客户端总是通过建立一个连接与发送一个HTTP请求来发起一个事务。
*   服务器不能主动去与客户端联系，也不能给客户端发出一个回调连接。客户端与服务器端都可以提前中断一个连接。例如，当浏览器下载一个文件时，你可以通过点击“停止”键来中断文件的下载，关闭与服务器的HTTP连接。
* HTTP协议是无状态的，同一个客户端的这次请求和上次请求是没有对应关系，对HTTP服务器来说，它并不知道这两个请求是否来自同一个客户端。为了解决这个问题， `Web程序`引入了`Cookie机制`来维护连接的可持续状态。
*  `Cookie`意为“甜饼”，是由W3C组织提出，最早由Netscape社区发展的一种机制。目前Cookie已经成为标准，所有的主流浏览器如IE、Netscape、Firefox、Opera等都支持Cookie。由于HTTP是一种无状态的协议，服务器单从网络连接上无从知道客户身份。怎么办呢？就给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理。
 *  HTTP协议是建立在TCP协议之上的，因此TCP攻击一样会影响HTTP的通讯，例如比较常见的一些攻击：`SYN Flood`是当前最流行的DoS（拒绝服务攻击）与DdoS（分布式拒绝服务攻击）的方式之一，这是一种利用`TCP协议`缺陷，发送大量伪造的TCP连接请求，从而使得被攻击方资源耗尽（CPU满负荷或内存不足）的攻击方式。

#### HTTP请求包（浏览器信息）
* Request包的结构, Request包分为3部分，第一部分叫Request line（请求行）, 第二部分叫Request header（请求头）,第三部分是body（主体）。header和body之间有个空行，请求包的例子所示:
```
GET /domains/example/ HTTP/1.1		//请求行: 请求方法 请求URI HTTP协议/协议版本
Host：www.iana.org				//服务端的主机名
User-Agent：Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.4 (KHTML, like Gecko) Chrome/22.0.1229.94 Safari/537.4			//浏览器信息
Accept：text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8	//客户端能接收的MIME
Accept-Encoding：gzip,deflate,sdch		//是否支持流压缩
Accept-Charset：UTF-8,*;q=0.5		//客户端字符编码集
//空行,用于分割请求头和消息体
//消息体,请求资源参数,例如POST传递的参数

```

*  抓取简书的首页的信息
```
Request URL:https://www.jianshu.com/
Request Method:GET
Status Code:304 Not Modified
Remote Address:127.0.0.1:8888
Response Headers
view source
Cache-Control:max-age=0, private, must-revalidate
Connection:keep-alive
Content-Security-Policy:script-src 'self' 'unsafe-inline' 'unsafe-eval' *.jianshu.com *.jianshu.io api.geetest.com static.geetest.com dn-staticdown.qbox.me zz.bdstatic.com *.google-analytics.com hm.baidu.com push.zhanzhang.baidu.com res.wx.qq.com qzonestyle.gtimg.cn as.alipayobjects.com ;style-src 'self' 'unsafe-inline' *.jianshu.com *.jianshu.io api.geetest.com static.geetest.com ;
Date:Mon, 02 Jul 2018 09:10:04 GMT
ETag:W/"e5bf3f5c57dcac3addf979d6e74fa906"
Server:Tengine
Set-Cookie:locale=zh-CN; path=/
Strict-Transport-Security:max-age=31536000; includeSubDomains; preload
X-Content-Type-Options:nosniff
X-Dscp-Value:0
X-Frame-Options:DENY
X-Request-Id:01a51f5d-b6ea-45c2-91a7-5fdec3a8eb70
X-Runtime:0.022833
X-Via:1.1 PSfjfzdx2mj93:9 (Cdn Cache Server V2.0), 1.1 yangdianxin66:10 (Cdn Cache Server V2.0)
X-XSS-Protection:1; mode=block
Request Headers
view source
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding:gzip, deflate, sdch, br
Accept-Language:zh-CN,zh;q=0.8
Cache-Control:max-age=0
Connection:keep-alive
Cookie:__guid=163745081.662820947905634200.1526461023901.8958; signin_redirect=https%3A%2F%2Fwww.jianshu.com%2F; read_mode=day; default_font=font2; monitor_count=5; sensorsdata2015jssdkcross=%7B%22distinct_id%22%3A%22163682ac930acd-0e49b894cda713-6b1b1279-2073600-163682ac9319a0%22%2C%22%24device_id%22%3A%22163682ac930acd-0e49b894cda713-6b1b1279-2073600-163682ac9319a0%22%2C%22props%22%3A%7B%22%24latest_traffic_source_type%22%3A%22%E7%9B%B4%E6%8E%A5%E6%B5%81%E9%87%8F%22%2C%22%24latest_referrer%22%3A%22%22%2C%22%24latest_referrer_host%22%3A%22%22%2C%22%24latest_search_keyword%22%3A%22%E6%9C%AA%E5%8F%96%E5%88%B0%E5%80%BC_%E7%9B%B4%E6%8E%A5%E6%89%93%E5%BC%80%22%7D%7D; Hm_lvt_0c0e9d9b1e7d617b3e6842e85b9fb068=1528795394,1530514591; Hm_lpvt_0c0e9d9b1e7d617b3e6842e85b9fb068=1530522699; locale=zh-CN
Host:www.jianshu.com
If-None-Match:W/"e5bf3f5c57dcac3addf979d6e74fa906"
Upgrade-Insecure-Requests:1
User-Agent:Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36
```

*  HTTP协议定义了很多与服务器交互的请求方法，最基本的有4种，分别是GET,POST,PUT,DELETE。
*  一个URL地址用于描述一个网络上的资源
*   而HTTP中的GET, POST, PUT, DELETE就对应着对这个资源的查，增，改，删4个操作。
*   最常见的就是GET和POST了。GET一般用于获取/查询资源信息，而POST一般用于更新资源信息。


####  由于简书地址是`https`,我通过fiddler去抓一个不是http的，看起来比较好看

![get请求](https://upload-images.jianshu.io/upload_images/5363507-ffe931a62ec9b6b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![post请求](https://upload-images.jianshu.io/upload_images/5363507-c1cbdc9b1689de16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  可以得出GET和POST的区别:
    *  GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditPosts.aspx?name=test1&id=123456。POST方法是把提交的数据放在HTTP包的body中。
   *  GET提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制。
   *   GET方式提交数据，会带来安全问题，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码。

#### HTTP响应包（服务器信息）
```
HTTP/1.1 200 OK						//状态行
Server: nginx/1.0.8					//服务器使用的WEB软件名及版本
Date:Date: Tue, 30 Oct 2012 04:14:25 GMT		//发送时间
Content-Type: text/html				//服务器发送信息的类型
Transfer-Encoding: chunked			//表示发送HTTP包是分段发的
Connection: keep-alive				//保持连接状态
Content-Length: 90					//主体内容长度
//空行 用来分割消息头和主体
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"... //消息体
```

* 获取新浪首页的数据如下
``` 
HTTP/1.1 200 OK
Server: nginx
Date: Mon, 02 Jul 2018 09:21:33 GMT
Content-Type: text/html
Content-Length: 576063
Connection: keep-alive
Last-Modified: Mon, 02 Jul 2018 09:20:01 GMT
Vary: Accept-Encoding
X-Powered-By: shci_v1.03
Expires: Mon, 02 Jul 2018 09:22:28 GMT
Cache-Control: max-age=60
Age: 4
Via: http/1.1 ctc.ningbo.ha2ts4.97 (ApacheTrafficServer/6.2.1 [cHs f ]), http/1.1 ctc.xiamen.ha2ts4.41 (ApacheTrafficServer/6.2.1 [cHs f ])
X-Via-Edge: 153052329372561c48b773cd64cde73000845
X-Cache: HIT.41
X-Via-CDN: f=edge,s=ctc.xiamen.ha2ts4.34.nb.sinaedge.com,c=119.139.196.97;f=Edge,s=ctc.xiamen.ha2ts4.41,c=222.76.214.34

<!DOCTYPE html>
<!-- [ published at 2018-07-02 17:20:00 ] -->
<html>
<head>
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>新浪首页</title>
	<meta name="keywords" content="新浪,新浪网,SINA,sina,sina.com.cn,新浪首页,门户,资讯" />
	<meta name="description" content="新浪网为全球用户24小时提供全面及时的中文资讯，内容覆盖国内外突发新闻事件、体坛赛事、娱乐时尚、产业资讯、实用信息等，设有新闻、体育、娱乐、财经、科技、房产、汽车等30多个内容频道，同时开设博客、视频、论坛等自由互动交流空间。" />
    <link rel="mask-icon" sizes="any" href="//www.sina.com.cn/favicon.svg" color="red">
	<meta name="stencil" content="PGLS000022" />
...
```

* `Response`包中的第一行叫做状态行，由HTTP协议版本号、状态码、状态消息三部分组成
* 状态码用来告诉HTTP客户端，HTTP服务器是否产生了预期的Response。HTTP/1.1协议中定义了5类状态码，第一个数字定义了响应的类别
  *   1XX：提示信息-表示请求已被成功接收，继续处理
  *   2XX：成功 。表示请求已被成功接收，理解，接收
  *  3XX: 重定向：要完成请求必须进行更进一步的处理
  *  4XX：客户端错误：请求语法错误或者是请求无法实现
  *  5XX：服务器错误，服务器未能够实现合法的请求
* 当输入的简书的地址为'htt://www.jianshu.com/'的时候，状态码为`301`
![image.png](https://upload-images.jianshu.io/upload_images/5363507-cd19ecadfecd9e0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 301 Moved Permanently
被请求的资源已永久移动到新位置，并且将来任何对此资源的引用都应该使用本响应返回的若干个 URI 之一。如果可能，拥有链接编辑功能的客户端应当自动把请求的地址修改为从服务器反馈回来的地址

* 使用网易云在听歌的时候
![image.png](https://upload-images.jianshu.io/upload_images/5363507-cfd00450a0861b7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 206 Partial Content
服务器已经成功处理了部分 GET 请求。类似于 FlashGet 或者迅雷这类的 HTTP下载工具都是使用此类响应实现断点续传或者将一个大文档分解为多个下载段同时下载


#### HTTP协议是无状态的和Connection: keep-alive的区别
* 无状态是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。从另一方面讲，打开一个服务器上的网页和你之前打开这个服务器上的网页之间没有任何联系。
* HTTP是一个无状态的面向连接的协议，无状态不代表HTTP不能保持TCP连接，更不能代表HTTP使用的是UDP协议（面对无连接）。
*  从HTTP/1.1起，默认都开启了Keep-Alive保持连接特性，简单地说，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的TCP连接。
* Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同服务器软件（如Apache）中设置这个时间。

![一次请求的request和response](https://upload-images.jianshu.io/upload_images/5363507-760d4b35e0b2db65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*  一次URL请求但是左边栏里面为什么会有那么多的资源请求？
    * 这个就是浏览器的一个功能，第一次请求url，服务器端返回的是`html`页面，然后浏览器开始渲染HTML：当解析到HTML DOM里面的图片连接，css脚本和js脚本的链接，浏览器就会自动发起一个请求静态资源的HTTP请求，获取相对应的静态资源，然后浏览器就会渲染出来，最终将所有资源整合、渲染，完整展现在我们面前的屏幕上。
*  最后说明一点 ：网页优化方面有一项措施是减少HTTP请求次数，就是把尽量多的css和js资源合并在一起，目的是尽量减少网页请求静态资源的次数，提高网页加载速度，同时减缓服务器的压力。
*  鉴于本人的能力有限，如有错误还望指出，tks








