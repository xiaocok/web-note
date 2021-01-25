# CORS跨域
跨域HTTP请求(cross-origin HTTP request)

---

### 1、跨域CORS
#### 1.1 简介
* 相同的域
    当两个域具有相同的协议(如http), 相同的域名或者host，相同的端口(如80)，那么我们就可以认为它们是相同的域（协议，域名，端口都必须相同）。

* 跨域
    指着协议，域名，端口不一致，出于安全考虑，跨域的资源之间是无法交互的(例如一般情况跨域的JavaScript无法交互，当然有很多解决跨域的方案)

#### 1.2 场景
> 当一个资源请求一个其它域名的资源时会发起一个跨域HTTP请求(cross-origin HTTP request)。
比如说，域名A(http://domaina.example)的某 Web 应用通过<img>标签引入了域名B(http://domainb.foo)的某图片资源(http://domainb.foo/image.jpg)，域名A的 Web 应用就会导致浏览器发起一个跨域 HTTP 请求。
在当今的 Web 开发中，使用跨域 HTTP 请求加载各类资源（包括CSS、图片、JavaScript 脚本以及其它类资源），已经成为了一种普遍且流行的方式。

#### 1.3 跨域场景分析

| 当前页面url | 被请求页面url | 是否跨域 | 原因 |
| --- | --- | --- | --- |
|http://www.test.com/       |   http://www.test.com/index.html	|   否	    |   同源（协议、域名、端口号相同）   |
|http://www.test.com/	    |   https://www.test.com/index.html	|   跨域	|   协议不同（http/https）         |
|http://www.test.com/	    |   http://www.baidu.com/	        |   跨域	|   主域名不同（test/baidu）       |
|http://www.test.com/	    |   http://blog.test.com/	        |   跨域	|   子域名不同（www/blog）         |
|http://www.test.com:8080/	|   http://www.test.com:7001/	    |   跨域	|   端口号不同（8080/7001）        |

#### 1.4 浏览器禁止跨域
> 正如大家所知，出于安全考虑，浏览器会限制脚本中发起的跨域请求。
比如，使用 XMLHttpRequest 对象和Fetch发起 HTTP 请求就必须遵守同源策略。 
具体而言，Web 应用程序通过 XMLHttpRequest 对象或Fetch能且只能向同域名的资源发起 HTTP 请求，而不能向任何其它域名发起请求。
为了能开发出更强大、更丰富、更安全的Web应用程序，开发人员渴望着在不丢失安全的前提下，Web 应用技术能越来越强大、越来越丰富。
比如，可以使用 XMLHttpRequest 发起跨站 HTTP 请求。

> 准确的说应该是，跨域并非浏览器限制了发起跨站请求，而是跨站请求可以正常发起，但是返回结果被浏览器拦截了。
最好的例子是[CSRF跨站伪造攻击](CSRF攻击.md)，请求是发送到了后端服务器无论是否跨域！
注意：有些浏览器不允许从HTTPS的域跨域访问HTTP，比如Chrome和Firefox，这些浏览器在请求还未发出的时候就会拦截请求，这是一个特例。

#### 1.5 非同源限制
1. 无法读取非同源网页的 Cookie、LocalStorage 和 IndexedDB
2. 无法接触非同源网页的 DOM
3. 无法向非同源地址发送 AJAX 请求

### 2、解决跨域的几种方案
#### 解决方案
1. 设置document.domain解决无法读取非同源网页的 Cookie问题<br/>
    仅限主域相同，子域不同的跨域应用场景
    因为浏览器是通过document.domain属性来检查两个页面是否同源，因此只要通过设置相同的document.domain，两个页面就可以共享Cookie。

    ```javascript
    // 两个页面都设置
    document.domain = 'test.com';
    ```

2. 跨文档通信 API：window.postMessage()<br/>
    调用postMessage方法实现父窗口http://test1.com向子窗口http://test2.com发消息（子窗口同样可以通过该方法发送消息给父窗口）

    它可用于解决以下方面的问题：
    * 页面和其打开的新窗口的数据传递
    * 多窗口之间消息传递
    * 页面与嵌套的iframe消息传递
    * 上面三个场景的跨域数据传递

    ```javascript
    // 父窗口打开一个子窗口
    var openWindow = window.open('http://test2.com', 'title');
     
    // 父窗口向子窗口发消息(第一个参数代表发送的内容，第二个参数代表接收消息窗口的url)
    openWindow.postMessage('Nice to meet you!', 'http://test2.com');
    ```
    
    调用message事件，监听对方发送的消息
    ```javascript
    // 监听 message 消息
    window.addEventListener('message', function (e) {
      console.log(e.source); // e.source 发送消息的窗口
      console.log(e.origin); // e.origin 消息发向的网址
      console.log(e.data);   // e.data   发送的消息
    },false);
    ```
    
3. JSONP<br/>
    JSONP 是服务器与客户端跨源通信的常用方法。最大特点就是简单适用，兼容性好（兼容低版本IE），缺点是只支持get请求，不支持post请求。<br/>
    核心思想：网页通过添加一个< script >元素，向服务器请求 JSON 数据，服务器收到请求后，将数据放在一个指定名字的回调函数的参数位置传回来。
    
    * 原生实现：
    ```javascript
    <script src="http://test.com/data.php?callback=dosomething"></script>
    // 向服务器test.com发出请求，该请求的查询字符串有一个callback参数，用来指定回调函数的名字
     
    // 处理服务器返回回调函数的数据
    <script type="text/javascript">
        function dosomething(res){
            // 处理获得的数据
            console.log(res.data)
        }
    </script>
    ```
    
    *  jQuery ajax：
    ```javascript
    $.ajax({
        url: 'http://www.test.com:8080/login',
        type: 'get',
        dataType: 'jsonp',  // 请求方式为jsonp
        jsonpCallback: "handleCallback",    // 自定义回调函数名
        data: {}
    });
    ```
    
    * Vue.js
    ```javascript
    this.$http.jsonp('http://www.domain2.com:8080/login', {
        params: {},
        jsonp: 'handleCallback'
    }).then((res) => {
        console.log(res); 
    })
    ```
    
4. CORS<br/>
    CORS 是跨域资源分享（Cross-Origin Resource Sharing）的缩写。它是 W3C 标准，属于跨源 AJAX 请求的根本解决方法。<br/>
    1). 普通跨域请求：只需服务器端设置Access-Control-Allow-Origin<br/>
    2). 带cookie跨域请求：前后端都需要进行设置<br/>
 
    **【前端设置】**<br/>
    根据xhr.withCredentials字段判断是否带有cookie
    * 原生ajax
    ```javascript
    var xhr = new XMLHttpRequest(); // IE8/9需用window.XDomainRequest兼容
     
    // 前端设置是否带cookie
    xhr.withCredentials = true;
     
    xhr.open('post', 'http://www.domain2.com:8080/login', true);
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    xhr.send('user=admin');
     
    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4 && xhr.status == 200) {
            alert(xhr.responseText);
        }
    };
    ```
    
    * jQuery ajax 
    ```javascript
    $.ajax({
       url: 'http://www.test.com:8080/login',
       type: 'get',
       data: {},
       xhrFields: {
           withCredentials: true    // 前端设置是否带cookie
       },
       crossDomain: true,   // 会让请求头中包含跨域的额外信息，但不会含cookie
    });
    ```
    
    * vue-resource
    ```javascript
    Vue.http.options.credentials = true
    ```
    
    * axios 
    ```javascript
    axios.defaults.withCredentials = true
    ```

    **【服务端设置】**<br/>
    服务器端对于CORS的支持，主要是通过设置Access-Control-Allow-Origin来进行的。如果浏览器检测到相应的设置，就可以允许Ajax进行跨域的访问。
    * Java后台
    * GoLang后台
    * Nodejs后台
    * PHP后台<br/>
        ```php
        <?php
         header("Access-Control-Allow-Origin:*");
        ``` 
    * Apache<br/>
        需要使用mod_headers模块来激活HTTP头的设置，它默认是激活的。你只需要在Apache配置文件的<Directory>, <Location>, <Files>或<VirtualHost>的配置里加入以下内容即可。
        ```text
        Header set Access-Control-Allow-Origin *
        ```
   
5. window.name<br/>
6. websockets<br/>

#### 备注
> Chrome和Firefox浏览器不允许从HTTPS的域跨域访问HTTP。

> 如果源地址为https, 那么目标地址也必须为https。其中必须使用有效的证书。
> 如果目标地址为不安全的证书，请求会被浏览器拦截，将无法发送https请求。
> 不安全的https页面的请求，浏览器会提示用户网页不安全，用户可以点击继续访问。

> 而在跨域模式下，XMLHttpRequest请求，浏览器无法提示网页不安全，因此直接拦截请求。


### 3、跨域资源共享CORS
> CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。
> 它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。

#### 3.1 Cors简介
> CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。
整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。
浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。
因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

> 浏览器必须能支持跨源共享带来的新的组件，包括请求头和策略执行。同样，服务器端则需要解析这些新的请求头，并按照策略返回相应的响应头以及所请求的资源。

跨源资源共享标准( cross-origin sharing standard ) 使得以下场景可以使用跨站 HTTP 请求：
* 使用 XMLHttpRequest 或 Fetch发起跨站 HTTP 请求。
* Web 字体 (CSS 中通过 @font-face 使用跨站字体资源), 因此，网站就可以发布 TrueType 字体资源，并只允许已授权网站进行跨站调用。
* WebGL 贴图
* 使用drawImage绘制Images/video 画面到canvas.
* 样式表（使用 CSSOM）
* Scripts (for unmuted exceptions).

#### 3.2 Cors的两种请求
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

满足以下两大条件,属于简单请求
```text
(1) 请求方法是以下三种方法之一：
    HEAD
    GET
    POST
(2) HTTP的头信息不超出以下几种字段：
    Accept
    Accept-Language
    Content-Language
    Last-Event-ID
    Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
    未使用自定义请求头（类似于 X-Modified 这种）
```

这是为了兼容表单（form），因为历史上表单一直可以发出跨域请求。AJAX 的跨域设计就是，只要表单可以发，AJAX 就可以直接发。
凡是不同时满足上面两个条件，就属于非简单请求。
浏览器对这两种请求的处理，是不一样的。

#### 3.3 简单请求
##### **基本流程**
对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。

下面是一个例子，浏览器发现这次跨源AJAX请求是简单请求，就自动在头信息之中，添加一个Origin字段。
```text
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

上面的头信息中，Origin字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。

如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。
浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段（详见下文），就知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。
注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。
```text
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

上面的头信息之中，有三个与CORS请求相关的字段，都以Access-Control-开头。

###### 1. Access-Control-Allow-Origin
```text
该字段是必须的,他的值要么是请求Origin字段的值,要么是一个*, 表示接受任意域名的请求.
```

###### 2. Access-Control-Allow-Credentials
```text
该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可.
```

###### 3. Access-Control-Expose-Headers
```text
该字段可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值。
```

#### 3.4 withCredentials请求
上面说到，CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段。
```text
Access-Control-Allow-Credentials: true
```

另一方面，开发者必须在AJAX请求中打开withCredentials属性。
```text
var xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```

否则，即使服务器同意发送Cookie，浏览器也不会发送。或者，服务器要求设置Cookie，浏览器也不会处理。

但是，如果省略withCredentials设置，有的浏览器还是会一起发送Cookie。这时，可以显式关闭withCredentials。
```text
xhr.withCredentials = false;
```

需要注意的是，如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。


#### 3.5 非简单请求
##### **预检请求**

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。
非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。

浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

下面是一段浏览器的JavaScript脚本。
```text
var url = 'http://api.alice.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```

上面代码中，HTTP请求的方法是PUT，并且发送一个自定义头信息X-Custom-Header。

浏览器发现，这是一个非简单请求，就自动发出一个"预检"请求，要求服务器确认可以这样请求。下面是这个"预检"请求的HTTP头信息。
```text
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

"预检"请求用的请求方法是OPTIONS，表示这个请求是用来询问的。头信息里面，关键字段是Origin，表示请求来自哪个源。

除了Origin字段，"预检"请求的头信息包括两个特殊字段
```text
(1) Access-Control-Request-Method
该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。

(2) Access-Control-Request-Headers
该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是X-Custom-Header。
```

##### **预检请求的回应**

服务器收到"预检"请求以后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨源请求，就可以做出回应。
```text
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

上面的HTTP回应中，关键的是Access-Control-Allow-Origin字段，表示 http://api.bob.com 可以请求数据。该字段也可以设为星号，表示同意任意跨源请求。
```text
Access-Control-Allow-Origin: *
```

如果服务器否定了"预检"请求，会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被XMLHttpRequest对象的onerror回调函数捕获。控制台会打印出如下的报错信息。
```text
XMLHttpRequest cannot load http://api.alice.com.
Origin http://api.bob.com is not allowed by Access-Control-Allow-Origin.
```

服务器回应的其他CORS相关字段如下。
```text
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

#### 3.6 字段说明
###### 1.Access-Control-Allow-Origin
```text
首先，客户端请求时要带上一个Origin，用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。然后服务端在返回时需要带上这个字段，并把对方传过来的值返回去。告知客户端，允许这次请求。
这个字段也可以设置为*，即允许所有客户端访问。但是这样做会和Access-Control-Allow-Credentials 起冲突。可能导致跨域请求失败。
```

###### 2.Access-Control-Allow-Credentials
```text
这个字段是一个BOOL值，可以允许客户端携带一些校验信息，比如cookie等。如果设置为Access-Control-Allow-Origin：*，而该字段是true，并且客户端开启了withCredentials, 仍然不能正确访问。需要把Access-Control-Allow-Origin的值设置为客户端传过来的值。
```

###### 3.Access-Control-Allow-Credentials
```text
该字段与简单请求时的含义相同。
```

** 4.Access-Control-Max-Age **
```text
该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求。
```

#### 3.7 浏览器的正常请求和回应
一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段。服务器的回应，也都会有一个Access-Control-Allow-Origin头信息字段。

下面是"预检"请求之后，浏览器的正常CORS请求。
```text
PUT /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
X-Custom-Header: value
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
上面头信息的Origin字段是浏览器自动添加的。

下面是服务器正常的回应。
```text
Access-Control-Allow-Origin: http://api.bob.com
Content-Type: text/html; charset=utf-8
```


#### 3.8 CORS与JSONP比较
CORS与JSONP的使用目的相同，但是比JSONP更强大。

JSONP只支持GET请求，CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，
以及可以向不支持CORS的网站请求数据。



### 4 开启中间件进行跨域
#### 4.1 安装cors包
	go get -u -v github.com/gin-contrib/cors

#### 4.2 配置cors跨域

** gin原生代码实现cors **
```go
package main
import (
    "github.com/gin-gonic/gin"
    "strings"
    "fmt"
    "net/http"
)

func main() {
        r := gin.Default()
        //开启中间件 允许使用跨域请求。
        //注意：cors中间件一定要在路由中间件之前注册，否则cors跨域失效。
        r.Use(Cors())
        r.run()
}

func Cors() gin.HandlerFunc {
    return func(c *gin.Context) {
        method := c.Request.Method
        origin := c.Request.Header.Get("Origin") //请求头部
        if origin != "" {
            //接收客户端发送的origin （重要！）
            c.Writer.Header().Set("Access-Control-Allow-Origin", origin)
            //服务器支持的所有跨域请求的方法
            c.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE,UPDATE")
            //允许跨域设置可以返回其他子段，可以自定义字段
            c.Header("Access-Control-Allow-Headers", "Authorization, Content-Length, X-CSRF-Token, Token,session")
            // 允许浏览器（客户端）可以解析的头部 （重要）
            c.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers")
            //设置缓存时间
            c.Header("Access-Control-Max-Age", "172800")
            //允许客户端传递校验信息比如 cookie (重要)
            c.Header("Access-Control-Allow-Credentials", "true")
        }

        //允许类型校验
        if method == "OPTIONS" {
            c.JSON(http.StatusOK, "ok!")
        }

        defer func() {
            if err := recover(); err != nil {
                log.Printf("Panic info is: %v", err)
            }
        }()

        c.Next()
    }
}

```

** 处理跨域请求,支持options访问 **
```go
func Cors() gin.HandlerFunc {
	return func(c *gin.Context) {
		method := c.Request.Method

		c.Header("Access-Control-Allow-Origin", "*")
		c.Header("Access-Control-Allow-Headers", "Content-Type,AccessToken,X-CSRF-Token, Authorization, Token")
		c.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS")
		c.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers, Content-Type")
		c.Header("Access-Control-Allow-Credentials", "true")

		//放行所有OPTIONS方法
		if method == "OPTIONS" {
			c.AbortWithStatus(http.StatusNoContent)
		}
		// 处理请求
		c.Next()
	}
}
```

** 使用gin的cors包 **
```go
package main

import (
	"github.com/gin-contrib/cors"
	"github.com/gin-gonic/gin"
	"time"
)

func main() {
	router := gin.Default()
	// CORS for https://foo.com and https://github.com origins, allowing:
	// - PUT and PATCH methods
	// - Origin header
	// - Credentials share
	// - Preflight requests cached for 12 hours
	router.Use(cors.New(cors.Config{
		AllowOrigins:     []string{"https://foo.com"},
		AllowMethods:     []string{"PUT", "PATCH"},
		AllowHeaders:     []string{"Origin"},
		ExposeHeaders:    []string{"Content-Length"},
		AllowCredentials: true,
		AllowOriginFunc: func(origin string) bool {
			return origin == "https://github.com"
		},
		MaxAge: 12 * time.Hour,
	}))
	router.Run()
}
```

#### 参考
* [跨域共享CORS详解及Gin配置跨域](https://www.cnblogs.com/you-men/p/14054348.html)
* [HTTP访问控制(CORS),解决跨域问题](https://blog.csdn.net/thc1987/article/details/54572272)
* [什么是跨域？跨域解决方法](https://blog.csdn.net/qq_38128179/article/details/84956552)
* [Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
* [Fetch Standard](https://fetch.spec.whatwg.org)
