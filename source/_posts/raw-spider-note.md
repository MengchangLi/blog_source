---
title: 原始笔记--python爬虫
author: chanoch
toc: true
mathjax: true
date: 2023-01-30 13:03:10
tags: 
- 爬虫
- 原始笔记
- python
category: 
- 原始笔记
description: 学习python爬虫的原始笔记，尚未进行整理。
---

# 预备知识

## URL

- URI称为统一资源标志符，包含URL(统一资源定位符)和URN(统一资源名称)

  ![image-20230202193031196](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/image-20230202193031196.png)

- 但URN几乎不使用，一般网页链接可以直接称为URL

- url的格式如下：

  ```
  scheme://netloc/path;params?query#fragment
  ```

  - `scheme`：使用的协议
  - `netloc`：域名
  - `path`：路径
  - `params`：参数
  - `query`：查询条件，一般用于GET类型的URL
  - `fragment`：锚点，定位页面内部的下拉位置

## HTTP和HTTPS

- 超文本标记语言采用标签来表示不同的元素，排列成为网页

- HTTP和HTTPS是超文本传输协议，客户端和服务端根据HTTP协议请求和响应数据

- HTTPS是在HTTP的基础上经过SSL进行加密，客户端和服务端遵守协议进行通讯，加密方式有以下三种：
  - 对称密钥加密：密钥和密文一同传输
  - 非对称密钥加密：制定公钥和私钥，先将公钥发送给客户端(效率低；无法保证客户端拿到的是服务器端的公钥)
  - 证书密钥加密：证书认证机构对公钥进行签名(HTTPS使用)

- 请求

  - 请求方法：常见分为GET和POST，其它可以参考[http://www.runoob.com/http/http-methods.html](http://www.runoob.com/http/http-methods.html)

    - GET请求的参数包含在URL中，POST请求的参数以表单形式传输，包含在请求体中

      > 一般POST请求的表单数据会在响应的from字段中返回

    - GET请求提交数据有1024B的限制，POST没有

      > 一般情况下，提交敏感信息使用POST请求

  - 请求地址：即URL

  - 请求头：要使用的附加信息
    - `Accept`：请求报头域，指定客户端可以接受的信息类型
    - `Accept-Language`：指定客户端能接受的语言
    - `Accept-Encoding`：指定客户端能接受的编码
    - `Host`：指定请求将要发送到的服务器主机名和端口号，HTTP1.1后必须包含
    - `Cookie`：标识维持当前访问会话的数据
    - `Referer`：标识请求是从哪个界面发送的
    - `User-Agent`：标识客户使用的操作系统及版本、浏览器及版本等信息
    - `Content-Type`：也叫互联网媒体类型，表示具体请求中媒体类型信息，可以查阅[https://tool.oschina.net/commons](https://tool.oschina.net/commons)

  - 请求体：POST请求为表单数据(此时请求头中`Content-Type` 与POST请求有固定对应关系)，GET则为空

    | Content-Type                      | 提交数据的方式   |
    | --------------------------------- | ---------------- |
    | application/x-www-form-urlencoded | 表单数据         |
    | multipart/form-data               | 表单文件上传     |
    | application/json                  | 序列化 JSON 数据 |
    | text/xml                          | XML 数据         |

- 响应

  - 响应状态码：200为成功，404为未找到页面，500为服务器内部错误等
  - 响应头：
    - `Date`：响应产生的时间
    - `Last-Modified`：指定资源的最后修改时间
    - `Content-Encoding`：指定响应内容的编码
    - `Server`：包含服务器的信息，比如名称、版本号等
    - `Content-Type`：文档类型
    - `Set-Cookie`：设置 Cookies。响应头中的 `Set-Cookie `告诉浏览器需要将此内容放在 Cookies 中，下次请求携带 Cookies 请求
    - `Expires`：指定响应的过期时间，如果在期限内再次访问时，就可以直接从缓存中加载

  - 响应体：内容即为数据



## 网页

### 网页构成

- HTML：即超文本标记语言，使用不同类型的标签表示不同的元素，嵌套排列成为网页

  > 一个网页的标准形式是 html 标签内嵌套 `head` 和 `body `标签，`head `内定义网页的配置和引用，`body` 内定义网页的正文

- CSS：即层叠样式表，层叠指HTML引用了几个样式文件且发生冲突时，浏览器依据层叠顺序处理，样式是指网页中元素的排列，大小，格式，间距等

- JavaScript：脚本语言，实现了实时动态的交互功能，使用`<script>`引入

### 网页结构

- 节点树及节点：根据[W3C(万维网联盟)的DOM标准](http://www.w3school.com.cn/htmldom/dom_nodes.asp)，HTML内容都为节点，构成节点树，具有相应地层级关系，顶端节点为根(root)

![](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/2-11.jpg)

-  CSS选择器：css利用选择器来定位节点

  > 选择器也是爬虫解析网页的一种方式

  - 选择器中间加空格表示嵌套，不加空格表示并列，`> `表示直属的父子标签

  - `#`开头代表选择id，后紧跟id名称

  - `.`代表选择class，后紧跟class名称

  - 标签名也是筛选

  - 其它语法如下表

    | 选　择　器           | 例　　子           | 例子描述                                      |
    | -------------------- | ------------------ | --------------------------------------------- |
    | *                    | *                  | 选择所有节点                                  |
    | element,element      | div,p              | 选择所有 div 节点和所有 p 节点                |
    | element+element      | div+p              | 选择紧接在 div 节点之后的所有 p 节点          |
    | [attribute]          | [target]           | 选择带有 target 属性的所有节点                |
    | [attribute=value]    | [target=blank]     | 选择 target="blank" 的所有节点                |
    | [attribute~=value]   | [title~=flower]    | 选择 title 属性包含单词 flower 的所有节点     |
    | :link                | a:link             | 选择所有未被访问的链接                        |
    | :visited             | a:visited          | 选择所有已被访问的链接                        |
    | :active              | a:active           | 选择活动链接                                  |
    | :hover               | a:hover            | 选择鼠标指针位于其上的链接                    |
    | :focus               | input:focus        | 选择获得焦点的 input 节点                     |
    | :first-letter        | p:first-letter     | 选择每个 p 节点的首字母                       |
    | :first-line          | p:first-line       | 选择每个 p 节点的首行                         |
    | :first-child         | p:first-child      | 选择属于父节点的第一个子节点的所有 p 节点     |
    | :before              | p:before           | 在每个 p 节点的内容之前插入内容               |
    | :after               | p:after            | 在每个 p 节点的内容之后插入内容               |
    | :lang(language)      | p:lang             | 选择带有以 it 开头的 lang 属性值的所有 p 节点 |
    | element1~element2    | p~ul               | 选择前面有 p 节点的所有 ul 节点               |
    | [attribute^=value]   | a[src^="https"]    | 选择其 src 属性值以 https 开头的所有 a 节点   |
    | [attribute$=value]   | a[src$=".pdf"]     | 选择其 src 属性以.pdf 结尾的所有 a 节点       |
    | [attribute*=value]   | a[src*="abc"]      | 选择其 src 属性中包含 abc 子串的所有 a 节点   |
    | :first-of-type       | p:first-of-type    | 选择属于其父节点的首个 p 节点的所有 p 节点    |
    | :last-of-type        | p:last-of-type     | 选择属于其父节点的最后 p 节点的所有 p 节点    |
    | :only-of-type        | p:only-of-type     | 选择属于其父节点唯一的 p 节点的所有 p 节点    |
    | :only-child          | p:only-child       | 选择属于其父节点的唯一子节点的所有 p 节点     |
    | :nth-child(n)        | p:nth-child        | 选择属于其父节点的第二个子节点的所有 p 节点   |
    | :nth-last-child(n)   | p:nth-last-child   | 同上，从最后一个子节点开始计数                |
    | :nth-of-type(n)      | p:nth-of-type      | 选择属于其父节点第二个 p 节点的所有 p 节点    |
    | :nth-last-of-type(n) | p:nth-last-of-type | 同上，但是从最后一个子节点开始计数            |
    | :last-child          | p:last-child       | 选择属于其父节点最后一个子节点的所有 p 节点   |
    | :root                | :root              | 选择文档的根节点                              |
    | :empty               | p:empty            | 选择没有子节点的所有 p 节点（包括文本节点）   |
    | :target              | #news:target       | 选择当前活动的 #news 节点                     |
    | :enabled             | input:enabled      | 选择每个启用的 input 节点                     |
    | :disabled            | input:disabled     | 选择每个禁用的 input 节点                     |
    | :checked             | input:checked      | 选择每个被选中的 input 节点                   |
    | :not(selector)       | :not               | 选择非 p 节点的所有节点                       |
    | ::selection          | ::selection        | 选择被用户选取的节点部分                      |

### 动态页面

- 使用AJAX，ASP，JSP等动态网页技术实现网页的局部加载或刷新(一般通过抓包获取交换的JSON数据)
- 浏览器打开页面时，先加载HTML内容，在引入文件处执行代码，而代码可能改变HTML中的节点
- 在爬虫请求时，不会加载代码，需要分析Ajax接口或使用Selenium，Splash等库实现渲染

- 检测：

  - 对URL发请求，查看是否缺失部分数据
  - 抓包工具查看是否有动态请求
  - 浏览网页是否有局部刷新

- 提取：

  - 直接抓包
  - 用正则解析html中的js等源码

### 浏览器抓包

- 一般F12进入浏览器抓包，Elements为网页源码数据，Network为数据包
- Network中XHR表示动态请求，可以查询到GET/POST方法，请求的url，参数，响应数据格式(`Content-Type`)，响应数据

- `Content-Type`：
  - `text/plain`：字符串
  - `application/json`：json格式

### 编码格式

- requests库会基于http头部中`Content-Type`对响应的编码进行推测，其中的charset决定了编码方式，如果未指定，`Content-Type`默认为`text/html`

- 可以使用chardet库推测编码格式

  ```python
  chardet.detect(raw_data)
  ```

- 可以通过修改`response`的`encoding`属性进行修改

  ```python
  response.encoding = chardet.detect(response.content)
  ```


> 通用中文乱码：
>
> 1.整体设置为utf-8
>
> 2.
>
> ```python
> str.encode('iso-8859-1').decode('gbk')
> ```





## 会话和Cookie

- HTTP是无状态的，即服务器不知道客户端是什么状态
- 会话(Session)和Cookie用于保持HTTP连接状态
  - 会话在服务器端，保存用户的会话信息，存储特定用户的属性和配置信息
  - Cookie在客户端，在浏览器请求时附带上Cookie，服务器能识别用户，判断状态，返回对应的响应
- 会话维持：客户端第一次请求服务器时，服务器会在响应头中带有`Set-Cookie`字段，标记用户，在下一次请求时，浏览器就会把Cookie放入请求
- 属性结构：
  - `Name`：Cookie名称，一旦创建，不可更改
  - `Value`：Cookie的值，如果为Unicode，采用字符编码，如果为二进制，使用BASE64编码
  - `Max Age`：失效时间，单位为秒，如果为负数，表示关闭浏览器即失效
  - `Path`：Cookie使用的路径，如果设定，则只有路径为/Path/的页面能用此Cookie
  - `Domain`：可以访问该Cookie的域名
  - `Size`：Cookie的大小
  - `Http`字段：Cookie的`httponly`属性，若为`True`，只能在`HTTP Headers`中带有Cookie信息，不能通过`document.cookie`访问
  - `Secure`：是否使用安全协议，默认为`False`


- 持久Cookie：严格来说，是`Max Age`或`Expires`字段设置较长的Cookie，会保存在营盘中

> 关闭浏览器时，服务器无法知晓，因此关闭浏览器不会导致会话失效



## 代理

- 代理指代理服务器，它能代理网络用户区获取网络信息，本机在访问Web服务器时，由代理服务器进行中转
- 优势：
  - 突破自身IP访问限制
  - 访问一些单位或团体内部资源
  - 提高访问速度(相对)，代理服务器通常有一个硬盘缓冲区，在多用户访问相同信息时，可以从缓冲区提取信息，提高访问速度
  - 隐藏真实IP

- 分类：
  - 按协议分
  - 按匿名程度
    - 高度匿名：将数据包原封不动地转发，且记录的IP是代理服务器的IP
    - 普通匿名：会在数据包上做一些改动，通常在HTTP头中加入`HTTP_VIA` 和 `HTTP_X_FORWARDED_FOR`，服务端可以知晓代理服务器，也有一定几率查到客户端真实IP
    - 透明代理：不但改动数据包，也会告诉服务器端客户端的真实IP，常见的如内网中硬件防火墙

- 常见代理
  - 免费代理
  - 付费代理服务
  - ADSL拨号，拨一次号更换次以IP



## Json串

- 动态刷新的内容一般为Json串，可以使用request模块的`response.json()`获取

- `json.dump()`

  将字典类型的数据以Json格式存入文件

  ```python
  import json
  
  json.dump(dict_obj,file,ensure_ascii)
  ```

  - `dict_obj`：字典类型的数据
  - `file`：打开的文件描述符
  - `ensure_ascii`：默认为True，对于非ASCII码，需要设置为`False`



# 爬虫基本原理

## 概述

### 定义

- 爬虫，就是通过编写程序，**模拟**浏览器上网，让其去**抓取**互联网上数据，简单来说，爬虫就是获取网页并提取和保存信息的自动化程序

### 流程

- 获取网页源码
- 分析网页源码，提取想要的数据，常见的有正则，CSS选择器，Xpath等
- 保存数据，持久化存储

### 分类

- 通用爬虫：搜索引擎的抓取系统的重要组成部分

- 聚焦爬虫：建立在通用爬虫的基础上，抓取页面中特定的局部内容

- 增量式爬虫：监测网站中数据更新的情况，只会抓取网站中最新更新的数据

### 合法性

- 在法律中不被禁止
- 但具有违法风险
  - 善意爬虫：
  - 恶意爬虫：干扰了被访问网站的正常运营；抓取到了受到法律保护的特定类型的数据或信息

- 避免风险

  - 优化自己的程序，避免干扰访问网站的正常运行

  - 传播爬取到的数据时，审查抓取到的内容，涉及到商业机密等敏感内容时需要及时停止爬取或传播



## 矛与盾

### 反爬机制

- 通过制定相应的策略或技术手段，防止爬虫程序进行数据的爬取

### 反反爬策略

- 爬虫程序通过制定相关的策略或技术手段，破解反爬机制

### robots.txt协议

- 也成为网络爬虫排除标准
- 君子协议，规定了网站中不能被爬虫爬取的数据
- 通常是一个叫作 robots.txt 的文本文件，一般放在网站的根目录下

### UA伪装

- UA检测：网站服务器可能基于身份标识进行检测

- UA伪装：让爬虫的身份标识伪装为浏览器



# 基本请求库

## urllib库

> `urllib`模块出现较早，可以被更高效的`requests`模块替代

- python内置的HTTP请求库，分为四个模块
  - `request`：HTTP请求模块
  - `error`：异常请求模块
  - `parse`：工具模块，主要提供URL处理方法
  - `robotparser`：用于识别robots.txt文件，使用较少

### 发送请求(request模块)

- `urlopen`

  > `urlopen`文档：[https://docs.python.org/3/library/urllib.request.html](https://docs.python.org/3/library/urllib.request.html)

  提供了最基本的HTTP请求方法，还带有处理授权验证，重定向，Cookie等功能

  ```python
  import urllib.request
  
  data = bytes(urllib.parse.urlencode({'':''}, encoding=''))
  response = urllib.request.urlopen(url, data=data. timeout=)
  ```

  - `response`：`urlopen`返回一个`HTTPResponse`对象，包含`read`，`readinto`，`getheader`，`getheaders`，`fileno`等方法，以及`msg`，`ersion`，`status`，`reason`，`debuglevel`，`closed`属性

  - `data`：bytes类型的请求数据(需要用`bytes`方法转换，要先用`urlencode`方法将字典转换为字符串)，设置后，请求方式为POST

  - `timeout`：设置超时时间，超时就会抛出`urllib.error.URLError`错误，可以使用`reason`方法查看，`try-except`捕获

    ```python
    import socket  
    import urllib.request  
    import urllib.error  
    
    try:  
        response = urllib.request.urlopen('http://httpbin.org/get', timeout=0.1)  
    except urllib.error.URLError as e:  
        if isinstance(e.reason, socket.timeout):  
            print('TIME OUT')
    ```

  - `context`：用于指定SSL设置，必须为`ssl.SSSLContext`类型
  - `cafile`和`capath`：指定CA证书和路径

- `Request`

  `Request`类可以为请求加入Headers等信息，可以直接作为`urlopen`的参数

  ```python
  import urllib.request  
  
  request = urllib.request.Request('https://python.org')  
  response = urllib.request.urlopen(request)  
  ```

  ```python
  class urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)
  ```

  - `url`：指定请求的url

  - `data`：必须为`bytes`类型，如果为字典，先用`urllib.parse.urlencode()`进行编码再转换为`bytes`类型

  - `headers`：字典类型，即请求头

    > `headers`可以使用`Request`的`add_headers`添加，参数为两个字符串

  - `origin_req_host`：请求的host名称或IP

  - `unverifiable`：请求是否无法验证，即用户是否有权限接受请求的结果，默认为`False`

  - `nethod`：指定请求的方法

- `Handler`类

  `urllib.request`中`Handler`类是各种处理器，可以用于处理登陆验证，Cookies等

  - `BaseHandler`：其它`Handler`的父类，提供最基本的方法

  - `HTTPRedirectHandler`：用于处理重定向

  - `HTTPCookieProcessor`：用于处理Cookies

    更多可以参阅：[https://docs.python.org/3/library/urllib.request.html#urllib.request.BaseHandler](https://docs.python.org/3/library/urllib.request.html#urllib.request.BaseHandler)

- `OpenerDirector`类

  `OpenerDirector`类也叫Opener，可以实现更高级的功能，`urlopen`就是`urllib`提供的一个Opener，需要借助`Handler`构建Opener

  - 验证

    ![3-2](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/3-2.jpg)

    ```python
    from urllib.request import HTTPPasswordMgrWithDefaultRealm, HTTPBasicAuthHandler, build_opener  
    from urllib.error import URLError  
    
    username = 'username'  
    password = 'password'  
    url = 'http://localhost:5000/'  
    
    p = HTTPPasswordMgrWithDefaultRealm()  
    p.add_password(None, url, username, password)  
    auth_handler = HTTPBasicAuthHandler(p)  
    opener = build_opener(auth_handler)  
    
    result = opener.open(url)
    ```

    最基本的验证界面可以用`HTTPBasicAuthHandler`完成：

    - 首先实例化一个`HTTPBasicAuthHandler`对象，参数为`HTTPPasswordMgrWithDefaultRealm`对象，使用`add_password`方法加入用户名和密码，由此建立一个处理验证的`Handler`对象

    - 其次，将`Handler`作为参数使用`build_opener`创建Opener

    - 最后，使用`open`方法进行请求

  - 代理

    ```python
    from urllib.error import URLError  
    from urllib.request import ProxyHandler, build_opener  
    
    proxy_handler = ProxyHandler({  
        'http': 'http://127.0.0.1:9743',  
        'https': 'https://127.0.0.1:9743'  
    })  
    opener = build_opener(proxy_handler)  
    ```

    代理可以使用`ProxyHandler`来构建Opener

  - Cookies

    生成Cookie必须先声明一个`CookieJar`对象，然后使用`HTTPCookieProcessor`构建`Handler`，再构建Opener，如果要存入文件，可以使用`MozillaCookieJar`或`LWPCookieJar`，两者存储格式不同

    ```python
    import http.cookiejar, urllib.request  
    
    cookie = http.cookiejar.CookieJar() 
    # filename = 'cookies.txt'  
    # cookie = http.cookiejar.MozillaCookieJar(filename)  
    handler = urllib.request.HTTPCookieProcessor(cookie)  
    opener = urllib.request.build_opener(handler)
    response = opener.open('http://www.baidu.com')
    # cookie.save(ignore_discard=True, ignore_expires=True)
    ```

    读取Cookie也需要使用对应格式的类生成`CookieJar`对象，其余与生成时类似

    ```python
    cookie = http.cookiejar.LWPCookieJar()  
    cookie.load('cookies.txt', ignore_discard=True, ignore_expires=True)  
    ```

### 处理异常(error模块)

`error`模块定义了`requests`模块产生的异常，出现问题便会抛出

- `URLError`

  继承自`OSError`类，是`error`模块的基类，由`request`模块产生的异常都可以用`URLError`捕获

  - `reason`属性

    `reason`属性包含了返回错误的原因，有时可能为对象，例如超时为`socket.timeout`对象

- `HTTPError`

  专用于处理HTTP请求错误

  - `code`：包含HTTP状态码
  - `reason`：与父类相同
  - `headers`：返回请求头

### 解析链接(parse模块)

`parse`模块用于处理URL

- `urlparse`方法

  用于拆解标准的URL，返回拆解后的类(实际上为元组，但可以同时使用索引和属性)

  ```python
  urllib.parse.urlparse(urlstring, scheme='', allow_fragments=True)
  ```

  - `urlstring`：待解析的URL
  - `scheme`：指定默认协议，如果链接没有携带协议信息，会使用默认协议
  - `allow_fragments`：是否解析fragments，如果为`False`，则fragments会被解析为path、parameters 或者 query 的一部分

- `urlunparse`方法

  `urlparse`方法的反方法，接受URL的6个参数的可迭代对象(长度必须为6)，拼接为URL

  ```python
  from urllib.parse import urlunparse  
  
  data = ['http', 'www.baidu.com', 'index.html', 'user', 'a=6', 'comment'] 
  print(urlunparse(data))
  ```

- `urlsplit`方法

  与`urlparse`类似，但将params合并到path中，返回5个参数

- `urlunsplit`方法

  与`urlunparse`，但也是5个参数

- `urljoin`方法

  与`urlunparse`和`urlunsplit`类似，生成链接

- `urlencode`方法

  构造GET请求参数

  ```python
  from urllib.parse import urlencode  
  
  params = {  
      'name': 'germey',  
      'age': 22  
  }  
  base_url = 'http://www.baidu.com?'  
  url = base_url + urlencode(params)  
  ```

- `parse_qs`方法

  `urlencode`的反方法，获取url的GET请求的参数，返回为字典

- `parse_qsl`方法

  同上，但返回值为元组组成的列表，元组第一个参数为参数名，第二个值为参数值

- `quote`方法

  对URL进行编码(在URL中只能出现ASCII类型的安全字符)

  ```python
  from urllib.parse import quote  
  
  keyword = ' 壁纸 '  
  url = 'https://www.baidu.com/s?wd=' + quote(keyword)  
  ```

- `unquote`方法

  `quote`的反方法，对URL进行解码

### 分析Robots协议(tobotparser模块)

- `RobotFileParser`类

  根据网站的robots.txt文件判断爬虫是否有权限爬取页面

  - 声明：可以在声明时传入robots.txt的链接，也可以使用`set_url()`方法

    ```python
    urllib.robotparser.RobotFileParser(url='')
    ```

  - `set_url()`：设置robots.txt文件的链接

  - `read()`：读取robots.txt文件并进行分析，在进行判断前，需要执行，否则其它判断都会为False

  - `parse()`：解析robots.txt文件，传入文件某些行的内容，根据语法进行解析

  - `can_fetch()`：传入User-Agent和要抓取的URL，判断是否能抓取

  - `mtime()`：返回上次抓取和分析robot.txt文件的时间，主要用于定期检查抓取最新的robots.txt

  - `modified()`：将当前时间设置为上次抓取和分析robots.txt的时间



## requests模块

- python中原生的基于网络请求的模块
- 作用：模拟浏览器发送请求

### 环境安装

``` 
pip install request
```

### 浏览器请求过程(requests模块的编码流程)

- 指定url

  > requests中会自动对url进行编码

- UA伪装

- 发起请求

- 获取响应数据

- 数据解析

- 持久化存储

### GET请求

- `requests.get()`

  对指定url发起get请求，在请求过程中会自动对参数进行处理

  ```python
  requests.get(url,params,headers)
  ```

  - `url`：发起请求的url(字符串，可以直接带参数)
  - `params`：url携带的参数(字典)
  - `headers`：所使用的头信息(字典)

- 响应对象

  ```python
  response.text			# 获取字符串类型的响应数据
  response.json()			# 获取Json类型的响应数据，生成字典
  response.content		# 获取二进制形式的响应数据
  
  response.status_code	# 获取响应状态码
  response.headers		# 获取响应头
  response.cookies		# 获取cookies
  response.url			# 获取URL
  response.history		# 获取请求历史
  ```

  > 若调用`json()`方法时，返回结果不是json格式，会抛出`json.decoder.JSONDecodeError`异常

  `request`提供了内置的状态码查询对象`requests.codes`

  ```python
  # 信息性状态码  
  100: ('continue',),  
  101: ('switching_protocols',),  
  102: ('processing',),  
  103: ('checkpoint',),  
  122: ('uri_too_long', 'request_uri_too_long'),  
  
  # 成功状态码  
  200: ('ok', 'okay', 'all_ok', 'all_okay', 'all_good', '\\o/', '✓'),  
  201: ('created',),  
  202: ('accepted',),  
  203: ('non_authoritative_info', 'non_authoritative_information'),  
  204: ('no_content',),  
  205: ('reset_content', 'reset'),  
  206: ('partial_content', 'partial'),  
  207: ('multi_status', 'multiple_status', 'multi_stati', 'multiple_stati'),  
  208: ('already_reported',),  
  226: ('im_used',),  
  
  # 重定向状态码  
  300: ('multiple_choices',),  
  301: ('moved_permanently', 'moved', '\\o-'),  
  302: ('found',),  
  303: ('see_other', 'other'),  
  304: ('not_modified',),  
  305: ('use_proxy',),  
  306: ('switch_proxy',),  
  307: ('temporary_redirect', 'temporary_moved', 'temporary'),  
  308: ('permanent_redirect',  
        'resume_incomplete', 'resume',), # These 2 to be removed in 3.0  
  
  # 客户端错误状态码  
  400: ('bad_request', 'bad'),  
  401: ('unauthorized',),  
  402: ('payment_required', 'payment'),  
  403: ('forbidden',),  
  404: ('not_found', '-o-'),  
  405: ('method_not_allowed', 'not_allowed'),  
  406: ('not_acceptable',),  
  407: ('proxy_authentication_required', 'proxy_auth', 'proxy_authentication'),  
  408: ('request_timeout', 'timeout'),  
  409: ('conflict',),  
  410: ('gone',),  
  411: ('length_required',),  
  412: ('precondition_failed', 'precondition'),  
  413: ('request_entity_too_large',),  
  414: ('request_uri_too_large',),  
  415: ('unsupported_media_type', 'unsupported_media', 'media_type'),  
  416: ('requested_range_not_satisfiable', 'requested_range', 'range_not_satisfiable'),  
  417: ('expectation_failed',),  
  418: ('im_a_teapot', 'teapot', 'i_am_a_teapot'),  
  421: ('misdirected_request',),  
  422: ('unprocessable_entity', 'unprocessable'),  
  423: ('locked',),  
  424: ('failed_dependency', 'dependency'),  
  425: ('unordered_collection', 'unordered'),  
  426: ('upgrade_required', 'upgrade'),  
  428: ('precondition_required', 'precondition'),  
  429: ('too_many_requests', 'too_many'),  
  431: ('header_fields_too_large', 'fields_too_large'),  
  444: ('no_response', 'none'),  
  449: ('retry_with', 'retry'),  
  450: ('blocked_by_windows_parental_controls', 'parental_controls'),  
  451: ('unavailable_for_legal_reasons', 'legal_reasons'),  
  499: ('client_closed_request',),  
  
  # 服务端错误状态码  
  500: ('internal_server_error', 'server_error', '/o\\', '✗'),  
  501: ('not_implemented',),  
  502: ('bad_gateway',),  
  503: ('service_unavailable', 'unavailable'),  
  504: ('gateway_timeout',),  
  505: ('http_version_not_supported', 'http_version'),  
  506: ('variant_also_negotiates',),  
  507: ('insufficient_storage',),  
  509: ('bandwidth_limit_exceeded', 'bandwidth'),  
  510: ('not_extended',),  
  511: ('network_authentication_required', 'network_auth', 'network_authentication')
  ```

  例如判断响应是否成功可以使用`requests.codes.ok`

  ```python
  if r.status_code == requests.codes.ok:
      pass
  ```

### POST请求

- `requests.post()`

  对指定url发起post请求

  ```python
  requests.pose(url,data,headers)
  ```

  - `url`：post请求的url(字符串)
  - `data`：请求的参数(字典)

  - `headers`：所使用的头信息(字典)

  > 注意POST请求中参数变为`data`

### 文件上传

```python
import requests

files = {'file': open('favicon.ico', 'rb')}
r = requests.post('http://httpbin.org/post', files=files)
```

> 上传文件的返回值中，会包含file字段，而不使用from字段

### Cookie

- 获取cookies

  ```python
  response.cookies
  for key, value in r.cookies.items():	# 也可以使用items方法遍历
      pass
  ```

- 携带cookies

  ```python
  headers = {
      'Cookies': ''
  }
  ```

  也可以使用`RequestsCookieJar`对象构造

  ```python
  jar = requests.cookies.RequestsCookieJar()
  jar.set(key, value)
  requests.get(url, cookie=jar)
  ```

### 会话维持

每一次get或post请求实际上相当于不同的会话，如果需要会话维持(例如登陆后获取用户信息)可以采用以下两种方法

- 请求时设置相同的cookies值

- 使用`Session`对象

  `Session`对象会保持同一个会话，且不同用户手动设置cookies

  ```python
  import requests
  
  s = requests.Session()
  s.get()
  ```

### SSL证书验证

在发送请求时，可以使用`verify`参数设定是否检查证书，默认为`True`

如果请求一个HTTPS站点，且证书验证错误，则会抛出`requests.excceptions.SSLError`异常

也可以在请求时使用`cert`参数指定本地证书，可以是单个文件（包含密钥和证书）或一个包含两个文件路径的元组

> 注意私有证书的key必须是解密状态，不支持加密

### 代理

可以使用`proxies`参数设置代理进行请求

> `request`可以使用http代理和socks代理，socks代理可以代理http

### 超时设置

使用`timeout`设置超时时间，请求时间分为连接时间和读取时间，传入元组可以分别设置

`timeout`默认为`None`，即永远不会超时

### 身份认证

`requests`也像`urllib`自带身份认证，可以使用`HTTPBasicAuth`类，简略写法可以省去，传入一个元组，`requests`会自动调用

```python
import requests  
from requests.auth import HTTPBasicAuth  

r = requests.get('http://localhost:5000', auth=HTTPBasicAuth('username', 'password'))  
r = requests.get('http://localhost:5000', auth=('username', 'password'))
```

OAuth 认证可以参考[https://requests-oauthlib.readthedocs.org/](https://requests-oauthlib.readthedocs.org/)

### Prepared Request类

`urllib`可以构造`Request`对象进行请求，`requests`也有相同功能，叫做`Prepared Request`

```python
from requests import Request, Session

url = 'http://httpbin.org/post'
data = {'name': 'germey'}
headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36'
}
s = Session()
req = Request('POST', url, data=data, headers=headers)
prepped = s.prepare_request(req)
r = s.send(prepped)
```

`prepare_request`使用`Session`的`prepare_request`方法构造，调用`send()`方法发送

> `Request`对象可以将请求视为独立的对象，在构造队列调度时非常方便



# 数据解析

## 分类

- 正则
- bs4
- ***xpath***

- pyquery

## 原理

-  解析的局部文本内容都会在标签之间或标签的属性中存储
-  步骤：
   - 指定标签的定位
   - 标签或标签对应属性中存储的数据值进行提取



## 正则表达式

> [正则表达式测试工具](http://tool.oschina.net/regex/)

### 规则

- 单字符

  - `. `：除换行符外的任意一个字符

  - `[] `：匹配集合中任意一个字符

  - `[^]`：匹配不在集合中的一个字符

  - `\d` ：匹配一个数字

  - `\w` ：匹配一个数字，字母，下划线，中文

  - `\s` ：所有的空白字符，包括空格，制表符，换页符等。等价于`[ \f\n\r\t\v]`

    > 以上存在大写取反

  - `\t`：匹配制表符

  - `\n`：匹配换行符

- 数量修饰

  - `*` ：0或任意次

  - `+`：至少一次

  - `?` ：0或1次，非贪婪模式

  - `{m}`：固定m次

  - `{m,} `：至少m次

  - `{m,n}`：m-n次


- 边界

  - `$` ：匹配开头

  - `^ `：匹配结尾

  - `\A`：匹配字符串开头

  - `\Z`：匹配字符串结尾，不匹配换行

  - `\z`：匹配字符串结尾，匹配换行

  - `\G`：匹配最后匹配完成的位置

- 分组

  - `(ab)`

    > 使用()可以指明需要提取的对象


- 贪婪模式

  - 贪婪模式：使用`.*`，会匹配尽可能多的内容

  - 非贪婪模式：使用`.*?`，匹配尽可能少的内容

    > 注意如果`.*?`在结尾可能匹配不到内容


- 转义

  如果匹配用于正则中的字符，需要加`\`进行转义

### 参数(拓展正则)

- `re.I`：忽略大小写

- `re.M`：多行匹配，影响^和$

- `re.S`：使.能匹配换行符在内的所有字符

  > 由于HTML中大多有换行符，因此尽量加上`re.S`

### `re`模块

- `re.match()`

  从开头检测正则是否匹配字符串

  ```python
  re.match(regex, content)
  ```

  返回一个`SRE_Match `对象，`group()`方法可以获取匹配到的内容(如果有分组则传入`index`参数获取第几个分组)，`span()`可以获取匹配的范围

  无匹配结果返回`None`

- `re.search()`

  匹配第一个符合的内容，返回匹配对象

  ```python
  re.search(pattern, string, flag)
  ```

  > 为了保证匹配结果，可以尽量使用`search()`代替`match()`
  
- `re.findall()`

  返回匹配的列表内容

  ```python
  re.findall(pattern, string, flag)
  ```

- `re.sub()`

  替换匹配到的内容

  ```python
  re.sub(pattern, replace, string, max, flag)
  ```

  - `max`：最多替换次数，默认为全部
  
- `re.split()`

  用正则表达式切割字符串，返回为列表

  ```python
  re.split(pattern, string, flag)
  ```

- `re.compile()`

  生成正则表达式对象

  ```python
  regex = re.compile(pattern, flag)
  ```

  - `pattern`：正则表达式
  - `flags`：功能标志位，拓展正则表达式

- `regex.findall()`

  用正则表达式对象匹配字符串内容

  ```python
  regex.findall(string, pos, endpos)
  ```

  - `pos`，`endpos`：起始和结束的位置

- `re.split()`

  用正则表达式切割字符串，返回为列表

  ```python
  re.split(pattern, string, flag)
  ```



## bs4解析(python独有)

### 原理

bs4会将html文档转换为一个树形结构，每个节点对应一个标签或内容，对应一个python对象，可以分为4类：

- `tag`：所有html标签可以视为tag对象
- `navigableString`：字符串类，标签中的文本内容
- `beautifulSoup`：一个html文档的全部内容，根节点，特殊的tag对象
- `comment`：html中的注释和特殊字符串，特殊的`navigableString`

### 步骤

- 实例化一个`BeautifulSoup`对象，并将页面源码加载到该对象中
- 通过调用`BeautifulSoup`对象的相关属性和方法进行数据提取

### 环境安装

```python
pip install bs4
pip install lxml
```

> lxml为bs4支持的解析器之一，支持解析HTML和XML，速度快，容错率高，推荐使用

### BeautifulSoup

- 实例化

  - ```python
    soup = BeautifulSoup(fp, 'lxml')
    ```

    将本地的html文件加载到该对象中，使用lxml解析器

    `fp`：文件描述符

  - ```python
    soup = BeautifulSoup(response.text, 'lxml')
    ```

    将互联网获取的页面源码加载到该对象

  > bs4对不标准的HTML字符串，可以在初始化时自动更正

- 方法和属性

  > 除text/string等特殊外，其它返回仍为`bs4.element.Tag `类型，可以继续调用方法解析

  - `soup.tagName`

    获取**第一次**出现的tagName

  - 子节点

    - `soup.tagName.contents`
  
      `soup.tagName.children`
  
      获取第一个p节点的内容与直接子节点的列表
  
    - `soup.tagName.descendants`
  
      获取所有子孙节点及内容的列表
  
  - 父节点
  
    - `soup.tagName.parent`
  
      获取直接父节点及器内容(本节点同样包含一次)
  
    - `soup.tagName.parents`
  
      获取所有的祖先节点
  
  - 兄弟节点
  
    - `soup.tagName.next_sibling`
  
      下一个兄弟节点
  
    - `soup.tagName.previous_sibling`
  
      上一个兄弟节点
  
    - `soup.tagName.next_siblings`
  
      之前的所有兄弟节点
  
    - `soup.tagName.previous_siblings`
  
      之后的所有兄弟节点
  
  - `soup.find()`
  
    获取**第一次**出现的tagName
  
    ```python
    soup.find('tagName') # 等同于soup.tagName
    soup.find('tagName', attribute='') # 获取第一次具有固定属性的tagName
    ```
  
    - `attribute`：标签的属性，`class`需要写为`class_`
  
    > find有许多类似方法，例如`find_parents`，`find_next_siblings`等
  
  - `soup.find_all()`
  
    获取所有tagName的标签内容列表
  
    ```python
    soup.findall(name='tagName', attrs={'':''}, text) 
    ```
  
    - `name`：使用名称查找
  
    - `attrs`：使用属性查找，常用属性如`id`，`class`可以直接传入
  
      > `class`需要写为`class_`
  
    - `text`：使用文本查找节点，可以是字符串，也可以是正则表达式对象
  
  - `soup.select()`
  
    返回符合选择器的标签列表
  
    ```python
    soup.select('selector') # 标签选择器
    ```
  
    - `selector`：某种选择器
  
    ```python
    .p1	# 类选择器，html中对应<p class="p1">
    #id	# id选择器，html中对应<p id="text">
    p	# 标签选择器，html中对应<p>
    p,a,li	# 分组选择器，一次选取多个标签
    duv ul li	# 后代选择器，选择包含在之前标签中的最后一个标签
    [name="pra1"]	# 属性选择器，如name，对应<p name="pra1">
    *	# 通用选择器
    p+a	# 兄弟选择器，选择同级的标签
    div>p	# 选择直属于div的p
    ```
  
  - `soup.tagName.text/string/get_text()`
  
    获取标签之间的文本数据
  
    > `text/get_text()`获取标签中所有的文本内容(包括子标签)，`string`只能获取直接属于该标签的文本
  
  - `soup.tagName['attribute']`
  
    `soup.tagName.attrs['attribute']`
  
    获取对应标签的属性内容，可以直接或使用attrs获取，单独使用attrs返回所有属性的字典

## `pyquery`

`pyquery`是利用CSS选择器对网页进行解析的工具

### 安装

```shell
pip install pyquery
```

### 使用

- 实例化

  初始化`pyquery`时，也需要传入HTML文本来实例化一个`PyQuery`对象

  - 字符串初始化

    ```python
    from pyquery import PyQuery
    doc = PyQuery(html)
    ```

  - url初始化

    传入网页的url后，`pyquery`会先对url进行请求，用得到的HTML内容进行初始化

    ```python
    from pyquery import PyQuery
    doc = PyQuery(url='http://cuiqingcai.com')
    ```

  - 文件初始化

    ```python
    from pyquery import PyQuery
    doc = PyQuery(filename='demo.html')
    ```

- 使用选择器

  - 基本用法

    ```python
    doc = doc('#container .list li')
    ```

    传入一个CSS选择器，选择出所有符合条件的节点，返回仍为`PyQuery`对象

  - 查找节点

    - 子孙节点

      ```python
      doc.find(selector)
      ```

      查找出所有符合`selector`的所有子孙节点

    - 直系子节点

      ```python
      doc.children(selector)
      ```

      查找出所有符合`selector`的所有子节点

    - 父节点

      ```python
      doc.parent(selector)
      ```

      查找节点的直接父节点

    - 祖先节点

      ```python
      doc.parents(selector)
      ```

    - 兄弟节点

      ```python
      doc.siblings(selector)
      ```

  - 遍历

    `pyquery`的结果可能是多个节点，但返回不为列表

    对单个节点，可以直接输出或转为字符串

    对于多节点，可以使用`items`方法获取生成器，并进行遍历

    ```python
    lis = doc('li').items()
    for li in lis:
        pass
    ```

  - 获取信息

    - 获取属性

      调用`attr()`方法获取属性

      ```python
      doc.attr('href')
      doc.attr.href	# 两者结果一致
      ```

      > 多个节点调用`attr`只会返回第一个

    - 获取文本

      调用`text()`或`html()`方法获取文本，`text()`会忽略所有html标签(子节点)，`html()`会返回节点内部的html文本

      ```python
      doc.text()	# text会返回所有节点的文本内容的字符串，用空格分割
      doc.html()	# html只会返回第一个
      ```

  - 节点操作

    - 添加，删除当前节点`class`属性

      ```python
      doc.addClass(className)
      doc.removeClass(className)
      ```

    - `attr`，`text`，`html`

      ```python
      doc.attr(tagName, content)	# 改变属性值
      doc.text()	# 改变内容
      doc.html()	# 改变html文本
      ```

    - `remove()`

​	更多节点操作可以参考[http://pyquery.readthedocs.io/en/latest/api.html](http://pyquery.readthedocs.io/en/latest/api.html)

- 伪类选择器

  CSS选择器支持多种伪类选择器，例如第一个节点，奇偶数节点等

  可以参考[http://www.w3school.com.cn/css/index.asp](http://www.w3school.com.cn/css/index.asp)

## xpath

即[XML路径语言](https://www.w3.org/TR/xpath/)，是最常用且最便捷高效的一种解析方式，通用性最强

### 常用规则

- 根据层级关系，例如，html下的`head`下的`title`表示为`/html/head/title`

  - `/`：开始表示从根目录开始定位，后续表示一个层级
  - `//`：开始表示从任意位置开始定位，后续表示多个层级
  - `[@attrName="attrValue"]`：属性定位，跟在标签后，表示指定属性的标签
  - `[n]`：索引定位，跟在标签后，定位到第n个标签(从1开始)
  - `.`：表示当前标签
  - `..`：当前标签的父标签
  - 取文本：
    - `/text()`：取得直属于标签的文本
    - `//text()`：取得所有属于标签的文本，包括子标签
  - 取属性：
    - `/@attrName`：取得标签的特定属性值
  - 关系运算法：主要使用逻辑或` |`

>  xpath不可以匹配`tbody`标签
>
>  xpath表达式可以在浏览器中直接生成

### 步骤

- 实例化一个`etree`的对象，且需要将页面源码加载到该对象中
- 调用`etree`对象中的`xpath`方法结合xpath表达式实现标签的定位和内容的捕获

### 环境安装

```python
pip install lxml
```

### etree

- 实例化对象

  ```python
  from lxml import etree
  
  etree.parse(filePath)	# 输入文件路径，返回一个etree对象
  etree.HTML(page_text)	# 输入html文本，返回一个etree对象
  ```

  > 在etree对象初始化时，不标准的HTML会自动更正

- xpath表达式

  ```python
  etree.xpath('xpath表达式')	# 返回符合表达式的列表
  lable.xpath('xpath表达式')	# 返回的标签对象也有xpath表达式方法，可以使用./来表示当前label下
  ```


>  xpath返回都为列表，使用需要索引

- 属性多值

  ```python
  tree.xpath('//li[contains(@class, "li")]/a/text()')	# 如果属性有多个值，则可以使用contains()选取包含属性的
  ```

- 多属性

  ```python
  tree.xpath('//li[contains(@class, "li") and @name="item"]/a/text()')	#多属性匹配使用and来连接
  ```

> 更多运算符：[http://www.w3school.com.cn/xpath/xpath_operators.asp](http://www.w3school.com.cn/xpath/xpath_operators.asp)

- 索引选择

  除了能使用索引定位`[n]`(从1开始)，还能使用在`[]`中使用`last()`，`position()<n`等，具体可以参考[http://www.w3school.com.cn/xpath/xpath_functions.asp](http://www.w3school.com.cn/xpath/xpath_functions.asp)

- 节点轴选择

  Xpath也提供了节点轴选择

  ```python
  tree.xpath('//li[1]/ancestor::*')
  tree.xpath('//li[1]/attribute::*')
  ```

  `::`前加`ancestor`(获取祖先节点)，`attribute`(获取所有属性)，`child`(获取子节点)，`descentdant`(获取子孙节点)，`following`(当前节点之后的所有节点)，`following-sibling`(当前节点之后所有同级节点)等轴名

  Xpath轴可以参考[http://www.w3school.com.cn/xpath/xpath_axes.asp]([http://www.w3school.com.cn/xpath/xpath_axes.asp)



# 数据存储

数据存储有多种方式，最简单的是文件存储，也可以保存到数据库中

## 文件存储

### txt文件

txt文件存储操作简单，兼容性好，但不利于检索

```python
# 第一种
f = open(filename, 'w', encoding)
f.write(content)
f.close()

# 第二种
with open(filename, 'w', encoding) as f:
    f.write(content)
```

- 打开方式
  - r：只读
  - b：以二进制形式打开
  - w：只写，如果文件存在则覆盖
  - a：追加打开
  - \+：跟在r或w后，以读写方式打开

### JSON文件

JSON，即JavaScript对象标记，是一种轻量级的数据交换格式

- 对象和数组

  在 JavaScript 语言中，一切都是对象。任何支持的类型都可以通过JSON表示，对象和数组是两种特殊且常用的类型

  - 对象：在 JavaScript 中是使用花括号` {}` 包裹起来的内容，数据结构为`{key1: value1, key2: value2, ...}`，键名可以是整数和字符串，值可以是任意类型
  - 数组：数组在 JavaScript 中是方括号` [] `包裹起来的内容，数据结构为`[value1, value2...]`

- JSON库

  - 读取

    ```python
    data = json.loads(str)	# 将字符串转为JSON对象
    
    with open('data.json', 'r') as file:
        str = file.read()
        data = json.loads(str)
    ```

    > 注意字符串中JSON需要使用双引号，否则`loads`方法会报错

  - 访问

    ```python
    data[0]['name']
    data[0].get('name')	# 推荐，若键值不存在，则返回None
    ```

  - 输出

    ```python
    json.dumps(data, indent=2, ensure_ascii=False)	
    # 将JSON对象转换为字符串
    # indent表示缩进，可以保留JSON格式
    # ensure_ascii默认为True，但无法输出中文
    
    with open('data.json', 'w') as file:
        file.write(json.dumps(data))
    ```

### CSV文件

CSV文件，即逗号分割值文件，以纯文本形式存储表格数据，记录间由某种换行符分割，记录的字段间使用分割符(一般为逗号或制表符)分割

- 写入

  - 写入行

    ```python
    import csv
    
    with open('data.csv', 'w', encoding='utf-8') as csvfile:
        writer = csv.writer(csvfile, delimiter= ' ')
        writer.writerow(['id', 'name', 'age'])
    ```

    使用`writer`类创建对象，调用`writerow()`写入每行数据，`delimiter`指定分割符

  - 写入多行

    writerows`方法可以写入多行，要传入二维列表

  - 写入字典

    ```python
    import csv
    
    with open('data.csv', 'w', encoding='utf-8') as csvfile:
        fieldnames = ['id', 'name', 'age']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerow({'id': '10001', 'name': 'Mike', 'age': 20})
    ```

    首先使用`writeheader()`方法写入表头，然后使用`writerow()`方法写入数据，需要与表头一致

    > 也可以使用pandas库的`DataFrame`对象的`to_csv()`方法写入csv文件

- 读取

  ```python
  with open('data.csv', 'r', encoding='utf-8') as csvfile:  
      reader = csv.reader(csvfile)  
      for row in reader:  
         	pass 
  ```

  读取时使用`reader`类型的对象加载csv文件

  > 或者使用pandas的`read_csv()`方法

## 关系型数据库存储

关系型数据库是基于关系模型的数据库，通过二维表来保存，每一列是一个字段，每一行是一条记录，多个表组成数据库

### PyMysql使用

- 连接数据库

  ```python
  import pymysql
  
  db = pymysql.connect(host='domain/ip', user='user', password='pass', port=3306)
  cursor = db.cursor()
  cursor.execute('')
  data = cursor.fetchone()	# data = cursor.fetchall()
  
  db.close()
  ```

  连接mysql需要声明一个`MySql`对象，传入ip，用户名，密码，端口

  连接成功后，调用`cursor()`方法获取操作mysql的游标，使用`execute()`方法指定SQL语句，如果有返回数据，则使用`fetchone()`方法获取第一条，`fetchall()`获取所有数据(也可以使用`wihle`和`fetchone()`代替)

- 插入数据

  ```python
  try:
      cursor.execute(sql)
      db.commit()
  except:
      db.rollback()
  ```

  `execute()`方法可以将sql与param元组进行格式化拼接，`commit()`方法用于实现数据的写入，数据插入，更新，删除都需要调用该方法生效，`rollback()`执行数据回滚，一般用于异常处理

  根据字典动态构造插入的例子，则可以不修改SQL语句，只需修改字典

  ```python
  data = {
      'id': '20120001',
      'name': 'Bob',
      'age': 20
  }
  table = 'students'
  keys = ', '.join(data.keys())
  values = ', '.join(['% s'] * len(data))
  sql = 'INSERT INTO {table}({keys}) VALUES ({values})'.format(table=table, keys=keys, values=values)
  try:
     if cursor.execute(sql, tuple(data.values())):
         print('Successful')
         db.commit()
  except:
      print('Failed')
      db.rollback()
  db.close()
  ```

  > 异常时回滚涉及事务的问题，事务机制可以确保数据一致性
  >
  > | 属　　性              | 解　　释                                                     |
  > | --------------------- | ------------------------------------------------------------ |
  > | 原子性（atomicity）   | 事务是一个不可分割的工作单位，事务中包括的诸操作要么都做，要么都不做 |
  > | 一致性（consistency） | 事务必须使数据库从一个一致性状态变到另一个一致性状态。一致性与原子性是密切相关的 |
  > | 隔离性（isolation）   | 一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰 |
  > | 持久性（durability）  | 持续性也称永久性（permanence），指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响 |

- 更新数据

  ```python
  sql = 'UPDATE students SET age = % s WHERE name = % s'
  ```

  根据字典灵活传值

  ```python
  data = {
      'id': '20120001',
      'name': 'Bob',
      'age': 21
  }
  
  table = 'students'
  keys = ', '.join(data.keys())
  values = ', '.join(['% s'] * len(data))
  
  sql = 'INSERT INTO {table}({keys}) VALUES ({values}) ON DUPLICATE KEY UPDATE'.format(table=table, keys=keys, values=values)
  update = ','.join(["{key} = % s".format(key=key) for key in data])
  sql += update
  try:
      if cursor.execute(sql, tuple(data.values())*2):
          print('Successful')
          db.commit()
  except:
      print('Failed')
      db.rollback()
  db.close()
  ```

  实际上是插入语句，但加入`ON DUPLICATE KEY UPDATE`，表示如果主键存在，则执行更新操作，若果不存在则插入

- 删除数据

  ```python
  sql = 'DELETE FROM  {table} WHERE {condition}'.format(table=table, condition=condition)
  ```

- 查询数据

  ```python
  sql = 'SELECT * FROM students WHERE age >= 20'
  ```

## 非关系数据库

非关系型数据库，基于键值对，不需要经过SQL层的解析，数据没有耦合，性能很高，对于爬虫数据存储，某些字段可能提取失败而缺失，而且数据随时可能调整。使用非关系型数据库，可以避免提前建表，进行序列化操作等

### 分类

- 键值存储数据库：Redis，Voldemort等
- 列存储数据库：Cassandra等
- 文档型数据库：MongoDB等
- 图形数据库：Neo4J等

### MongoDB

MongoDB是基于分布式文件存储的非关系数据库系统，其内容存储形式类似JSON对象，且字段值可以包含其它文档，数组及文档数组

- 连接MongoDB

  ```python
  import pymongo
  client = pymongo.MongoClient(host='localhost', port=27017)
  ```

  创建`Mongoclient`对象需要传入MongoDB的IP及端口

- 指定数据库

  ```python
  db = client.name
  db = client['name']
  ```

  使用两种形式都可以得到对应的数据库对象

- 指定集合

  MongoDB每个数据库又包含许多集合，类似关系型数据库中的表

  ```python
  collection = db.name
  collection = db['name']
  ```

- 插入数据

  ```python
  result = collection.insert(dict)	# 官方不再推荐使用
  result = collection.insert_one(dict)
  result = collection.insert_many(dict)
  ```

  插入字典类型的数据，会自动生成 ObjectId 类型的_id 属性。

  `insert()`方法可以插入任意条数据，返回\_id或\_id的列表，`insert_one()`和`insert_many()`返回的是`pymongo.results.InsertOneResult`和`pymongo.results.InsertManyResult`对象，需要调用`inserted_id`和`inserted_ids`属性获取\_id

- 查询数据

  ```python
  result = collection.find_one(dict)
  result = collection.find(dict)
  ```

  如果要用\_id查询，需要使用bson库中的ObjectId

  ```python
  from bson.objectid import ObjectId
  
  result = collection.find_one({'_id': ObjectId('593278c115c2602667ec6bae')})
  ```

  `find_one`的结果是字典类型，如果结果不存在，返回`None`

  `find`的结果是`Cursor`类型，相当于一个生成器，可以遍历获取结果

  查询时可以使用比较符号和功能符号

  | 符　　号 | 含　　义   | 示　　例                    |
  | -------- | ---------- | --------------------------- |
  | $lt      | 小于       | {'age': {'$lt': 20}}        |
  | $gt      | 大于       | {'age': {'$gt': 20}}        |
  | $lte     | 小于等于   | {'age': {'$lte': 20}}       |
  | $gte     | 大于等于   | {'age': {'$gte': 20}}       |
  | $ne      | 不等于     | {'age': {'$ne': 20}}        |
  | $in      | 在范围内   | {'age': {'$in': [20, 23]}}  |
  | $nin     | 不在范围内 | {'age': {'$nin': [20, 23]}} |

  | 符　　号 | 含　　义       | 示　　例                                          | 示例含义                          |
  | -------- | -------------- | ------------------------------------------------- | --------------------------------- |
  | $regex   | 匹配正则表达式 | {'name': {'$regex': '^M.*'}}                      | name 以 M 开头                    |
  | $exists  | 属性是否存在   | {'name': {'$exists': True}}                       | name 属性存在                     |
  | $type    | 类型判断       | {'age': {'$type': 'int'}}                         | age 的类型为 int                  |
  | $mod     | 数字模操作     | {'age': {'$mod': [5, 0]}}                         | 年龄模 5 余 0                     |
  | $text    | 文本查询       | {'$text': {'$search': 'Mike'}}                    | text 类型的属性中包含 Mike 字符串 |
  | $where   | 高级条件查询   | {'$where': 'obj.fans_count == obj.follows_count'} | 自身粉丝数等于关注数              |

> 可以参考[https://docs.mongodb.com/manual/reference/operator/query/](https://docs.mongodb.com/manual/reference/operator/query/)

- 计数

  查询结果由多少条数据，可以使用`count()`方法

  ```python
  count = collection.find().count()
  ```

- 排序

  排序时，调用`sort()`方法，并传入排序字段和升降序标志即可

  ```python
  results = collection.find().sort('name', pymongo.ASCENDING)
  ```

  > `pymongo.ASCENDING`为升序，`pymongo.DESCENDING`为降序

- 偏移

  可以使用`skip(n)`进行跳转偏移

  ```python
  results = collection.find().sort('name', pymongo.ASCENDING).skip(2)
  ```

  > 在数据量十分巨大的时候，最好不要使用大的偏移量，可能导致内存溢出

- 指定结果的个数

  ```python
  results = collection.find().sort('name', pymongo.ASCENDING).skip(2).limit(2)
  ```

- 更新

  可以使用`update()`方法，指定更新条件和更新后的数据，返回字典中`nModified`代表修改的数据条数

  ```python
  condition = {'name': 'Kevin'}
  student = collection.find_one(condition)
  student['age'] = 25
  result = collection.update(condition, student)	# 对name为Kevin的数据将age变为25
  print(result)
  ```

  > 官方也推荐使用`update_one()`和`update_many()`，使用条件更加严格，第二个参数要使用$类型操作符为字典的键名

- 删除

  直接调用`remove()`方法，并传入条件，符合条件的数据都会被删除

> 更多可以参考[http://api.mongodb.com/python/current/api/pymongo/collection.html](http://api.mongodb.com/python/current/api/pymongo/collection.html)和[http://api.mongodb.com/python/current/api/pymongo/](http://api.mongodb.com/python/current/api/pymongo/)

### Redis

- 安装

  ```shell
  pip install redis-py
  ```

- `Redis`和`StrictRedis`

  redis-py提供了`Redis`和`StrictRedis`两个类对redis实行操作，`StrictRedis`实现了绝大部分Redis命令，参数也一一对应，`Redis`是`StrictRedis`的子类，主要是用于向后兼容旧版本库中的几个方法，推荐使用`StrictRedis`类

- 连接Redis

  ```python
  from redis import StrictRedis  
  
  redis = StrictRedis(host='localhost', port=6379, db=0, password='foobared')  
  ```

  > 也可以先构建`ConnectionPool`类，再将其作为参数传入`StrictRedis`类，实际`StrictRedis`类实例时也是调用`ConnectionPool`类
  >
  > `ConnectionPool`类可以使用万张的URL创建
  >
  > ```python
  > redis://[:password]@host:port/db  # 使用TCP
  > rediss://[:password]@host:port/db  # 使用TCP+SSL
  > unix://[:password]@/path/to/socket.sock?db=db	# 使用Redis UNIX socket
  > ```

- 键操作

  | 方　　法           | 作　　用                                      | 参数说明                   | 示　　例                         | 示例说明                        | 示例结果  |
  | ------------------ | --------------------------------------------- | -------------------------- | -------------------------------- | ------------------------------- | --------- |
  | exists(name)       | 判断一个键是否存在                            | name：键名                 | redis.exists('name')             | 是否存在 name 这个键            | True      |
  | delete(name)       | 删除一个键                                    | name：键名                 | redis.delete('name')             | 删除 name 这个键                | 1         |
  | type(name)         | 判断键类型                                    | name：键名                 | redis.type('name')               | 判断 name 这个键类型            | b'string' |
  | keys(pattern)      | 获取所有符合规则的键                          | pattern：匹配规则          | redis.keys('n*')                 | 获取所有以 n 开头的键           | [b'name'] |
  | randomkey()        | 获取随机的一个键                              |                            | randomkey()                      | 获取随机的一个键                | b'name'   |
  | rename(src, dst)   | 重命名键                                      | src：原键名；dst：新键名   | redis.rename('name', 'nickname') | 将 name 重命名为 nickname       | True      |
  | dbsize()           | 获取当前数据库中键的数目                      |                            | dbsize()                         | 获取当前数据库中键的数目        | 100       |
  | expire(name, time) | 设定键的过期时间，单位为秒                    | name：键名；time：秒数     | redis.expire('name', 2)          | 将 name 键的过期时间设置为 2 秒 | True      |
  | ttl(name)          | 获取键的过期时间，单位为秒，1 表示永久不过期 | name：键名                 | redis.ttl('name')                | 获取 name 这个键的过期时间      | 1        |
  | move(name, db)     | 将键移动到其他数据库                          | name：键名；db：数据库代号 | move('name', 2)                  | 将 name 移动到 2 号数据库       | True      |
  | flushdb()          | 删除当前选择数据库中的所有键                  |                            | flushdb()                        | 删除当前选择数据库中的所有键    | True      |
  | flushall()         | 删除所有数据库中的所有键                      |                            | flushall()                       | 删除所有数据库中的所有键        | True      |

- 字符串操作

  | 方　　法                      | 作　　用                                                     | 参数说明                                                     | 示　　例                                                     | 示例说明                                                  | 示例结果                                      |
  | ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------------------- | --------------------------------------------- |
  | set(name, value)              | 给数据库中键名为 name 的 string 赋予值 value                 | n ame：键名；value：值                                       | redis.set('name', 'Bob')                                     | 给 name 这个键的 value 赋值为 Bob                         | True                                          |
  | get(name)                     | 返回数据库中键名为 name 的 string 的 value                   | name：键名                                                   | redis.get('name')                                            | 返回 name 这个键的 value                                  | b'Bob'                                        |
  | getset(name, value)           | 给数据库中键名为 name 的 string 赋予值 value 并返回上次的 value | name：键名；value：新值                                      | redis.getset('name', 'Mike')                                 | 赋值 name 为 Mike 并得到上次的 value                      | b'Bob'                                        |
  | mget(keys, *args)             | 返回多个键对应的 value 组成的列表                            | keys：键名序列                                               | redis.mget(['name', 'nickname'])                             | 返回 name 和 nickname 的 value                            | [b'Mike', b'Miker']                           |
  | setnx(name, value)            | 如果不存在这个键值对，则更新 value，否则不变                 | name：键名                                                   | redis.setnx('newname', 'James')                              | 如果 newname 这个键不存在，则设置值为 James               | 第一次运行结果是 True，第二次运行结果是 False |
  | setex(name, time, value)      | 设置可以对应的值为 string 类型的 value，并指定此键值对应的有效期 | n ame：键名；time：有效期；value：值                         | redis.setex('name', 1, 'James')                              | 将 name 这个键的值设为 James，有效期为 1 秒               | True                                          |
  | setrange(name, offset, value) | 设置指定键的 value 值的子字符串                              | name：键名；offset：偏移量；value：值                        | redis.set('name', 'Hello') redis.setrange ('name', 6, 'World') | 设置 name 为 Hello 字符串，并在 index 为 6 的位置补 World | 11，修改后的字符串长度                        |
  | mset(mapping)                 | 批量赋值                                                     | mapping：字典或关键字参数                                    | redis.mset({'name1': 'Durant', 'name2': 'James'})            | 将 name1 设为 Durant，name2 设为 James                    | True                                          |
  | msetnx(mapping)               | 键均不存在时才批量赋值                                       | mapping：字典或关键字参数                                    | redis.msetnx({'name3': 'Smith', 'name4': 'Curry'})           | 在 name3 和 name4 均不存在的情况下才设置二者值            | True                                          |
  | incr(name, amount=1)          | 键名为 name 的 value 增值操作，默认为 1，键不存在则被创建并设为 amount | name：键名；amount：增长的值                                 | redis.incr('age', 1)                                         | age 对应的值增 1，若不存在，则会创建并设置为 1            | 1，即修改后的值                               |
  | decr(name, amount=1)          | 键名为 name 的 value 减值操作，默认为 1，键不存在则被创建并将 value 设置为 - amount | name：键名；amount：减少的值                                 | redis.decr('age', 1)                                         | age 对应的值减 1，若不存在，则会创建并设置为1            | 1，即修改后的值                              |
  | append(key, value)            | 键名为 key 的 string 的值附加 value                          | key：键名                                                    | redis.append('nickname', 'OK')                               | 向键名为 nickname 的值后追加 OK                           | 13，即修改后的字符串长度                      |
  | substr(name, start, end=-1)   | 返回键名为 name 的 string 的子字符串                         | name：键名；start：起始索引；end：终止索引，默认为1，表示截取到末尾 | redis.substr('name', 1, 4)                                   | 返回键名为 name 的值的字符串，截取索引为 1~4 的字符       | b'ello'                                       |
  | getrange(key, start, end)     | 获取键的 value 值从 start 到 end 的子字符串                  | key：键名；start：起始索引；end：终止索引                    | redis.getrange('name', 1, 4)                                 | 返回键名为 name 的值的字符串，截取索引为 1~4 的字符       | b'ello'                                       |

- 列表操作

  Redis 还提供了列表存储，列表内的元素可以重复，而且可以从两端存储

  | 方　　法                 | 作　　用                                                     | 参数说明                                            | 示　　例                         | 示例说明                                                     | 示例结果           |
  | ------------------------ | ------------------------------------------------------------ | --------------------------------------------------- | -------------------------------- | ------------------------------------------------------------ | ------------------ |
  | rpush(name, *values)     | 在键名为 name 的列表末尾添加值为 value 的元素，可以传多个    | name：键名；values：值                              | redis.rpush('list', 1, 2, 3)     | 向键名为 list 的列表尾添加 1、2、3                           | 3，列表大小        |
  | lpush(name, *values)     | 在键名为 name 的列表头添加值为 value 的元素，可以传多个      | name：键名；values：值                              | redis.lpush('list', 0)           | 向键名为 list 的列表头部添加 0                               | 4，列表大小        |
  | llen(name)               | 返回键名为 name 的列表的长度                                 | name：键名                                          | redis.llen('list')               | 返回键名为 list 的列表的长度                                 | 4                  |
  | lrange(name, start, end) | 返回键名为 name 的列表中 start 至 end 之间的元素             | name：键名；start：起始索引；end：终止索引          | redis.lrange('list', 1, 3)       | 返回起始索引为 1 终止索引为 3 的索引范围对应的列表           | [b'3', b'2', b'1'] |
  | ltrim(name, start, end)  | 截取键名为 name 的列表，保留索引为 start 到 end 的内容       | name：键名；start：起始索引；end：终止索引          | ltrim('list', 1, 3)              | 保留键名为 list 的索引为 1 到 3 的元素                       | True               |
  | lindex(name, index)      | 返回键名为 name 的列表中 index 位置的元素                    | name：键名；index：索引                             | redis.lindex('list', 1)          | 返回键名为 list 的列表索引为 1 的元素                        | b'2'               |
  | lset(name, index, value) | 给键名为 name 的列表中 index 位置的元素赋值，越界则报错      | name：键名；index：索引位置；value：值              | redis.lset('list', 1, 5)         | 将键名为 list 的列表中索引为 1 的位置赋值为 5                | True               |
  | lrem(name, count, value) | 删除 count 个键的列表中值为 value 的元素                     | name：键名；count：删除个数；value：值              | redis.lrem('list', 2, 3)         | 将键名为 list 的列表删除两个 3                               | 1，即删除的个数    |
  | lpop(name)               | 返回并删除键名为 name 的列表中的首元素                       | name：键名                                          | redis.lpop('list')               | 返回并删除名为 list 的列表中的第一个元素                     | b'5'               |
  | rpop(name)               | 返回并删除键名为 name 的列表中的尾元素                       | name：键名                                          | redis.rpop('list')               | 返回并删除名为 list 的列表中的最后一个元素                   | b'2'               |
  | blpop(keys, timeout=0)   | 返回并删除名称在 keys 中的 list 中的首个元素，如果列表为空，则会一直阻塞等待 | keys：键名序列；timeout：超时等待时间，0 为一直等待 | redis.blpop('list')              | 返回并删除键名为 list 的列表中的第一个元素                   | [b'5']             |
  | brpop(keys, timeout=0)   | 返回并删除键名为 name 的列表中的尾元素，如果 list 为空，则会一直阻塞等待 | keys：键名序列；timeout：超时等待时间，0 为一直等待 | redis.brpop('list')              | 返回并删除名为 list 的列表中的最后一个元素                   | [b'2']             |
  | rpoplpush(src, dst)      | 返回并删除名称为 src 的列表的尾元素，并将该元素添加到名称为 dst 的列表头部 | src：源列表的键；dst：目标列表的 key                | redis.rpoplpush('list', 'list2') | 将键名为 list 的列表尾元素删除并将其添加到键名为 list2 的列表头部，然后返回 | b'2'               |

- 集合操作

  Redis 还提供了集合存储，集合中的元素都是不重复的

  | 方　　法                       | 作　　用                                                 | 参数说明                                  | 示　　例                                        | 示例说明                                                     | 示例结果                     |
  | ------------------------------ | -------------------------------------------------------- | ----------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ | ---------------------------- |
  | sadd(name, *values)            | 向键名为 name 的集合中添加元素                           | name：键名；values：值，可为多个          | redis.sadd('tags', 'Book', 'Tea', 'Coffee')     | 向键名为 tags 的集合中添加 Book、Tea 和 Coffee 这 3 个内容   | 3，即插入的数据个数          |
  | srem(name, *values)            | 从键名为 name 的集合中删除元素                           | name：键名；values：值，可为多个          | redis.srem('tags', 'Book')                      | 从键名为 tags 的集合中删除 Book                              | 1，即删除的数据个数          |
  | spop(name)                     | 随机返回并删除键名为 name 的集合中的一个元素             | name：键名                                | redis.spop('tags')                              | 从键名为 tags 的集合中随机删除并返回该元素                   | b'Tea'                       |
  | smove(src, dst, value)         | 从 src 对应的集合中移除元素并将其添加到 dst 对应的集合中 | src：源集合；dst：目标集合；value：元素值 | redis.smove('tags', 'tags2', 'Coffee')          | 从键名为 tags 的集合中删除元素 Coffee 并将其添加到键为 tags2 的集合 | True                         |
  | scard(name)                    | 返回键名为 name 的集合的元素个数                         | name：键名                                | redis.scard('tags')                             | 获取键名为 tags 的集合中的元素个数                           | 3                            |
  | sismember(name, value)         | 测试 member 是否是键名为 name 的集合的元素               | name：键值                                | redis.sismember('tags', 'Book')                 | 判断 Book 是否是键名为 tags 的集合元素                       | True                         |
  | sinter(keys, *args)            | 返回所有给定键的集合的交集                               | keys：键名序列                            | redis.sinter(['tags', 'tags2'])                 | 返回键名为 tags 的集合和键名为 tags2 的集合的交集            | {b'Coffee'}                  |
  | sinterstore(dest, keys, *args) | 求交集并将交集保存到 dest 的集合                         | dest：结果集合；keys：键名序列            | redis.sinterstore ('inttag', ['tags', 'tags2']) | 求键名为 tags 的集合和键名为 tags2 的集合的交集并将其保存为 inttag | 1                            |
  | sunion(keys, *args)            | 返回所有给定键的集合的并集                               | keys：键名序列                            | redis.sunion(['tags', 'tags2'])                 | 返回键名为 tags 的集合和键名为 tags2 的集合的并集            | {b'Coffee', b'Book', b'Pen'} |
  | sunionstore(dest, keys, *args) | 求并集并将并集保存到 dest 的集合                         | dest：结果集合；keys：键名序列            | redis.sunionstore ('inttag', ['tags', 'tags2']) | 求键名为 tags 的集合和键名为 tags2 的集合的并集并将其保存为 inttag | 3                            |
  | sdiff(keys, *args)             | 返回所有给定键的集合的差集                               | keys：键名序列                            | redis.sdiff(['tags', 'tags2'])                  | 返回键名为 tags 的集合和键名为 tags2 的集合的差集            | {b'Book', b'Pen'}            |
  | sdiffstore(dest, keys, *args)  | 求差集并将差集保存到 dest 集合                           | dest：结果集合；keys：键名序列            | redis.sdiffstore ('inttag', ['tags', 'tags2'])  | 求键名为 tags 的集合和键名为 tags2 的集合的差集并将其保存为 inttag | 3                            |
  | smembers(name)                 | 返回键名为 name 的集合的所有元素                         | name：键名                                | redis.smembers('tags')                          | 返回键名为 tags 的集合的所有元素                             | {b'Pen', b'Book', b'Coffee'} |
  | srandmember(name)              | 随机返回键名为 name 的集合中的一个元素，但不删除元素     | name：键值                                | redis.srandmember('tags')                       | 随机返回键名为 tags 的集合中的一个元素                       | Srandmember (name)           |

- 有序集合操作

  有序集合比集合多了一个分数字段，利用它可以对集合中的数据进行排序

  | 方　　法                                                     | 作　　用                                                     | 参数说明                                                     | 示　　例                                    | 示例说明                                                     | 示例结果                            |
  | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------- | ------------------------------------------------------------ | ----------------------------------- |
  | zadd(name, *args, **kwargs)                                  | 向键名为 name 的 zset 中添加元素 member，score 用于排序。如果该元素存在，则更新其顺序 | name：键名；args：可变参数                                   | redis.zadd('grade', 100, 'Bob', 98, 'Mike') | 向键名为 grade 的 zset 中添加 Bob（其 score 为 100），并添加 Mike（其 score 为 98） | 2，即添加的元素个数                 |
  | zrem(name, *values)                                          | 删除键名为 name 的 zset 中的元素                             | name：键名；values：元素                                     | redis.zrem('grade', 'Mike')                 | 从键名为 grade 的 zset 中删除 Mike                           | 1，即删除的元素个数                 |
  | zincrby(name, value, amount=1)                               | 如果在键名为 name 的 zset 中已经存在元素 value，则将该元素的 score 增加 amount；否则向该集合中添加该元素，其 score 的值为 amount | name：键名；value：元素；amount：增长的 score 值             | redis.zincrby('grade', 'Bob', -2)           | 键名为 grade 的 zset 中 Bob 的 score 减 2                    | 98.0，即修改后的值                  |
  | zrank(name, value)                                           | 返回键名为 name 的 zset 中元素的排名，按 score 从小到大排序，即名次 | name：键名；value：元素值                                    | redis.zrank('grade', 'Amy')                 | 得到键名为 grade 的 zset 中 Amy 的排名                       | 1                                   |
  | zrevrank(name, value)                                        | 返回键为 name 的 zset 中元素的倒数排名（按 score 从大到小排序），即名次 | name：键名；value：元素值                                    | redis.zrevrank ('grade', 'Amy')             | 得到键名为 grade 的 zset 中 Amy 的倒数排名                   | 2                                   |
  | zrevrange(name, start, end, withscores= False)               | 返回键名为 name 的 zset（按 score 从大到小排序）中 index 从 start 到 end 的所有元素 | name：键值；start：开始索引；end：结束索引；withscores：是否带 score | redis.zrevrange ('grade', 0, 3)             | 返回键名为 grade 的 zset 中前四名元素                        | [b'Bob', b'Mike', b'Amy', b'James'] |
  | zrangebyscore (name, min, max, start=None, num=None, withscores=False) | 返回键名为 name 的 zset 中 score 在给定区间的元素            | name：键名；min：最低 score；max：最高 score；start：起始索引；num：个数；withscores：是否带 score | redis.zrangebyscore ('grade', 80, 95)       | 返回键名为 grade 的 zset 中 score 在 80 和 95 之间的元素     | [b'Bob', b'Mike', b'Amy', b'James'] |
  | zcount(name, min, max)                                       | 返回键名为 name 的 zset 中 score 在给定区间的数量            | name：键名；min：最低 score；max：最高 score                 | redis.zcount('grade', 80, 95)               | 返回键名为 grade 的 zset 中 score 在 80 到 95 的元素个数     | 2                                   |
  | zcard(name)                                                  | 返回键名为 name 的 zset 的元素个数                           | name：键名                                                   | redis.zcard('grade')                        | 获取键名为 grade 的 zset 中元素的个数                        | 3                                   |
  | zremrangebyrank (name, min, max)                             | 删除键名为 name 的 zset 中排名在给定区间的元素               | name：键名；min：最低位次；max：最高位次                     | redis.zremrangebyrank ('grade', 0, 0)       | 删除键名为 grade 的 zset 中排名第一的元素                    | 1，即删除的元素个数                 |
  | zremrangebyscore (name, min, max)                            | 删除键名为 name 的 zset 中 score 在给定区间的元素            | name：键名；min：最低 score；max：最高 score                 | redis.zremrangebyscore ('grade', 80, 90)    | 删除 score 在 80 到 90 之间的元素                            | 1，即删除的元素个数                 |

- 散列操作

  Redis 还提供了散列表的数据结构，我们可以用 name 指定一个散列表的名称，表内存储了各个键值对

  | 方　　法                     | 作　　用                                               | 参数说明                                   | 示　　例                                       | 示例说明                                             | 示例结果                                                     |
  | ---------------------------- | ------------------------------------------------------ | ------------------------------------------ | ---------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
  | hset(name, key, value)       | 向键名为 name 的散列表中添加映射                       | name：键名；key：映射键名；value：映射键值 | hset('price', 'cake', 5)                       | 向键名为 price 的散列表中添加映射关系，cake 的值为 5 | 1，即添加的映射个数                                          |
  | hsetnx(name, key, value)     | 如果映射键名不存在，则向键名为 name 的散列表中添加映射 | name：键名；key：映射键名；value：映射键值 | hsetnx('price', 'book', 6)                     | 向键名为 price 的散列表中添加映射关系，book 的值为 6 | 1，即添加的映射个数                                          |
  | hget(name, key)              | 返回键名为 name 的散列表中 key 对应的值                | name：键名；key：映射键名                  | redis.hget('price', 'cake')                    | 获取键名为 price 的散列表中键名为 cake 的值          | 5                                                            |
  | hmget(name, keys, *args)     | 返回键名为 name 的散列表中各个键对应的值               | name：键名；keys：键名序列                 | redis.hmget('price', ['apple', 'orange'])      | 获取键名为 price 的散列表中 apple 和 orange 的值     | [b'3', b'7']                                                 |
  | hmset(name, mapping)         | 向键名为 name 的散列表中批量添加映射                   | name：键名；mapping：映射字典              | redis.hmset('price', {'banana': 2, 'pear': 6}) | 向键名为 price 的散列表中批量添加映射                | True                                                         |
  | hincrby(name, key, amount=1) | 将键名为 name 的散列表中映射的值增加 amount            | name：键名；key：映射键名；amount：增长量  | redis.hincrby('price', 'apple', 3)             | key 为 price 的散列表中 apple 的值增加 3             | 6，修改后的值                                                |
  | hexists(name, key)           | 键名为 name 的散列表中是否存在键名为键的映射           | name：键名；key：映射键名                  | redis.hexists('price', 'banana')               | 键名为 price 的散列表中 banana 的值是否存在          | True                                                         |
  | hdel(name, *keys)            | 在键名为 name 的散列表中，删除键名为键的映射           | name：键名；keys：键名序列                 | redis.hdel('price', 'banana')                  | 从键名为 price 的散列表中删除键名为 banana 的映射    | True                                                         |
  | hlen(name)                   | 从键名为 name 的散列表中获取映射个数                   | name：键名                                 | redis.hlen('price')                            | 从键名为 price 的散列表中获取映射个数                | 6                                                            |
  | hkeys(name)                  | 从键名为 name 的散列表中获取所有映射键名               | name：键名                                 | redis.hkeys('price')                           | 从键名为 price 的散列表中获取所有映射键名            | [b'cake', b'book', b'banana', b'pear']                       |
  | hvals(name)                  | 从键名为 name 的散列表中获取所有映射键值               | name：键名                                 | redis.hvals('price')                           | 从键名为 price 的散列表中获取所有映射键值            | [b'5', b'6', b'2', b'6']                                     |
  | hgetall(name)                | 从键名为 name 的散列表中获取所有映射键值对             | name：键名                                 | redis.hgetall('price')                         | 从键名为 price 的散列表中获取所有映射键值对          | {b'cake': b'5', b'book': b'6', b'orange': b'7', b'pear': b'6'} |

- `RedisDump`

  `RedisDump`提供了强大的Redis数据的导入和导出功能

  - `redis-dump`

    ```shell
    Usage: redis-dump [global options] COMMAND [command options]   
        -u, --uri=S                      Redis URI (e.g. redis://hostname[:port])  
        -d, --database=S                 Redis database (e.g. -d 15)  
        -s, --sleep=S                    Sleep for S seconds after dumping (for debugging)  
        -c, --count=S                    Chunk size (default: 10000)  
        -f, --filter=S                   Filter selected keys (passed directly to redis' KEYS command)  
        -O, --without_optimizations      Disable run time optimizations  
        -V, --version                    Display version  
        -D, --debug  
            --nosafe
    ```

    其中` - u `代表 Redis 连接字符串，`-d `代表数据库代号，`-s `代表导出之后的休眠时间，`-c` 代表分块大小，默认是 10000，`-f `代表导出时的过滤器，`-O `代表禁用运行时优化，`-V `用于显示版本，`-D` 表示开启调试。

    ```shell
    redis-dump -u :passwd@localhost:6379
    ```

    运行后，可以将数据库数据导出

  - `dump-load`

    ```shell
    redis-load --help  
      Try: redis-load [global options] COMMAND [command options]   
        -u, --uri=S                      Redis URI (e.g. redis://hostname[:port])  
        -d, --database=S                 Redis database (e.g. -d 15)  
        -s, --sleep=S                    Sleep for S seconds after dumping (for debugging)  
        -n, --no_check_utf8  
        -V, --version                    Display version  
        -D, --debug  
            --nosafe
    ```

    其中` - u` 代表 Redis 连接字符串，`-d `代表数据库代号，默认是全部，`-s `代表导出之后的休眠时间，`-n `代表不检测 UTF-8 编码，`-V `表示显示版本，`-D` 表示开启调试。



# 动态页面抓取

Ajax抓取是解决动态页面的一种方式，Ajax是JavaScript动态渲染中较为容易抓取的一种，但JavaScript渲染不止Ajax一种，并且，如果Ajax接口可能含有加密参数，很难找出规律

解决动态页面抓取的另一种方法是直接模拟浏览器运行来实现，直接获取渲染后的界面源码，不再分析JavaScript如何渲染出界面

## Ajax数据

### 原理

Ajax，全称为 Asynchronous JavaScript and XML，即异步的 JavaScript 和 XML，是利用 JavaScript 在保证页面不被刷新、页面链接不改变的情况下与服务器交换数据并更新部分网页的技术

- 发送请求

  Ajax是由JavaScript实现的，实际上执行了如下代码

  ```javascript
  var xmlhttp;
  if (window.XMLHttpRequest) {
      //code for IE7+, Firefox, Chrome, Opera, Safari
      xmlhttp=new XMLHttpRequest();} else {//code for IE6, IE5
      xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
  xmlhttp.onreadystatechange=function() {if (xmlhttp.readyState==4 && xmlhttp.status==200) {document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
      }
  }
  xmlhttp.open("POST","/ajax/",true);
  xmlhttp.send();
  ```

  实际上新建了`XMLHttpRequest`对象，然后调用`onreadystatechange`属性进行了监听，然后调用`open()`和`send()`方法对服务器进行了请求

- 解析内容

  当服务器返回响应时，`onreadystatechange`对应的方法就会触发，然后调用`xmlhttp`的`responseText`属性可以获取响应内容

- 渲染网页

  JavaScript有改变网页内容的能力，在解析完响应内容后，就调用JavaScript对网页进行处理，这样的操作也叫DOM操作

### Ajax分析方法

- 查看请求

  借助浏览器的开发者工具，在Network选项卡中，筛选出XHR

  XHR是Ajax中特殊的请求类型，一般来说，Ajax请求中的`Request Headers`中有一个信息为`X-Requested-With:XMLHttpRequest`

- 过滤请求

### Ajax结果抓取

- 分析请求

  得到请求类型，请求连接，请求参数，并推测各个参数的意义

- 分析响应

  得到响应的格式，并分析响应的意义

## Selenium

### 原理

Selenium是一个自动化测试工具，可以利用它驱动浏览器完成特定的动作，还可以获取浏览器当前呈现的页面的源代码

### 使用

- 声明浏览器对象

  > 需要先安装对应的驱动

  Selenium支持多种浏览器

  ```python
  from selenium import webdriver
  
  browser = webdriver.Chrome()
  browser = webdriver.Firefox()
  browser = webdriver.Edge()
  browser = webdriver.PhantomJS()
  browser = webdriver.Safari()
  ```

- 访问页面

  ```python
  from selenium import webdriver
  
  browser = webdriver.Chrome()
  browser.get('https://www.taobao.com')
  text = browser.page_source
  browser.close()
  ```

  使用`get`方法可以请求网页，`page_source`属性能获取浏览器呈现的网页源码

- 查找节点

  - 单个节点

    使用find_element族方法，可以获取第一个符合条件的标签

    ```python
    find_element_by_id()
    find_element_by_name()
    find_element_by_xpath()
    find_element_by_link_text()
    find_element_by_partial_link_text()
    find_element_by_tag_name()
    find_element_by_class_name()
    find_element_by_css_selector()
    
    from selenium.webdriver.common.by import By
    find_element()	# 通用方法，第一个为查找方式By.*，另一个为值
    ```

    返回值为`WebElement`类型

  - 多个节点

    使用find_elements族方法，可以获取所有符合条件的标签

    ```python
    find_elements_by_id
    find_elements_by_name
    find_elements_by_xpath
    find_elements_by_link_text
    find_elements_by_partial_link_text
    find_elements_by_tag_name
    find_elements_by_class_name
    find_elements_by_css_selector
    
    from selenium.webdriver.common.by import By
    lis = browser.find_elements(By.CSS_SELECTOR, '.service-bd li')
    ```

    返回值为`WebElement`类型的列表

- 节点交互

  > 节点操作可以参考：[http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webelement)

  在选中标签获得标签对象后，常用有输入文字时使用`send_keys()`方法，清空文字时使用`clear()`方法，点击按钮时使用`click()`方法

  ```python
  from selenium import webdriver
  import time
  
  browser = webdriver.Chrome()
  browser.get('https://www.taobao.com')
  input = browser.find_element_by_id('q')
  input.send_keys('iPhone')
  time.sleep(1)
  input.clear()
  input.send_keys('iPad')
  button = browser.find_element_by_class_name('btn-search')
  button.click()
  ```

- 动作链

  一些没有特定的执行对象的操作，可以使用动作链完成

  > 动作链可以参考：[http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.common.action_chains](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.common.action_chains)

  ```python
  from selenium import webdriver
  from selenium.webdriver import ActionChains
  
  browser.get(url)
  source = browser.find_element_by_css_selector('#draggable')
  target = browser.find_element_by_css_selector('#droppable')
  actions = ActionChains(browser)
  actions.drag_and_drop(source, target)
  ```

  使用动作链需要先声明一个`ActionChains`对象，通过调用`drag_and_drop()`方法，最后调用`perform()`方法执行动作

- 执行JavaScript

  对于一些动作，可以直接模拟运行JavaScript代码进行实现

  ```python
  browser.execute_script('')
  ```

- 获取节点信息

  获取节点信息可以直接使用`page_source`属性获取源代码后使用解析库进行解析，但Selenium也提供了选择节点的方法

  - 获取属性

    对`WebElement`类型的标签对象使用`get_attribute()`方法，传入属性名，就可以获取值

    ```python
    label.get_attribute('class')
    ```

  - 获取文本值

    调用`WebElement`类型的标签对象的`text`属性就能获取文本值

    ```python
    label.text
    ```

  - 获取ID，位置，标签名，大小

    ```python
    label.id
    label.location
    label.tag_name
    label.size
    ```

- 切换Frame

  网页中有一种节点叫`iframe`，也就是子Frame，相当于子页面。在Selenium打开页面后，默认是在父Frame中操作，如果要操作子Frame，需要使用`switch_to.frame()`来切换

- 延时等待

  在Selenium中，`get()`方法会在网页框架加载结束后完成执行，但此时如果界面有额外的JS请求，可能不能完全加载，因此需要等待一定时间

  延时等待可以分为隐式等待和显式等待

  - 隐式等待

    隐式等待需要调用`implicitly_wait()`来设定，默认值为0，即Selenium没有在DOM中找到节点时，隐式等待将会等待设定的时间再次查找DOM，没有则抛出异常

    ```python
    browser.implicitly_wait(10)
    ```

  - 显式等待

    隐式等待的缺点是只规定了一个固定等待时间，显式等待规定一个最大等待时间，如果在规定时间内节点加载出来，则返回节点，否则抛出异常

    ```python
    from selenium import webdriver
    from selenium.webdriver.common.by import By
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC
    
    browser = webdriver.Chrome()
    browser.get('https://www.taobao.com/')
    wait = WebDriverWait(browser, 10)
    input = wait.until(EC.presence_of_element_located((By.ID, 'q')))
    button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '.btn-search')))
    print(input, button)
    ```

    显式等待需要先实例化一个`WebDriverWait`对象指定最长等待时间，然后调用其`until()`方法，传入等待条件`expected_conditions`，例如，使用`EC.presence_of_element_located()`判断是否加载出节点，参数是节点的定位元组

    > 等待条件可以参考：[http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.support.expected_conditions](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.support.expected_conditions)

    如果没有加载出，则会抛出`TimeoutException`异常

- 前进后退

  ```python
  browser.back()
  browser.forward()
  ```

- Cookies

  使用Selenium，可以方便地对Cookies进行操作

  ```python
  browser.add_cookie({'':''})
  browser.get_cookies()
  browser.delete_all_cookies()
  ```

- 选项卡操作

  在访问网页时，会开启一个个选项卡，可以用Selenium进行操作

  ```python
  browser.switch_to_window(browser.window_handles[1])
  ```

- 异常处理

  对于异常，可以参考[http://selenium-python.readthedocs.io/api.html#module-selenium.common.exceptions](http://selenium-python.readthedocs.io/api.html#module-selenium.common.exceptions)

### Chrome Headless模式

```python
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--headless')
browser = webdriver.Chrome(chrome_options=chrome_options)
```

## Splash

Splash是一个JavaScript渲染服务，带有HTTP API的轻量级浏览器，同时对接了Python中的Twisted和QT库

### 功能

- 异步方式处理多个网页渲染
- 获取渲染后的源代码或截图
- 关闭图片渲染或使用Adblock规则加快渲染速度
- 执行特定的JavaScript脚本
- 通过Lua脚本控制页面渲染过程
- 获取渲染的详细过程并通过HAR(HTTP Archive)格式呈现

### 安装

使用docker服务可以方便的使用Splash

```shell
docker run -d -p 8050:8050 scrapinghub/splash
```

打开[http://localhost:8050/](http://localhost:8050/)即可打开Web界面

### 基本使用

在输入框内输入网址即可开始渲染，会返回渲染截图，HAR加载数据统计，网页源码等

![7-7](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/7-7.png)

渲染过程由一段Lua脚本控制

```python
function main(splash, args)
  assert(splash:go(args.url))
  assert(splash:wait(0.5))
  return {html = splash:html(),
    png = splash:png(),
    har = splash:har(),}
end
```

### Lua脚本

- 入口及返回值

  Splash会默认调用`main`方法，返回值可以为字符串，也可以是字典

  > 注意Lua中的字典形式`{hello="world!"}`

- 异步处理

  Splash支持异步处理，但无需显式指明回调方法，回调的跳转是在Splash内部完成的

  ```python
  function main(splash, args)
    local example_urls = {"www.baidu.com", "www.taobao.com", "www.zhihu.com"}
    local urls = args.urls or example_urls
    local results = {}
    for index, url in ipairs(urls) do
      local ok, reason = splash:go("http://" .. url)
      if ok then
        splash:wait(2)
        results[url] = splash:png()
      end
    end
    return results
  end
  ```

  其中，`wait`方法类似于`sleep`方法，当Splash执行到此处时，会转而处理其他任务，在指定之间后返回继续处理

  > Lua的语法：[http://www.runoob.com/lua/lua-basic-syntax.html](http://www.runoob.com/lua/lua-basic-syntax.html)

- `Splash`对象属性

  `main`方法第一个参数为`splash`对象，类似于Selenium中的`WebDriver`对象

  - `args`

    脚本中`main`方法的参数有一个`args`，Splash也支持省去，省去后`args`参数会直接作为`splash`对象的属性

  - `js_enabled`

    `js_ebabled`控制是否执行JavaScript代码，默认为`true`

    > 注意，如果调用`splash:evaljs('')`执行JavaScript代码便会抛出异常

  - `resource_timeout`

    可以设置加载超时时间，超时会抛出异常，默认为0或`nil`(类似于Python中的`None`)，表示不检测超时

  - `images_enabled`

    设置图片是否加载，默认加载，禁用可以节省流量并提高加载速度。但要注意禁止图片加载可能影响外层DOM节点的高度和位置，进而影响JavaScript渲染

    > 如果在加载图片后禁用加载图片，再重新加载页面，由于使用了缓存，图片可能仍会加载

  - `plugins_enabled`

    控制浏览器插件是否开启，默认为`false`

  - `scroll_position`

    设置页面上下或左右滚动，比较常用

    ```lua
    splash.scroll_position = {x=100, y=200}
    ```

- `Splash`对象方法

  - `go`

    用于请求某个链接，可以模拟GET和POST请求，同时支持传入请求头，表单等

    ```lua
    ok, reason = splash:go{url, baseurl=nil, headers=nil, http_method="GET", body=nil, formdata=nil}
    ```

    - `url`：请求的url
    - `baserl`：可选参数，默认为空，资源加载的相对路径
    - `headers`：可选参数，默认为空，请求的Headers
    - `http_method`：可选参数，默认为GET，支持POST
    - `body`：可选参数，默认为空，POST请求时的表单数据，使用的Content-Type为`application/json`
    - formdata：可选参数，默认为空，POST请求时的表单数据，使用的Content-Type为`application/x-www-form-urlencoded`
    - `ok`：如果`ok`为空，则网页加载错误，`reason`中包含了错误的原因

  - `wait`

    控制页面等待时间

    ```lua
    ok, reason = splash:wait{time, cancel_on_redirect=false, cancel_on_error=true}
    ```

    - `time`：等待的秒数
    - `cancel_on_redirect`：可选参数，默认` False`，如果发生了重定向就停止等待，并返回重定向结果
    - `cancel_on_error`：可选参数，默认 `False`，如果发生了加载错误就停止等待

  - `jsfunc`

    可以直接调用JavaScript 定义的方法，即将 JavaScript 定义的方法直接转换为Lua方法，但是所调用的方法需要用双中括号包围

    ```lua
    local name = splash:jsfunc([[function () {
        ...
      }
      ]])
    ```

    > 可以参阅[https://splash.readthedocs.io/en/stable/scripting-ref.html#splash-jsfunc](https://splash.readthedocs.io/en/stable/scripting-ref.html#splash-jsfunc)

  - `evaljs`

    可以执行JavaScript 代码并返回最后一条 JavaScript 语句的返回结果

    ```lua
    result = splash:evaljs(js)
    ```

  - `runjs`

    与 evaljs 方法的功能类似，但是更偏向于执行某些动作或声明某些方法

    ```lua
    splash:runjs("foo = function() {return 'bar'}")
    local result = splash:evaljs("foo()")
    ```

  - `autoload`

    设置每个界面访问时自动加载的代码或库，不执行任何操作

    ```lua
    ok, reason = splash:autoload{source_or_url, source=nil, url=nil}
    ```

    * `source_or_url`：JavaScript 代码或者 JavaScript 库链接。
    * `source`：JavaScript 代码。
    * `url`：JavaScript 库链接

  - `call_later`

    设定定时任务和延迟时间实现任务延迟执行，并且可以通过`cancel`方法重新执行定时任务，例如在0.2和1.2秒后截图一次

    ```lua
    local timer = splash:call_later(function()
      snapshots["a"] = splash:png()
      splash:wait(1.0)
      snapshots["b"] = splash:png()
    end, 0.2)
    ```

  - `http_get`

    模拟发送HTTP的GET请求

    ```lua
    response = splash:http_get{url, headers=nil, follow_redirects=true}
    ```

    * `url`：请求 URL
    * `headers`：可选参数，默认为空，请求的 Headers
    * `follow_redirects`：可选参数，默认为 True，是否启动自动重定向

  - `http_post`

    模拟发送一个POST请求

    ```lua
    response = splash:http_post{url, headers=nil, follow_redirects=true, body=nil}
    ```

    * `url`：请求 URL
    * `headers`：可选参数，默认为空，请求的 Headers
    * `follow_redirects`：可选参数，默认为 True，是否启动自动重定向
    * `body`：可选参数，默认为空，即表单数据

  - `set_content`

    设置页面的内容

  - `html`

    获取网页的源代码

  - `png`

    获取PNG格式的网页截图

  - `jpeg`

    获取JPEG格式的网页截图

  - `har`

    获取页面的加载描述过程

  - `url`

    获取正在访问的url

  - `get_cookies`

    获取当前页面的Cookies

  - `add_cookies`

    为当前页面添加Cookies

  - `clear_cookies`

    清除所有的Cookies

  - `get_viewport_size`

    获取当前浏览器页面的大小，返回为二维数组

  - `set_viewport_size`

    设置当前浏览器的大小

  - `set_viewport_full`

    设置全屏显示

  - `set_user_agent`

    设置浏览器的User-Agent

  - `set_custom_headers`

    设置请求的Headers

  - `select`

    使用CSS选择器选择符合条件的第一个节点

  - `select_all`

    使用CSS选择器选中所有符合条件的节点

  - `mouse_click`

    模拟鼠标的点击操作，传入坐标x，y值，或选中某个节点直接调用此方法

> Splash常用的操作可以参考[https://splash.readthedocs.io/en/stable/scripting-ref.html](https://splash.readthedocs.io/en/stable/scripting-ref.html)和[https://splash.readthedocs.io/en/stable/scripting-element-object.html](https://splash.readthedocs.io/en/stable/scripting-element-object.html)

### Splash API

Splash提供了一些HTTP接口，请求接口并传递响应参数就可以获得结果

- `render.html`

  获取Splash获取渲染后的页面的HTML代码，使用Splash的运行地址加接口名称就可以访问

  ```python
  import requests
  url = 'http://localhost:8050/render.html?url=https://www.baidu.com'
  response = requests.get(url)
  print(response.text)
  ```

  > 具体使用参考：[https://splash.readthedocs.io/en/stable/api.html#render-html](https://splash.readthedocs.io/en/stable/api.html#render-html)

- `render.png`

  获取网页截图，可以通过``width`和`height`控制宽高，返回二进制数据

  > 详细参考：[https://splash.readthedocs.io/en/stable/api.html#render-png](https://splash.readthedocs.io/en/stable/api.html#render-png)

- `render.har`

  获取页面加载的HAR数据，返回值为JSON格式的数据

- `render.json`

  包含前面三个接口的所有功能，返回时JSON格式，可以用参数控制

  > 详细参考：[https://splash.readthedocs.io/en/stable/api.html#render-json](https://splash.readthedocs.io/en/stable/api.html#render-json)

- `execute`

  `lua_source`参数中传递lua脚本源代码，执行lua脚本，并获取结果

  > 注意用 urllib.parse 模块里的 quote() 方法将脚本进行 URL 转码

### Splash均衡负载

可以利用Nginx搭建Splash的分布集群实现均衡负载

- Nginx配置

  ```nginx
  http {
  	upstream splash {
  		least_conn;	# 代表最少链接负载均衡，适合处理时间长短不一的情况
              		# 
  		server ...;	# 可以加入weight=；指定权重
          server ...;
          ...
  	}
  	server {
  		listen 8050;
  		location / {
              proxy_pass http://splash;
              auth_basic "Restricted";	# 若不希望公开，可以使用认证，使用htpasswd生成账户密码，需要在请求时加入auth参数
              auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
          }
  	}
  }
  ```

  

# 验证码识别

- 验证码是一种反爬机制
- 解决方法：
  - 人工肉眼识别
  - 库识别
  - 第三方自动识别

## 图形验证码

图形验证码可以使用OCR技术进行识别，tesserocr库提供了对应的接口

- 获取验证码

- 识别

  ```python
  import tesserocr
  from PIL import Image
  
  image = Image.open('code.jpg')
  result = tesserocr.image_to_text(image)
  ```

  ```python
  import tesserocr
  tesserocr.file_to_text('image.png')
  ```

  第一种方法借助PIL加载图片，第二种直接调用tesserocr的方法直接转换，但第一种方法识别效果较好

- 验证码处理

  对于有轻度多余线条的图片，可以先进行转灰度和二值化处理，提高识别精度

  ```python
  image = image.convert('L')
  image = image.convert('1')	# 二值化，默认阈值为127
  ```

## 极验滑动验证码

华东验证码需要拖动合并滑块才能完成验证

![8-5](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/8-5.jpg)

### 特点

极验验证码相较于图形验证码来说识别难度更大，还增加了机器学习方法识别拖动轨迹

当前极验验证码首先判断是否在同一会话下，如果是，则点击后直接通过验证，否则弹出滑动验证窗口

### 识别

- 过程

  识别验证需要完成三步

  - 模拟点击验证按钮
  - 识别滑动缺口的位置
  - 模拟拖动滑块

- 思路
  - 构造模拟表单提交，但需要分析加密参数构造
  - 模拟浏览器行为，需要注意模仿人类行为

### 实现

- 使用Selenium模拟点击按钮

- 识别缺口位置

  - 可以使用边缘检测算法

  - 在没有滑块前，会显示原图，可以通过原图对比来识别缺口位置

    ```python
    def get_position(self):
        """
        获取验证码位置
        :return: 验证码位置元组
        """
        img = self.wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'geetest_canvas_img')))
        time.sleep(2)
        location = img.location
        size = img.size
        top, bottom, left, right = location['y'], location['y'] + size['height'], location['x'], location['x'] + size['width']
        return (top, bottom, left, right)
    
    def get_geetest_image(self, name='captcha.png'):
        """
        获取验证码图片
        :return: 图片对象
        """
        top, bottom, left, right = self.get_position()
        print(' 验证码位置 ', top, bottom, left, right)
        screenshot = self.get_screenshot()
        captcha = screenshot.crop((left, top, right, bottom))
        return captcha
    
    def get_slider(self):
        """
        获取滑块
        :return: 滑块对象
        """
        slider = self.wait.until(EC.element_to_be_clickable((By.CLASS_NAME, 'geetest_slider_button')))
        return slider
    def is_pixel_equal(self, image1, image2, x, y):
        """
        判断两个像素是否相同
        :param image1: 图片 1
        :param image2: 图片 2
        :param x: 位置 x
        :param y: 位置 y
        :return: 像素是否相同
        """
        # 取两个图片的像素点
        pixel1 = image1.load()[x, y]
        pixel2 = image2.load()[x, y]
        threshold = 60
        if abs(pixel1[0] - pixel2[0]) <threshold and abs(pixel1[1] - pixel2[1]) < threshold and abs(pixel1[2] - pixel2[2]) < threshold:
            return True
        else:
            return False
            
    def get_gap(self, image1, image2):
        """
        获取缺口偏移量
        :param image1: 不带缺口图片
        :param image2: 带缺口图片
        :return:
        """
        left = 60
        for i in range(left, image1.size[0]):
            for j in range(image1.size[1]):
                if not self.is_pixel_equal(image1, image2, i, j):
                    left = i
                    return left
        return left
    ```

- 模拟拖动

  正常人拖动一般为先加速，后减速，极验验证码利用机器学习模型，匀速，围绕平均速度随机抖动都无法通过

  可以使用加速度公式完成

  ```python
  x = v0 * t + 0.5 * a * t * t 
  v = v0 + a * t
  ```

  ```python
  def get_track(self, distance):
      """
      根据偏移量获取移动轨迹
      :param distance: 偏移量
      :return: 移动轨迹
      """
      # 移动轨迹
      track = []
      # 当前位移
      current = 0
      # 减速阈值
      mid = distance * 4 / 5
      # 计算间隔
      t = 0.2
      # 初速度
      v = 0
      
      while current < distance:
          if current < mid:
              # 加速度为正 2
              a = 2
          else:
              # 加速度为负 3
              a = -3
          # 初速度 v0
          v0 = v
          # 当前速度 v = v0 + at
          v = v0 + a * t
          # 移动距离 x = v0t + 1/2 * a * t^2
          move = v0 * t + 1 / 2 * a * t * t
          # 当前位移
          current += move
          # 加入轨迹
          track.append(round(move))
      return track
  def move_to_gap(self, slider, tracks):
      """
      拖动滑块到缺口处
      :param slider: 滑块
      :param tracks: 轨迹
      :return:
      """
      ActionChains(self.browser).click_and_hold(slider).perform()
      for x in tracks:
          ActionChains(self.browser).move_by_offset(xoffset=x, yoffset=0).perform()
      time.sleep(0.5)
      ActionChains(self.browser).release().perform()
  ```

## 点触验证码

典型的点触验证码如下图所示，一个提供点触验证码的网站为[https://www.touclick.com/](https://www.touclick.com/)

![8-18](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/8-18.jpg)

![8-21](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/8-21.jpg)

### 特点

- 识别难度大，需要先识别文字，然后识别文字对应的图像，或背景改绕大

### 实现

- 可以借助第三方平台解决，例如[超级鹰](https://www.chaojiying.com)

## 宫格验证码

![8-25](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/8-25.jpg)

### 识别思路

- 可以发现，这种宫格所有的情况并不多，可以借助穷举模板实现

- 只匹配箭头，但箭头只有几个像素值，成功率不高
- 匹配全图，虽然模板多，但工作量更少，精度更高

### 实现

- 获取模板

  ```python
  import time
  from io import BytesIO
  from PIL import Image
  from selenium import webdriver
  from selenium.common.exceptions import TimeoutException
  from selenium.webdriver.common.by import By
  from selenium.webdriver.support.ui import WebDriverWait
  from selenium.webdriver.support import expected_conditions as EC
  
  
  USERNAME = ''
  PASSWORD = ''
  
  class CrackWeiboSlide():
      def __init__(self):
          self.url = 'https://passport.weibo.cn/signin/login'
          self.browser = webdriver.Chrome()
          self.wait = WebDriverWait(self.browser, 20)
          self.username = USERNAME
          self.password = PASSWORD
  
      def __del__(self):
          self.browser.close()
  
      def open(self):
          """
          打开网页输入用户名密码并点击
          :return: None
          """
          self.browser.get(self.url)
          username = self.wait.until(EC.presence_of_element_located((By.ID, 'loginName')))
          password = self.wait.until(EC.presence_of_element_located((By.ID, 'loginPassword')))
          submit = self.wait.until(EC.element_to_be_clickable((By.ID, 'loginAction')))
          username.send_keys(self.username)
          password.send_keys(self.password)
          submit.click()
  
      def get_position(self):
          """
          获取验证码位置
          :return: 验证码位置元组
          """
          try:
              img = self.wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'patt-shadow')))
          except TimeoutException:
              print(' 未出现验证码 ')
              self.open()
          time.sleep(2)
          location = img.location
          size = img.size
          top, bottom, left, right = location['y'], location['y'] + size['height'], location['x'], location['x'] + size['width']
          return (top, bottom, left, right)
  
      def get_screenshot(self):
          """
          获取网页截图
          :return: 截图对象
          """
          screenshot = self.browser.get_screenshot_as_png()
          screenshot = Image.open(BytesIO(screenshot))
          return screenshot
  
      def get_image(self, name='captcha.png'):
          """
          获取验证码图片
          :return: 图片对象
          """
          top, bottom, left, right = self.get_position()
          print(' 验证码位置 ', top, bottom, left, right)
          screenshot = self.get_screenshot()
          captcha = screenshot.crop((left, top, right, bottom))
          captcha.save(name)
          return captcha
  
      def main(self):
          """
          批量获取验证码
          :return: 图片对象
          """
          count = 0
          while True:
              self.open()
              self.get_image(str(count) + '.png')
              count += 1
  
  if __name__ == '__main__':
      crack = CrackWeiboSlide()
      crack.main()
  ```

- 模板匹配

  ```python
  from os import listdir
  
  def detect_image(self, image):
      """
      匹配图片
      :param image: 图片
      :return: 拖动顺序
      """
      for template_name in listdir(TEMPLATES_FOLDER):
          print(' 正在匹配 ', template_name)
          template = Image.open(TEMPLATES_FOLDER + template_name)
          if self.same_image(image, template):
              # 返回顺序
              numbers = [int(number) for number in list(template_name.split('.')[0])]
              print(' 拖动顺序 ', numbers)
              return numbers
  def is_pixel_equal(self, image1, image2, x, y):
      """
      判断两个像素是否相同
      :param image1: 图片 1
      :param image2: 图片 2
      :param x: 位置 x
      :param y: 位置 y
      :return: 像素是否相同
      """
      # 取两个图片的像素点
      pixel1 = image1.load()[x, y]
      pixel2 = image2.load()[x, y]
      threshold = 20
      if abs(pixel1[0] - pixel2[0]) <threshold and abs(pixel1[1] - pixel2[1]) < threshold and abs(pixel1[2] - pixel2[2]) < threshold:
          return True
      else:
          return False
  
  def same_image(self, image, template):
      """
      识别相似验证码
      :param image: 待识别验证码
      :param template: 模板
      :return:
      """
      # 相似度阈值
      threshold = 0.99
      count = 0
      for x in range(image.width):
          for y in range(image.height):
              # 判断像素是否相同
              if self.is_pixel_equal(image, template, x, y):
                  count += 1
      result = float(count) / (image.width * image.height)
      if result > threshold:
          print(' 成功匹配 ')
          return True
      return False
  ```

- 模拟拖动

  ```python
  def move(self, numbers):
      """
      根据顺序拖动
      :param numbers:
      :return:
      """
      # 获得四个按点
      circles = self.browser.find_elements_by_css_selector('.patt-wrap .patt-circ')
      dx = dy = 0
      for index in range(4):
          circle = circles[numbers[index] - 1]
          # 如果是第一次循环
          if index == 0:
              # 点击第一个按点
              ActionChains(self.browser) \
                  .move_to_element_with_offset(circle, circle.size['width'] / 2, circle.size['height'] / 2) \
                  .click_and_hold().perform()
          else:
              # 小幅移动次数
              times = 30
              # 拖动
              for i in range(times):
                  ActionChains(self.browser).move_by_offset(dx /times, dy /times).perform()
                  time.sleep(1 /times)
          # 如果是最后一次循环
          if index == 3:
              # 松开鼠标
              ActionChains(self.browser).release().perform()
          else:
              # 计算下一次偏移
              dx = circles[numbers[index + 1] - 1].location['x'] - circle.location['x']
              dy = circles[numbers[index + 1] - 1].location['y'] - circle.location['y']
  ```



# 代理

-  破解封ip的反爬机制
-  作用：
   - 突破自身IP访问的限制
   - 隐藏自身的IP

```python
proxy = {"https":""}	# 代理的ip字典，协议:地址
response = requests.get(url, proxies=proxy)	# 使用代理发送get请求
```

- 匿名度：
  - 透明：既知道使用了代理，也知道真实IP
  - 匿名：知道使用了代理，但不知道真实IP
  - 高匿：既不知道使用了代理，也不知道真实IP

## 代理设置

### urllib

- urllib模块需要使用`ProxyHandler`类创建Opener

  ```python
  from urllib.request import ProxyHandler, build_opener
  
  proxy = 'username:password@127.0.0.1:9743'	# 如果需要认证
  proxy_handler = ProxyHandler({
      'http': 'http://' + proxy,
      'https': 'https://' + proxy
  })
  opener = build_opener(proxy_handler)
  ```

- 如果为socks5类型，则可以直接设置

  ```python
  socks.set_default_proxy(socks.SOCKS5, '127.0.0.1', 9742)
  socket.socket = socks.socksocket
  ```

  需要先安装socks模块

  ```shell
  pip3 install PySocks
  ```

### requests

- 对于requests，只需要传入`proxies`参数即可

  ```python
  import requests
  
  proxy = '127.0.0.1:9743'
  proxies = {
      'http': 'http://' + proxy,
      'https': 'https://' + proxy,
  }
  response = requests.get('http://httpbin.org/get', proxies=proxies)
  ```

  > 使用socks代理需要安装模块
  >
  > ```shell
  > pip3 install "requests[socks]"
  > ```

### Selenium

- 对于Chrome，直接通过`ChromeOptions`来设置代理

  ```python
  from selenium import webdriver
  
  proxy = '127.0.0.1:9743'
  chrome_options = webdriver.ChromeOptions()
  chrome_options.add_argument('--proxy-server=http://' + proxy)
  browser = webdriver.Chrome(chrome_options=chrome_options)
  ```

  带有认证的代理设置相对比较麻烦，需要创建manifest.json配置文件和background.js脚本设置代理，运行后会生成一个proxy_auth_plugin.zip文件来保存当前配置

  ```python
  from selenium import webdriver
  from selenium.webdriver.chrome.options import Options
  import zipfile
  
  ip = '127.0.0.1'
  port = 9743
  username = 'foo'
  password = 'bar'
  
  manifest_json = """{"version":"1.0.0","manifest_version": 2,"name":"Chrome Proxy","permissions": ["proxy","tabs","unlimitedStorage","storage","<all_urls>","webRequest","webRequestBlocking"],"background": {"scripts": ["background.js"]
      }
  }
  """background_js ="""
  var config = {
          mode: "fixed_servers",
          rules: {
            singleProxy: {
              scheme: "http",
              host: "%(ip) s",
              port: %(port) s
            }
          }
        }
  
  chrome.proxy.settings.set({value: config, scope: "regular"}, function() {});
  
  function callbackFn(details) {
      return {
          authCredentials: {username: "%(username) s",
              password: "%(password) s"
          }
      }
  }
  
  chrome.webRequest.onAuthRequired.addListener(
              callbackFn,
              {urls: ["<all_urls>"]},
              ['blocking']
  )
  """ % {'ip': ip, 'port': port, 'username': username, 'password': password}
  
  plugin_file = 'proxy_auth_plugin.zip'
  with zipfile.ZipFile(plugin_file, 'w') as zp:
      zp.writestr("manifest.json", manifest_json)
      zp.writestr("background.js", background_js)
  chrome_options = Options()
  chrome_options.add_argument("--start-maximized")
  chrome_options.add_extension(plugin_file)
  browser = webdriver.Chrome(chrome_options=chrome_options)
  ```

## 代理池

代理池可以提前对代理进行筛选，保留可用代理

### 目标

代理池基本模块分为4个

- 存储模块：负责存储抓取的代理，要保证不重复并动态实时处理每个代理，可以使用Redis的`Sorted Set`
- 获取模块：定时从代理网站获取代理，成功后保存到数据库中
- 检测模块：定时检测数据库中的代理，最好是检测需要爬取的网站。需要设置代理的状态，设置分数标识，如果代理可用，加一分，否则减一分，当分数低于一定阈值后，进行删除
- 接口模块：用API提供对外的接口服务

![9-1](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/9-1.jpg)

### IP有效性判断

分数是判断代理稳定性的标准。可以设定以下规则：

- 分数100为可用，检测器会定时检测每个代理，一旦可用就设置为100，由此可以最大程度上筛选出可用代理
- 如果检测代理不可用，则分数减1，减至0后移除代理，保证拯救代理的机会多
- 新代理分数设置为10，检测的机会没有可用代理那么多，减少系统开销

### 实现

- 存储模块

  ```python
  MAX_SCORE = 100
  MIN_SCORE = 0
  INITIAL_SCORE = 10
  REDIS_HOST = 'localhost'
  REDIS_PORT = 6379
  REDIS_PASSWORD = None
  REDIS_KEY = 'proxies'
  
  import redis
  from random import choice
  
  
  class RedisClient(object):
      def __init__(self, host=REDIS_HOST, port=REDIS_PORT, password=REDIS_PASSWORD):
          """
          初始化
          :param host: Redis 地址
          :param port: Redis 端口
          :param password: Redis 密码
          """
          self.db = redis.StrictRedis(host=host, port=port, password=password, decode_responses=True)
      
      def add(self, proxy, score=INITIAL_SCORE):
          """
          添加代理，设置分数为最高
          :param proxy: 代理
          :param score: 分数
          :return: 添加结果
          """
          if not self.db.zscore(REDIS_KEY, proxy):
              return self.db.zadd(REDIS_KEY, score, proxy)
      
      def random(self):
          """
          随机获取有效代理，首先尝试获取最高分数代理，如果不存在，按照排名获取，否则异常
          :return: 随机代理
          """
          result = self.db.zrangebyscore(REDIS_KEY, MAX_SCORE, MAX_SCORE)
          if len(result):
              return choice(result)
          else:
              result = self.db.zrevrange(REDIS_KEY, 0, 100)
              if len(result):
                  return choice(result)
              else:
                  raise PoolEmptyError
      
      def decrease(self, proxy):
          """
          代理值减一分，小于最小值则删除
          :param proxy: 代理
          :return: 修改后的代理分数
          """
          score = self.db.zscore(REDIS_KEY, proxy)
          if score and score > MIN_SCORE:
              print(' 代理 ', proxy, ' 当前分数 ', score, ' 减 1')
              return self.db.zincrby(REDIS_KEY, proxy, -1)
          else:
              print(' 代理 ', proxy, ' 当前分数 ', score, ' 移除 ')
              return self.db.zrem(REDIS_KEY, proxy)
      
      def exists(self, proxy):
          """
          判断是否存在
          :param proxy: 代理
          :return: 是否存在
          """
          return not self.db.zscore(REDIS_KEY, proxy) == None
      
      def max(self, proxy):
          """
          将代理设置为 MAX_SCORE
          :param proxy: 代理
          :return: 设置结果
          """
          print(' 代理 ', proxy, ' 可用，设置为 ', MAX_SCORE)
          return self.db.zadd(REDIS_KEY, MAX_SCORE, proxy)
      
      def count(self):
          """
          获取数量
          :return: 数量
          """
          return self.db.zcard(REDIS_KEY)
      
      def all(self):
          """
          获取全部代理
          :return: 全部代理列表
          """return self.db.zrangebyscore(REDIS_KEY, MIN_SCORE, MAX_SCORE)```
  ```

- 获取模块

  ```python
  import json
  from .utils import get_page
  from pyquery import PyQuery as pq
  
  class ProxyMetaclass(type):
      def __new__(cls, name, bases, attrs):
          count = 0
          attrs['__CrawlFunc__'] = []
          for k, v in attrs.items():
              if 'crawl_' in k:
                  attrs['__CrawlFunc__'].append(k)
                  count += 1
          attrs['__CrawlFuncCount__'] = count
          return type.__new__(cls, name, bases, attrs)
  
  class Crawler(object, metaclass=ProxyMetaclass):
      def get_proxies(self, callback):
          proxies = []
          for proxy in eval("self.{}()".format(callback)):
              print(' 成功获取到代理 ', proxy)
              proxies.append(proxy)
          return proxies
  
      def crawl_daili66(self, page_count=4):
          """
          获取代理 66
          :param page_count: 页码
          :return: 代理
          """
          start_url = 'http://www.66ip.cn/{}.html'
          urls = [start_url.format(page) for page in range(1, page_count + 1)]
          for url in urls:
              print('Crawling', url)
              html = get_page(url)
              if html:
                  doc = pq(html)
                  trs = doc('.containerbox table tr:gt(0)').items()
                  for tr in trs:
                      ip = tr.find('td:nth-child(1)').text()
                      port = tr.find('td:nth-child(2)').text()
                      yield ':'.join([ip, port])
  
      def crawl_proxy360(self):
          """
          获取 Proxy360
          :return: 代理
          """
          start_url = 'http://www.proxy360.cn/Region/China'
          print('Crawling', start_url)
          html = get_page(start_url)
          if html:
              doc = pq(html)
              lines = doc('div[name="list_proxy_ip"]').items()
              for line in lines:
                  ip = line.find('.tbBottomLine:nth-child(1)').text()
                  port = line.find('.tbBottomLine:nth-child(2)').text()
                  yield ':'.join([ip, port])
  
      def crawl_goubanjia(self):
          """
          获取 Goubanjia
          :return: 代理
          """
          start_url = 'http://www.goubanjia.com/free/gngn/index.shtml'
          html = get_page(start_url)
          if html:
              doc = pq(html)
              tds = doc('td.ip').items()
              for td in tds:
                  td.find('p').remove()
                  yield td.text().replace(' ', '')
  
  from db import RedisClient
  from crawler import Crawler
  
  POOL_UPPER_THRESHOLD = 10000
  
  class Getter():
      def __init__(self):
          self.redis = RedisClient()
          self.crawler = Crawler()
      
      def is_over_threshold(self):
          """判断是否达到了代理池限制"""
          if self.redis.count()>= POOL_UPPER_THRESHOLD:
              return True
          else:
              return False
      
      def run(self):
          print(' 获取器开始执行 ')
          if not self.is_over_threshold():
              for callback_label in range(self.crawler.__CrawlFuncCount__):
                  callback = self.crawler.__CrawlFunc__[callback_label]
                  proxies = self.crawler.get_proxies(callback)
                  for proxy in proxies:
                      self.redis.add(proxy)
  ```

- 检测模块

  ```python
  VALID_STATUS_CODES = [200]
  TEST_URL = 'http://www.baidu.com'
  BATCH_TEST_SIZE = 100
  
  class Tester(object):
      def __init__(self):
          self.redis = RedisClient()
      
      async def test_single_proxy(self, proxy):
          """
          测试单个代理
          :param proxy: 单个代理
          :return: None
          """
          conn = aiohttp.TCPConnector(verify_ssl=False)
          async with aiohttp.ClientSession(connector=conn) as session:
              try:
                  if isinstance(proxy, bytes):
                      proxy = proxy.decode('utf-8')
                  real_proxy = 'http://' + proxy
                  print(' 正在测试 ', proxy)
                  async with session.get(TEST_URL, proxy=real_proxy, timeout=15) as response:
                      if response.status in VALID_STATUS_CODES:
                          self.redis.max(proxy)
                          print(' 代理可用 ', proxy)
                      else:
                          self.redis.decrease(proxy)
                          print(' 请求响应码不合法 ', proxy)
              except (ClientError, ClientConnectorError, TimeoutError, AttributeError):
                  self.redis.decrease(proxy)
                  print(' 代理请求失败 ', proxy)
      
      def run(self):
          """
          测试主函数
          :return: None
          """
          print(' 测试器开始运行 ')
          try:
              proxies = self.redis.all()
              loop = asyncio.get_event_loop()
              # 批量测试
              for i in range(0, len(proxies), BATCH_TEST_SIZE):
                  test_proxies = proxies[i:i + BATCH_TEST_SIZE]
                  tasks = [self.test_single_proxy(proxy) for proxy in test_proxies]
                  loop.run_until_complete(asyncio.wait(tasks))
                  time.sleep(5)
          except Exception as e:
              print(' 测试器发生错误 ', e.args)
  ```

- 接口模块

  ```python
  from flask import Flask, g
  from db import RedisClient
  
  __all__ = ['app']
  app = Flask(__name__)
  
  def get_conn():
      if not hasattr(g, 'redis'):
          g.redis = RedisClient()
      return g.redis
  
  @app.route('/')
  def index():
      return '<h2>Welcome to Proxy Pool System</h2>'
  
  @app.route('/random')
  def get_proxy():
      """
      获取随机可用代理
      :return: 随机代理
      """
      conn = get_conn()
      return conn.random()
  
  @app.route('/count')
  def get_counts():
      """
      获取代理池总量
      :return: 代理池总量
      """
      conn = get_conn()
      return str(conn.count())
  
  
  if __name__ == '__main__':
      app.run()
  ```

## 付费代理

付费代理分为两类：

- 一类提供接口获取海量代理
- 一类搭建了代理隧道，设置固定域名代理

可以尝试[讯代理](http://www.xdaili.cn/)，[阿布云代理](https://www.abuyun.com/)

## ADSL拨号代理

ADSL，非对称数字用户环路，上行和下行带宽部队称，采用频分服用技术将普通电话线分为电话，上行和下行3个独立信道

ADSL拨号上网每次拨号就更换一个IP，IP分布在多个A段，主机稳定性很好，响应也很快

> 购买可以使用[云立方](http://www.yunlifang.cn/dynamicvps.asp)，安装系统后进行拨号

### 设置代理服务器

- 安装TinyProxy

- 配置TinyProxy

  配置文件路径为/etc/tinyproxy/tinyproxy.conf，设置端口，允许主机

- 动态获取IP

  可以执行命令让主机动态切换IP，切换途中不可用，接口数据最理想的还是使用数据库存储，可以使用MySQL或者Redis的Hash存储

- 实现

  - 首先操作Redis数据库

    ```python
    import redis
    import random
    
    # Redis 数据库 IP
    REDIS_HOST = 'remoteaddress'
    # Redis 数据库密码，如无则填 None
    REDIS_PASSWORD = 'foobared'
    # Redis 数据库端口
    REDIS_PORT = 6379
    # 代理池键名
    PROXY_KEY = 'adsl'
    
    
    class RedisClient(object):
        def __init__(self, host=REDIS_HOST, port=REDIS_PORT, password=REDIS_PASSWORD, proxy_key=PROXY_KEY):
            """
            初始化 Redis 连接
            :param host: Redis 地址
            :param port: Redis 端口
            :param password: Redis 密码
            :param proxy_key: Redis 哈希表名
            """
            self.db = redis.StrictRedis(host=host, port=port, password=password, decode_responses=True)
            self.proxy_key = proxy_key
        
        def set(self, name, proxy):
            """
            设置代理
            :param name: 主机名称
            :param proxy: 代理
            :return: 设置结果
            """
            return self.db.hset(self.proxy_key, name, proxy)
        
        def get(self, name):
            """
            获取代理
            :param name: 主机名称
            :return: 代理
            """
            return self.db.hget(self.proxy_key, name)
        
        def count(self):
            """
            获取代理总数
            :return: 代理总数
            """
            return self.db.hlen(self.proxy_key)
        
        def remove(self, name):
            """
            删除代理
            :param name: 主机名称
            :return: 删除结果
            """
            return self.db.hdel(self.proxy_key, name)
        
        def names(self):
            """
            获取主机名称列表
            :return: 获取主机名称列表
            """
            return self.db.hkeys(self.proxy_key)
        
        def proxies(self):
            """
            获取代理列表
            :return: 代理列表
            """
            return self.db.hvals(self.proxy_key)
        
        def random(self):
            """
            随机获取代理
            :return:
            """
            proxies = self.proxies()
            return random.choice(proxies)
        
        def all(self):
            """
            获取字典
            :return:
            """return self.db.hgetall(self.proxy_key)
    ```

  - 定时拨号

    在服务器上定时拨号更换IP，并向数据库更新信息

    ```python
    import re
    import time
    import requests
    from requests.exceptions import ConnectionError, ReadTimeout
    from db import RedisClient
    
    # 拨号网卡
    ADSL_IFNAME = 'ppp0'
    # 测试 URL
    TEST_URL = 'http://www.baidu.com'
    # 测试超时时间
    TEST_TIMEOUT = 20
    # 拨号间隔
    ADSL_CYCLE = 100
    # 拨号出错重试间隔
    ADSL_ERROR_CYCLE = 5
    # ADSL 命令
    ADSL_BASH = 'adsl-stop;adsl-start'
    # 代理运行端口
    PROXY_PORT = 8888
    # 客户端唯一标识
    CLIENT_NAME = 'adsl1'
    
    class Sender():
        def get_ip(self, ifname=ADSL_IFNAME):
            """
            获取本机 IP
            :param ifname: 网卡名称
            :return:
            """
            (status, output) = subprocess.getstatusoutput('ifconfig')
            if status == 0:
                pattern = re.compile(ifname + '.*?inet.*?(\d+\.\d+\.\d+\.\d+).*?netmask', re.S)
                result = re.search(pattern, output)
                if result:
                    ip = result.group(1)
                    return ip
    
        def test_proxy(self, proxy):
            """
            测试代理
            :param proxy: 代理
            :return: 测试结果
            """
            try:
                response = requests.get(TEST_URL, proxies={
                    'http': 'http://' + proxy,
                    'https': 'https://' + proxy
                }, timeout=TEST_TIMEOUT)
                if response.status_code == 200:
                    return True
            except (ConnectionError, ReadTimeout):
                return False
    
        def remove_proxy(self):
            """
            移除代理
            :return: None
            """
            self.redis = RedisClient()
            self.redis.remove(CLIENT_NAME)
            print('Successfully Removed Proxy')
    
        def set_proxy(self, proxy):
            """
            设置代理
            :param proxy: 代理
            :return: None
            """
            self.redis = RedisClient()
            if self.redis.set(CLIENT_NAME, proxy):
                print('Successfully Set Proxy', proxy)
    
        def adsl(self):
            """
            拨号主进程
            :return: None
            """
            while True:
                print('ADSL Start, Remove Proxy, Please wait')
                self.remove_proxy()
                (status, output) = subprocess.getstatusoutput(ADSL_BASH)
                if status == 0:
                    print('ADSL Successfully')
                    ip = self.get_ip()
                    if ip:
                        print('Now IP', ip)
                        print('Testing Proxy, Please Wait')
                        proxy = '{ip}:{port}'.format(ip=ip, port=PROXY_PORT)
                        if self.test_proxy(proxy):
                            print('Valid Proxy')
                            self.set_proxy(proxy)
                            print('Sleeping')
                            time.sleep(ADSL_CYCLE)
                        else:
                            print('Invalid Proxy')
                    else:
                        print('Get IP Failed, Re Dialing')
                        time.sleep(ADSL_ERROR_CYCLE)
                else:
                    print('ADSL Failed, Please Check')
                    time.sleep(ADSL_ERROR_CYCLE)
    def run():
        sender = Sender()
        sender.adsl()
    ```

  - 接口模块

    ```python
    import json
    import tornado.ioloop
    import tornado.web
    from tornado.web import RequestHandler, Application
    
    # API 端口
    API_PORT = 8000
    
    class MainHandler(RequestHandler):
        def initialize(self, redis):
            self.redis = redis
        
        def get(self, api=''):
            if not api:
                links = ['random', 'proxies', 'names', 'all', 'count']
                self.write('<h4>Welcome to ADSL Proxy API</h4>')
                for link in links:
                    self.write('<a href=' + link + '>' + link + '</a><br>')
            
            if api == 'random':
                result = self.redis.random()
                if result:
                    self.write(result)
            
            if api == 'names':
                result = self.redis.names()
                if result:
                    self.write(json.dumps(result))
            
            if api == 'proxies':
                result = self.redis.proxies()
                if result:
                    self.write(json.dumps(result))
            
            if api == 'all':
                result = self.redis.all()
                if result:
                    self.write(json.dumps(result))
            
            if api == 'count':
                self.write(str(self.redis.count()))
    
    
    def server(redis, port=API_PORT, address=''):
        application = Application([(r'/', MainHandler, dict(redis=redis)),
            (r'/(.*)', MainHandler, dict(redis=redis)),
        ])
        application.listen(port, address=address)
        print('ADSL API Listening on', port)
        tornado.ioloop.IOLoop.instance().start()
    ```

## 请求队列

对于反爬能力很强的网站，可以借助数据库构造爬取队列，将待爬取的请求都放到队列中，请求失败时重新放回队列

```python
from pickle import dumps, loads
from request import WeixinRequest

class RedisQueue():
    def __init__(self):
        """初始化 Redis"""
        self.db = StrictRedis(host=REDIS_HOST, port=REDIS_PORT, password=REDIS_PASSWORD)

    def add(self, request):
        """
        向队列添加序列化后的 Request
        :param request: 请求对象
        :param fail_time: 失败次数
        :return: 添加结果
        """
        if isinstance(request, WeixinRequest):
            return self.db.rpush(REDIS_KEY, dumps(request))
        return False

    def pop(self):
        """
        取出下一个 Request 并反序列化
        :return: Request or None
        """
        if self.db.llen(REDIS_KEY):
            return loads(self.db.lpop(REDIS_KEY))
        else:
            return False

    def empty(self):
        return self.db.llen(REDIS_KEY) == 0
```



# 模拟登陆

## 登陆

- 请求登陆的Cookie可能是在请求页面以Set-Cookie的形式传送，加密方式可能隐藏在输入的源码中

- 登陆一般会发送一个post请求，请求会携带登陆前录入的相关信息

- 一些网站在登陆后拿到的不一定会是页面源码，可以使用pose请求的响应状态码判断登陆是否成功

  ```python
  response.status_code	# 获取响应状态码，200为登陆成功
  ```

> 一般来说，登陆后反爬的概率较低

## cookie操作

- http/https协议特性：无状态
- cookie：用来让服务器端记录客户端的相关状态 
  - 手动cookie处理：手动写headers(处理麻烦，可能有有效时长)
  - 自动处理：一般cookie可以从post请求中获取

### session会话对象

- 可以进行请求的发送

- 如果请求中产生了cookie，则cookie会被存储在该session对象中

  ```python
  session = request.Session()	# 创建session对象
  session.post()	# 使用session进行post请求发送
  session.get()
  ```

### Cookie池

Cookie池可以存储登陆后的Cookies信息，定时检测Cookie的有效性，在Cookie无效后删除并模拟登陆生成新的Cookie，Cookie池与代理池结构类似



# 异步爬虫

## 分类

- 多线程，多进程(不建议)

  - 好处：可以为相关阻塞操作开启单独线程或进程
  - 弊端：无法无限制开启多线程或多进程

- 进程池，线程池(适当使用)：

  - 好处：降低系统对进程或线程创建和销毁的频率，降低系统的开销
  - 弊端：池中线程或进程的数量有上限(如果阻塞方法远大于池中线程数量，效果不明显)
  - 原则：线程池处理的是阻塞且耗时的操作(视频的下载，写入文件...)

  ```python
  from multiprocessing.dummy import Pool
  
  pool = Pool(4)	# 实例化一个线程池对象
  pool.map(method, list)	# 将列表中的元素交给方法进行处理
  pool.close()	# 关闭线程池
  pool.join()	# 主线程等待子线程结束
  ```

- 单线程+异步协程

  - event_loop：事件循环，可以将函数注册到事件循环上，当满足条件，函数就会被循环执行
  - coroutine：协程对象，可以将协程对象注册到时间循环中，其会被事件循环调用
    - async关键字可以定义一个方法，在调用时不会被立即执行，而是返回一个协程对象

  - task：任务，对协程对象的进一步封装，包含了任务的各个状态
  - future：代表将来执行或还没有执行的任务，实际和task没有本质区别(实现方式不同)
  - async：定义一个协程
  - await：挂起阻塞方法的执行

  ```python
  import asyncio	# 导入包
  
  async def request():	# 定义一个方法，调用时不会立即执行，而是返回一个协程对象
      pass
  
  c = request()	# 接收协程对象
  
  loop = asyncio.get_event_loop()	# 创建一个事件循环对象
  
  task1 = loop.create_task(c)	# 基于loop创建一个task任务对象
  							# pending表示待执行，finished表示已执行
  task2 = asyncio.ensure_future(c)	# 创建一个future任务对象
  
  def callback_func(task):	# 定义回调函数，默认会将任务对象作为参数传入
      task.result()	# 返回任务的结果，即任务方法的返回值
  task.add_done_callback(callback_func)	# 绑定回调函数，在任务执行完成后调用
  
  loop.run_until_complete(c)	# 将协程对象或任务对象注册到loop中，并启动循环
  ```

- 多任务协程

  ```python
  tasks = [task1, ...]	# 创建任务列表
  loop.run_until_complete(asyncio.wait(tasks))	# 不能将tasks直接传入
  ```

  > 在异步协程中出现了同步模块相关的代码，那么就无法实现异步
  >
  > time.sleep(2)  --> await asyncio.sleep(2)
  >
  > 遇到阻塞操作要用await手动挂起

## aiohttp

在异步协程中出现了同步模块相关的代码，那么就无法实现异步，例如

```python
time.sleep(2)  --> await asyncio.sleep(2)	# 遇到阻塞操作要用await手动挂起
```

request库中请求是基于同步的，异步需要使用aiohttp库

- 安装

  ```python
  pip install aiohttp
  ```

- 使用

  ```python
  async with aiohttp.ClientSession() as session:
      async with await session.get (url) as response:	# 要用await手动挂起
          page_text = await response.text()	# text在此处为方法
          page_content = await response.read()	# read返回二进制形式的数据
          page_json = await response.json()	# 返回json对象
  ```

> 这里的get和post的proxy参数传入的为一个字符串而非字典





# scrapy框架

## 原理

### 架构

![13-1](https://raw.githubusercontent.com/chanochLi/photo/master/blog_pic/13-1.jpg)

* Engine，引擎，用来处理整个系统的数据流处理，触发事务，是整个框架的核心
* Item，项目，它定义了爬取结果的数据结构，爬取的数据会被赋值成该对象
* Scheduler， 调度器，用来接受引擎发过来的请求并加入队列中，并在引擎再次请求的时候提供给引擎
* Downloader，下载器，用于下载网页内容，并将网页内容返回给蜘蛛
* Spiders，蜘蛛，其内定义了爬取的逻辑和网页的解析规则，它主要负责解析响应并生成提取结果和新的请求
* Item Pipeline，项目管道，负责处理由蜘蛛从网页中抽取的项目，它的主要任务是清洗、验证和存储数据
* Downloader Middlewares，下载器中间件，位于引擎和下载器之间的钩子框架，主要是处理引擎与下载器之间的请求及响应
* Spider Middlewares， 蜘蛛中间件，位于引擎和蜘蛛之间的钩子框架，主要工作是处理蜘蛛输入的响应和输出的结果及新的请求

### 功能

- 高性能的持久化存储
- 异步的数据下载
- 高性能的数据解析
- 分布式

### 过程

* Engine 首先打开一个网站，找到处理该网站的 Spider 并向该 Spider 请求第一个要爬取的 URL
* Engine 从 Spider 中获取到第一个要爬取的 URL 并通过 Scheduler 以 Request 的形式调度
* Engine 向 Scheduler 请求下一个要爬取的 URL
* Scheduler 返回下一个要爬取的 URL 给 Engine，Engine 将 URL 通过 Downloader Middlewares 转发给 Downloader 下载
* 一旦页面下载完毕， Downloader 生成一个该页面的 Response，并将其通过 Downloader Middlewares 发送给 Engine
* Engine 从下载器中接收到 Response 并通过 Spider Middlewares 发送给 Spider 处理。
* Spider 处理 Response 并返回爬取到的 Item 及新的 Request 给 Engine
* Engine 将 Spider 返回的 Item 给 Item Pipeline，将新的 Request 给 Scheduler
* 重复第二步到最后一步，直到  Scheduler 中没有更多的 Request，Engine 关闭该网站，爬取结束

### 项目结构

```
scrapy.cfg
project/
    __init__.py
    items.py
    pipelines.py
    settings.py
    middlewares.py
    spiders/
        __init__.py
        spider1.py
        spider2.py
        ...
```

- scrapy.cfg：它是 Scrapy 项目的配置文件，其内定义了项目的配置文件路径、部署相关信息等内容

- items.py：它定义 Item 数据结构，所有的 Item 的定义都可以放这里

- pipelines.py：它定义 Item Pipeline 的实现，所有的 Item Pipeline 的实现都可以放这里

- settings.py：它定义项目的全局配置

- middlewares.py：它定义 Spider Middlewares 和 Downloader Middlewares 的实现

- spiders：其内包含一个个 Spider 的实现，每个 Spider 都有一个文件

## 安装

```python
pip install scrapy
```

## 使用

### 工程指令

```shell
$ scrapy startproject xxx	# 创建一个scrapy工程
$ cd xxx	# 进入工程目录
$ scrapy genspider spiderName www.xxx.com	# 生成爬虫文件，指定名称和初始网址
$ scrapy crawl spiderName	# 执行特定爬虫
```

### 爬虫文件

- 爬虫类：父类为scrapy.Spider

  - 属性：

    - name：爬虫文件的名称，已经设定

    - allowed_domains = [' ']：允许的域名，限定start_urls中哪些url可以被请求

    - start_urls = ['']：起始url列表，会被scrapy进行自动请求发送

      > 但一般allowed_domains一般注释不使用

  - 方法：

    - parse(self, response)：该方法负责解析返回的响应、提取数据或者进一步生成要处理的请求
    
      ```python
      def parse(self, response):
          selector_list = response.xpath('')	# 对响应对象使用xpath
          									# 返回的是Selector对象列表
          selector.extract()	# 调用selector对象extract方法提取，返回内容
          selector_list.extract()	# 对每个对象调用extract方法，返回仍为列表
          selector_list.extract_first()	# 若保证只有一个元素，对其解析
      ```
    
      > scrapy中response.text获取html，response.body获取二进制

### 配置文件

- ROBOTSTXT_OBEY：是否遵从robots协议
- LOG_LEVEL：输出日志类型，'ERROR'等
- USER_AGENT：scrapy所使用的UA伪装
- ITEM_PIPELINES：字典，键为管道类名，值为管道优先级，越小优先级越高
- IMAGES_STORE：字符串，指定ImagePipeline存储图片的目录

### 持久化存储(Item和Pipeline)

- 基于终端指令

  - 要求：只能将parse方法的返回值存储到本地的特定类型的文件中

  ```shell
  $ scrapy crawl spiderName -o file	# 将解析的输出保存到文件
  ```

  - 文件类型只能为json，jsonlines，jl，csv，xml，marshal，pickle

- 基于管道

  - 流程：

    - 数据解析

    - 在item.py的item类中定义相关的属性

      ```python
      content = scrapy.Field()	# 使用方法封装属性
      ```

    - 将解析的数据封装存储到item类型的对象中

      ```python
      item = xxxItem()
      item['content'] = xxx	# 需要使用['']访问，不能用.
      ```

    - 将item类型的对象提交给管道进行持久化存储

      ```python
      yield item	# 使用yield进行异步返回
      ```

      修改pipelines.py中的process_item进行修改，进行存储

      ```python
      class xxxPipeline(object):
          fp = None
          def open_spider(self, spider):
          # 重写父类方法，只会在开始时调用一次，打开文件只需一次
          
          def process_item(self, item, spider):
             	xxx = item['author']
              
              # 返回的item会传递给次优先级的管道
              return item
          
          def close_spider(self,spider):
          # 只会在结束时调用一次，可以用于关闭文件
      ```

    - 在配置文件中开启管道

### 全站爬取(CrawlSpider)

- 就将网站某板块下全部页码(分页)下的全部数据爬取

- 手动请求发送

  - 将所有页面的元素加入到start_urls中(不推荐)

  - 实现

    ```python
    yield parse(self, response):
        ...
        # 用新的url请求，并回调parse进行解析，为递归，需要退出条件
        yield scrapy.Request(url=new_url, callback=self.parse)
    ```

- CrawlSpider类：Spider的一个子类，用于全站数据爬取

  ```shell
  $ scrapy genspider -t crawl xxx www.xxx.com	# 基于cralSpicder创建爬虫文件
  ```

  ```python
  from scrapy.linkextractors import LinkExtractor
  from scrapy.spiders import CrawlSpider, Rule
  
  class TestSpider(CrawlSpider):
  	# 链接提取器
      link = LinkExtractor(allow=r'')	# allow指定规则，输入正则表达式
      # 规则解析器
      # follow决定是否对子页面也进行提取
      rules = (Rule(link, callback='parse_item', follow=False),)
  ```

  - 链接提取器：根据指定规则(allow参数的正则)进行指定链接的提取
  - 规则解析器：对链接提取器提取的链接进行制定规则(callback)的解析

### 请求传参

- 使用场景：解析的数据不在同一张页面中(深度爬取)

- 使用meta参数传送，使用meta方法接受

  ```python
  yield scrapy.Request(url, callback=self.parse1, meta=item)
  
  def parse1(self, response):
      item = response.meta['item']
  ```

### 图片爬取ImagesPipeline

- 对比

  - 字符串：只需要基于xpath解析且提交管道进行持久化存储
  - 图片：xpath解析出src属性值，还需单独对图片发送请求获取二进制数据

- 基于ImagesPipeline

  - 只需要解析出图片的地址，提交到管道，不需要传送二进制，管道会请求二进制数据并进行存储

  - 用IMAGES——STORE指定存储目录

    ```python
    from scrapy.pipelines.images import ImagesPipeline
    class YandeReDbPipeline(ImagesPipeline):
    
        # 重写父类方法，获取图片响应对象
        def get_media_requests(self, item, info):
            yield scrapy.Request(url=item['src'])
    
        # 重写父类方法，获取图片存储路径
        def file_path(self, request, response=None, info=None, *, item=None):
            imgName = request.url.split('/')[-1]
            yield imgName
    
        # 重写父类方法，返回item给下一个管道
        def item_completed(self, results, item, info):
            return item
    ```

- 图片懒加载

  - 使用src2伪属性，被拖动到可视区域，才会变为src进行加载
  - scrapy没有可视化，需要对伪属性进行解析

### 分布式爬虫

- 搭建一个分布式的机群，对一组资源进行分布联合爬取，提升爬虫效率

- 安装

  ```shell
  pip install scrapy-redis	# 原生scrapy不能实现分布式爬虫
  							# 组件可以实现调度器和管道的共享
  ```

- 实现流程

  - 修改爬虫文件

    - 导入模块，将爬虫类父类修改为RedisCrawlSpider

      ```python
      from scrapy_redis.spiders import RedisCrawlSpider
      ```

    - 注释start_urls和allowed_domains进行注释，加入redis_key='name'属性，表示可以被共享的调度器队列

    - 编写数据解析相关为操作

  - 修改配置文件

    - 指定可以被共享的管道

      ```python
      ITEM_PIPELINES = {
          'scrapy_redis.pipelines.RedisPipeline':400
      }
      ```

    - 指定调度器

      ```python
      DUPEFILTER_CLASS = 'scrapy_redis.dupefilter.RFPDupeFilter'	# 过滤器
      SCHEDULER = 'scrapy_redis.scheduler.Scheduler'	# 调度器
      SCHEDULER_PERSIST = True	# 是否持久化
      ```

  - redis相关配置：

    - 注释bind 127.0.0.1，否则其它客户端不能访问

    - 关闭保护模式：protected-mode no，否则其它客户端不能写数据

    - 开启redis服务

    - 指定redis服务器(默认为本机)

      ```python
      REDIS_HOST = 'ip'
      REDIS_PORT = port
      ```

  - 执行工程

    ```shell
    $ scrapy runspider xxx.py
    ```

  - 向调度器的队列放入一个起始url

    ```shell
    lpush redis_key www.xxx.com
    ```

### 增量式爬虫

- 可以使用redis的set，加入判断，根据返回值决定是否爬取

### Spider

- 对url封装为请求对象
- 对响应对象进行解析

### 调度器

- 对请求对象去重
- 对请求对象进行排队

### 下载器

>  twisted实现异步下载

- 对数据进行下载

### 管道

- 进行持久化存储

### 引擎

- 用于数据流处理
- 触发事务(对象实例化，调用方法)

> 根据收到的数据流判断触发的事务

### 中间件

- 爬虫中间件：引擎与Spider间

- 下载中间件：在引擎和下载间

  - 拦截请求：头信息(UA伪装)，代理ip

    > 配置文件中是基于全局的，但中间件中是对于每个请求

    - middlewares文件

      ```python
      # 下载中间件重写
      class MiddleproDownloaderMiddleware:
          @classmethod
          # 拦截请求
          def process_request(self, request, spider):
      		
              return None
      
          # 拦截异常
          def process_exception(self, request, exception, spider):
      
              pass
      ```

      在配置文件中设置DOWNLOADER__MIDDLEWARES使用下载中间件

  - 拦截响应：篡改响应数据或响应对象

    > 一般通过中间件拦截响应来处理动态加载问题

    ```python
        def process_response(self, request, response, spider):
            # 通过url指定request，通过request指定response
    
            if request.url in spider.url_list:
                # 五大板块的响应对象
                # 基于selemium获取动态响应对象替代原响应对象
                # selenium最好在爬虫对象中定义
                bro = spider.bro
                bro.get(request.url)
                time.sleep(3)
                page_text = bro.page_source
                new_response = HtmlResponse(url=request.url, body=page_text, encoding='utf-8', request=request)
                return new_response
            return response
    ```



# APP爬取

APP爬取相比于Web端更加容易，反爬虫能力没有那么强，APP端需要借助各种抓包工具，分析APP运行过程中的请求和响应，如果没有规律，可以利用mitmdump直接处理Response，另外，也要对APP进行自动化控制

## Charles

### 原理

Charles运行在PC上，运行时会在PC的8888端口开启HTTP/HTTPS代理服务

设置手机代理为Charles的代理地址，手机访问的数据包就会流经Charles，Charles起到中间人的作用，可以捕获所有的流量包，同时Charles还有权利对请求和响应进行修改

Charles对于明文抓包比较友好

## mitmproxy

mitmproxy类似于Charles，支持HTTP/HTTPS抓包，但为命令行形式，它有两个插件，一个是mitmdump，是mitmproxy的命令行接口，可以对接Python脚本，另一个是mitmweb，是一个Web程序，可以观察不获得请求

### 功能

- 拦截HTTP/HTTPS响应
- 保存HTTP会话并分析
- 模拟客户端发起请求，模拟服务端返回响应
- 进行流量转发
- 支持透明代理
- 利用Python对HTTP请求和响应进行处理

### mitmdump

mitmdump可以对接Python

```shell
mitmdump -w outfile	# 保存截获的数据包
mitmdump -s script.py	# 将截获的数据包交给脚本处理
```

```python
def request(flow):
    flow.request.headers['User-Agent'] = 'MitmProxy'
```

脚本中接受一个`HTTPFlow`对象

- 日志

  ```python
  from mitmproxy import ctx
  
  def request(flow):
      ctx.log.info(str(flow.request.headers))
  ```

  日志需要调用`ctx`模块，调用不同的方法进行输出

- Request

  ```python
  def request(flow):
      url = flow.request.url
  ```

  `flow.request`是一个请求对象，由URL，Headers，Cookies等常见属性，可以直接赋值修改

  > 更多可以参考[http://docs.mitmproxy.org/en/latest/scripting/api.html](http://docs.mitmproxy.org/en/latest/scripting/api.html)

- Response

  ```python
  def response(flow):
      response = flow.response
  ```

  使用`flow.response`是一个Response对象，可以获取每个请求的响应内容

## Appium

Appium是一个跨平台自动化测试工具，可以模拟App内部的各种操作，实际上继承自Selenium。对IOS，Appium使用UIAutomation来驱动，对Android，使用UiAutomator和Selendroid实现驱动

### 启动APP

- Appium内置驱动器

  需要配置启动 App 时的 Desired Capabilities 参数，它们分别是 platformName、deviceName、appPackage、appActivity

  * platformName，平台名称，需要区分是 Android 还是 iOS
  * deviceName，设备名称，是手机的具体类型
  * appPackage，APP 程序包名
  * appActivity，入口 Activity 名，这里通常需要以。开头

  Start Session后，手机启动App，PC上会弹出调试窗口，可以查看界面并查看源码，可以显示基本信息和可执行的操作

  Appium也提供了录制操作，在窗口的App的操作都会被记录，可以自动生成相应的代码

- Python代码驱动

  - 首先指定Appium Server

  - 使用字典配置Desired Capabilities 参数

    ```python
    server = 'http://localhost:4723/wd/hub'
    desired_caps = {
        'platformName': 'Android',
        'deviceName': 'MI_NOTE_Pro',
        'appPackage': 'com.tencent.mm',
        'appActivity': '.ui.LauncherUI'
    }
    
    from appium import webdriver
    from selenium.webdriver.support.ui import WebDriverWait
    
    driver = webdriver.Remote(server, desired_caps)
    ```

  - 之后就可以使用自动生成的代码

### API

代码操作APP，总结API为[https://github.com/appium/python-client](https://github.com/appium/python-client)

- 初始化

  需要配置Desired Capabilities 参数，可以参考[https://github.com/appium/appium/blob/master/docs/en/writing-running-appium/caps.md](https://github.com/appium/appium/blob/master/docs/en/writing-running-appium/caps.md)

- 查找元素

  可以使用类似Selenium中通用的方法查找

  在Android，可以使用UIAutomator进行查找

  ```python
  el = self.driver.find_element_by_android_uiautomator('new UiSelector().description("Animation")')
  ```

  在IOS，可以使用iOS Predicates或iOS Class Chain进行查找

  ```python
  el = self.driver.find_element_by_ios_predicate('wdName == "Buttons"')
  el = self.driver.find_element_by_ios_class_chain('XCUIElementTypeWindow/XCUIElementTypeButton[3]')
  ```

  可以参考[https://github.com/appium/appium-xcuitest-driver](https://github.com/appium/appium-xcuitest-driver)

- 点击

  可以使用`tap()`方法点击(最多5个手指)，设置按的时长(毫秒)

  ```python
  tap(self, positions, duration=None)
  ```

  * `positions`，点击的位置组成的列表
  * `duration`，点击持续时间

  也可以对元素调用`click()`方法进行点击

- 屏幕拖动

  ```python
  scroll(self, origin_el, destination_el)
  ```

  可以使用`scroll()`方法模拟屏幕滚动，实现从元素`origin_el`滚动至元素`destination_el`

  * `original_el`，被操作的元素
  * `destination_el`，目标元素

  也可以使用`swipe()`模拟从A滑到B

  ```python
  swipe(self, start_x, start_y, end_x, end_y, duration=None)
  ```

  * `start_x`，开始位置的横坐标
  * `start_y`，开始位置的纵坐标
  * `end_x`，终止位置的横坐标
  * `end_y`，终止位置的纵坐标
  * `duration`，持续时间，毫秒

  又或者使用`flick()`方法模拟从A点快速滑动到B点

  ```python
  flick(self, start_x, start_y, end_x, end_y)
  ```

  * `start_x`，开始位置的横坐标
  * `start_y`，开始位置的纵坐标
  * `end_x`，终止位置的横坐标
  * `end_y`，终止位置的纵坐标

- 拖拽

  可以使用`drag_and_drop()`方法实现某个元素拖动到另一个元素上

  ```python
  drag_and_drop(self, origin_el, destination_el)
  ```

  * original_el，被拖拽的元素
  * destination_el，目标元素

- 文本输入

  使用`set_text()`实现文本输入

  ```python
  el.set_text('Hello')
  ```

- 动作链

  与Selenium中的`ActionChains`类似，Appium中的`TouchAction`可支持的方法有`tap()`、`press()`、`long_press()`、`release()`、`move_to()`、`wait()`、`cancel()`等

> 更多可以参考[https://testerhome.com/topics/3711](https://testerhome.com/topics/3711)

### Appium+mitmdump

Appium可以对动作进行模拟，但只能获取呈现的信息，mitmproxy可以获取所有响应，但无法自动化，可以使用Appium进行操作，同时mitmdump截取数据的方法进行爬取数据
