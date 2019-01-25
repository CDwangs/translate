# [译]从输入URL到页面呈现的超详细过程——第一步:获取资源


>**原文链接：[From URL to Interactive](https://alistapart.com/article/from-url-to-interactive)<br>
>原文作者：[agustafson](https://alistapart.com/author/agustafson)<br>
>译者：[wangds](https://juejin.im/user/590d7963a0bb9f00588d8d04)**<br>

这是一个系列文章，分为四个部分介绍了从输入URL到页面呈现的详细过程
* **客户端从服务器获取资源（Server to Client）**
* **标签转化成DOM的过程（tags to DOM）**
* **CSS解析的过程（braces to pixels）**
* **编译执行javascript的过程（var to JIT）**

本篇翻译的是第一部分：Server to Client

每当我们在地址栏中输入URL或者点击一个链接的时候，实质就是对浏览器（客户端）下达一道指令去指定的位置（服务器）获得我们想要的资源。获取资源是浏览器展示web页面的第一步工作。

获取资源是一个过程，这个过程可以拆解成如下几个步骤。

#### 1.浏览器修正传输协议（CHECK FOR HSTS）

当浏览器接收到URL后，浏览器首先要修正URL的传输协议（http or https)。之所以采用修正这个词，是因为浏览器中有两个列表：[预加载 HSTS列表](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)和浏览器曾经访问过的HSTS列表，凡是在这两个列表内的地址，浏览器都会将传输协议改成HTTPS。举个例子，即使你在浏览器里输入了http://www.bing.com这个地址，最后访问的都是https://www.bing.com。

#### 2.浏览器检查服务工作线程（CHECK FOR SERVICE WORKERS）

接下来，浏览器要确认[服务工作线程](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API)是否能处理这个请求。服务工作线程是浏览器中相对较新的功能，主要的能力之一就是支持离线访问。服务工作线程通过拦截并代理请求的方式，从脚本控制的缓存内获得请求的资源，从而完成了网站离线访问得工作。

用户首次访问服务工作线程控制的网站或页面时，服务工作线程会立刻被下载（之后至少每24小时它会被下载一次）。下载完成后服务工作线程将依次进行安装和激活。如果网站有一个服务工作线程处于激活状态，新下载的服务工作线程在安装完成后不会立即激活而是处于等待状态。只有当网站不依赖旧的服务工作线程后，新的服务工作线程才会激活。服务工作线程是完全异步的。

如果没有服务工作线程，浏览器将会访问网络层，直接进行下一步工作。

#### 3.浏览器检查网络缓存（CHECK THE NETWORK CACHE）
在这一步中，浏览器首先确定本次请求的资源是否已经存在于自己的缓存中。如果浏览器是第一访问这个网站，肯定没有缓存，然后浏览器将发送请求至服务器，在请求头中添加Cache-Control这个字段用来制定本次请求关于缓存的规则。[Cache-Control](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)常见的取值有max-age（缓存内容失效时限）、no-store（是否开启缓存）、no-cache（使用缓存时是否需要与服务器验证）等。每次从浏览器的缓存中使用设置了no-cache资源前，都要发送一个验证请求到服务器，验证缓存中的资源是否是最新的。这个验证缓存的请求头中包含[If-Modified-Since（时间判定）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-Modified-Since)或者包含[If-None-Match（资源版本号）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-None-Match)，服务器接收到请求后根据修改时间或者最新版本号来断定浏览器中的资源是不是最新的，如果浏览器中的资源是最新的则响应HTTP 304,告知浏览器继续使用之前的缓存，不是最新的则返回HTTP 200，同时响应最新的资源给浏览器，更新缓存。

#### 4.浏览器检测连接（CHECK FOR CONNECTION）
这一步的工作是解析DNS获取正确的IP地址，解析域名的第一步是浏览器查找本机硬盘HOST文件是否有对应的IP地址，如果本机硬盘HOST文件内没有，浏览器会发送一个DNS请求到本地DNS服务器（Internet服务提供商托管，中国的电信联通等），本地DNS服务器会首先查找自己的缓存记录，如果没有将继续发送请求至DNS根服务器，逐级查找到正确的IP地址。

有些时候，浏览器可以预先知道将要访问哪些资源，提前与这些资源进行连接。方法就是在页面的link标签内添加[Resource Hint(资源暗示)](https://www.w3.org/TR/resource-hints/)来告知浏览器哪些资源需要预先连接加载。Resource Hint有DNS Prefetch、Preconnect、Prefetch和Prerender四种。举个应用例子，当我们在搜索引擎中搜索东西的时候，结果页面上会展示大量的相关链接。浏览器可以使用Resource Hint将最靠前的搜索结果进行预连接和预下载，这样当我们点击这些靠前链接的时候，打开速度将更快。

#### 5.浏览器与服务器建立链接（ESTABLISH CONNECTION）
到了这一步，客户端（也就是浏览器）已经正式与服务器建立了连接。如果我们使用[TLS](https://developer.mozilla.org/en-US/docs/Web/Security/Transport_Layer_Security)，则需要执行TLS握手来验证服务器提供的证书。

#### 6.浏览器发送请求到服务器（SEND THE REQUEST TO THE SERVER）
通常来说，浏览器发给服务器的第一个请求是最基础的页面请求。服务器会将一个HTML文件发送给浏览器。

#### 7.浏览器处理响应（HANDLE THE RESPONSE）
浏览器接收到来自服务器的响应资源后，将对这些响应资源进行分析。首先，浏览器将查看[响应头](https://developer.mozilla.org/zh-CN/docs/Glossary/Response_header),响应头是以键值对的形式存在的。如果浏览器查看响应头表示需要重定向（例如采用 Location header的方式），浏览器将拿到重定向的地址后，将重新从第一步修正传输协议（CHECK FOR HSTS）开始工作。

如果服务器的响应资源进行了压缩，浏览器还需要对这些资源进行解压。

另外，浏览器开始解析响应资源的时候，同时开始对这些资源进行缓存。

接下来，浏览器通过判断响应内资源的MIME类型确定用什么样的方式来加载资源。例如，在浏览器解析HTML的时候，碰到图像文件就以image对象的方式加载。另外浏览器在解析HTML时（未生成render树），还会继续下载响应内的资源。这一部分的知识将在下篇文章内详细讲解。

至此，浏览器将会这次访问的URL放入浏览器的历史记录里，我们可以通过浏览器的前进和后退功能访问他们。

如下的流程图的展示了1-7的步骤内容：

![流程图](https://alistapart.com/d/server-to-client/Flowchart_A_List_Apart_696w.png)

正如我们了解的一样，还有很多后续需要请求的资源，例如图片资源、css资源、JavaScript资源。以及css资源内又包含的图片或者使用import()，AJAX引用的其他资源等等。这些全部的资源才构造了具有交互性的页面。我们所有请求的资源都遵循上面的流程图，当然也遵循浏览器的缓存策略。

#### 细聊浏览器缓存（Caching）

前文已经提到，浏览器管理着[HTTP缓存](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching_FAQ),缓存中的资源用来快速展示我们曾经访问过的网站。缓存中储藏着类似于网站图标、基础JavaScript文件等不经常变动的资源。合理的使用缓存既可以减少网络请求的数量又可以加快页面的载入速度。

另外，HTTP缓存也是有限度的。浏览器通过响应头内的[Cache-Control](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)来判断需要缓存哪些资源和缓存资源的时限。比如浏览器可以通过Cache-Control: no-store来判断本次请求的资源是不需要缓存的，这种不缓存的内容一般是指需要频繁变动的资源。又比如Cache-Control: immutable用来断定本次请求的资源是永久不变的。在使用Cache-Control: immutable的时候，作者建议用不同的URL来对应同一资源的不同版本，以便浏览器缓存的资源不受影响。

当然，浏览器中不仅仅有HTTP这一种缓存。我们可以通过JavaScript进行程序化缓存。这就是我们第二步中提到的[服务工作线程](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API),服务工作线程可以拦截请求并返回服务工作线程缓存的资源。服务工作线程使网站的缓存更具有灵活性。另外，每个网站的程序化缓存都是独立，这些缓存与网站是一一对应的关系。

#### 源的概念（Origin model）

同源指的是通信协议、域名、端口完全相同。举个例子，https://www.bing.com:443 中https是通信协议，www.bing.com是域名，443是端口。https://www.bing.com:443 和 http://www.bing.com:80 就是不同源的。

同源是浏览器中非常重要的一个概念，隔离开来的数据变得更加安全。在大多数情况下，浏览器为了安全考虑都采用同源策略。在上面的两个例子中https://www.bing.com:443 和 http://www.bing.com:80 都无法查看对方的缓存。

假如bing.com想调用一个microsoft.com的JavaScript资源，正常情况下浏览器会因为同源策略而不允许。这个时候可以采用[CORS (Cross-Origin Resource Sharing)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)方法来进行跨域访问。microsoft.com服务器将进行一个声明表示bing.com可以访问哪些资源。

#### 结束语

以上就包含了资源从服务器到客户端的全部内容。下一篇文章我们将聊聊HTML标签是如何转化成DOM的。

限于本人水平有限，文中如有错误感谢您指正。




