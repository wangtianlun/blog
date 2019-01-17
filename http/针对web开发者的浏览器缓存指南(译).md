### 概要

缓存是一个非常重要同样也非常复杂的浏览器特性

在这篇文章中，我们将解释浏览器是如何利用缓存来使加载页面更快，哪些因素决定了缓存的周期，在必要的时候如何去避开缓存。

### 为什么缓存会如此重要呢？

所有的浏览器都会尝试去缓存静态资源的本地副本，以此去降低页面的加载时间以及体积最小的网络传输

不论服务是在相同的网络环境下还是在世界上其他的网络环境下，从本地缓存中取资源一定会比通过网络请求的方式要快

### 如何让浏览器缓存生效呢？

#### case1：用户之前没有访问过这个站点

浏览器针对此站点没有任何的缓存文件，所以浏览器会向站点服务去请求所有所需的资源

![image](https://img.souche.com/f2e/99227ede48680cf8f6b1b6e92bb65cbb.png)

下面是一张首次访问维基百科首页的资源下载完之后的截图，底部状态栏显示出有265kb的数据传输到了浏览器中

![image](https://img.souche.com/f2e/32f0b1092209ae7e8b1ed7d4be551a81.png)


#### case2：用户之前访问过站点

浏览器会去请求站点服务器取到html页面，然后会查看是否有对应静态资源的缓存（js, css, images）

![image](https://img.souche.com/f2e/11a5cd18f1c854ff870dc72d2628e103.png)

当我们重新刷新维基百科的页面时，就能看出因为缓存的原因与之前的请求状况不同的地方

![image](https://img.souche.com/f2e/46fb5222c46679ecc845d5bcd9a880a4.png)

此时数据的传输量已经降到了928bytes，相当于首页访问页面时的0.3%，Size这一列就显示出了我们的大多数的资源都来自于缓存

  Chrome即会在memory cache中去拉取内容，也会在disk cache中去拉取内容，因为在case1和case2中，我们还没有关闭浏览器，所以数据依然会存储在memory cache中

### 显示浏览器缓存

在chrome中，我们可以在地址栏中输入chrome://cache去查看缓存的内容，对于每一个已经缓存的文件，这里都将会显示一个页面链接，页面链接的内容包含一个更加详细的视图说明。

![image](https://img.souche.com/f2e/aa6a407b6a418f10eeabec664295921a.png)

### 浏览器是如何知道什么该缓存的

浏览器先去检查服务器所生成的http响应头信息，一般用于缓存相关的头信息有4个：

  - ETag
  - Cache-Control
  - Expires
  - Last-Modified

#### ETag

ETag（Entity Tag）是一个用于缓存校验token的字符串，它通常以文件的哈希值来表示

服务器可以添加ETag到它的响应里，浏览器接收到这个响应，那在未来的请求中（在文件过期之后），浏览器就可以根据这个字段去判断缓存中是否保留着一份过期的副本

如果通过对比hash值是相同的，那么就说明这个资源没有变化，服务器就会返回一个304状态码（Not Modified），浏览器就知道当前缓存的副本是安全的

![iamge](https://img.souche.com/f2e/befeccc51d20c53ff31f8496a18312e2.png)

注意：只有当缓存中的文件过期了，ETag才会被用在请求中


#### Cache-Control

Cache-Control头拥有许多指令，利用这些指令我们可以设置缓存的行为，过期时间，验证等，同时这些也可以组合起来一起进行设置。

##### Cache Behavior（缓存行为）

```javascript
  Cache-Control: public
```

public意味着资源可以被任意缓存（浏览器，CDN等）

```javascript
  Cache-Control: private
```

private意味着资源只能被浏览器缓存

```javascript
  Cache-Control: no-store
```

no-store意味着让浏览器总是去请求服务器以获取资源

```javascript
  Cache-Control: no-cache
```

no-cache有一点误导性，它并不是说“不要缓存”

这是在告诉浏览器去缓存这个文件，但在和服务器确认最新版本之前不要去使用它。这个验证的过程是通过ETag来完成的

这个行为一般会用在html文件中，因为浏览器总会去检查最新的html文件标识，所以这一点是讲得通的

##### Expiration

```javascript
  Cache-Control: max-age=60
```

这个指令指定了资源应该被缓存多少秒，所以max-age=60就意味着资源应该被缓存一分钟，RFC 2616推荐这里的最大值不要超过一年（max-age=31536000）

```javascript
  Cache-Control: s-max-age=60
```

这个指令仅仅是被用在中间缓存（比如CDN）

##### Validation

```javascript
  Cache-Control: must-revalidate
```

这个指令告诉缓存在使用它之前必须去验证资源的过期状态，如果已经过期就不应该被使用

##### Expires

Expires头部来源于http1.0版本，但是在当今许多站点中依然保留着

这个头部字段指定了一个日期，超过这个日期就代表资源是无效的

```javascript
  Expires: Wed, 25 Jul 2018 21:00:00 GMT
```

注意：如果已经指定了Cache-Control中的max-age指令，那浏览器就会忽略expires

### Last-Modified

Last-Modified头部也是来自http1.0版本

```javascript
  Last-Modified: Mon, 12 Dec 2016 14:45:00 GMT
```

这个字段包含了资源最后修改的日期和时间

### HTML Meta Tag

在HTML5版本之前，在html中使用元标签（meta tag）制定cache-control是一个有效的方式

```javascript
  <meta http-equiv="Cache-control" content="no-cache">
```

但在html5中已经不推荐这么做了，为什么？因为只有浏览器可以识别这个标签，而中间缓存（CDN）是识别不了的。

所以最好是都通过http头的方式去发送缓存指令

### HTTP Response

让我们来看一个简单的http响应

```javascript
  Accept-Ranges: bytes
  Cache-Control: max-age=3600
  Connection: Keep-Alive
  Content-Length: 4361
  Content-Type: image/png
  Date: Tue, 25 Jul 2017 17:26:16 GMT
  ETag: "1109-554221c5c8540"
  Expires: Tue, 25 Jul 2017 18:26:16 GMT
  Keep-Alive: timeout=5, max=93
  Last-Modified: Wed, 12 Jul 2017 17:26:05 GMT
  Server: Apache
```

- 第2行告诉我们max-age为1个小时
- 第5行告诉我们这是一个png图片资源
- 第7行告诉我们ETag将在1个小时之后去验证资源是否改变过
- 第8行，Expires头会被忽略，因为已经设置了Cache-Control: max-age=3600
- 第10行，Last-Modified头展示了图片资源的最后修改时间

### 缓存的误区

所以我们已经意识到缓存是个好东西，我们应该积极利用它

但是我们希望的是，不需要让用户每次都去刷新（Ctrl + F5）或者清空缓存才能看到我们页面的最新内容

这类缓存问题经常困扰者开发者以及用户，用户可能会看到一个错乱的页面或者是一个行为怪异的按钮，因为他们当前使用的已经是过期的静态资源文件

### 过期的文件（Stale Files）

下面这张截图描述了一个银行网站的用户与Chase Support之间的交流，可以看到是有关登陆表单出了问题

![image](https://img.souche.com/f2e/9a3aa25a87168b546e3065acfdbfa443.png)

该用户很有可能还在使用老的js文件，这就是导致点击登陆按钮是表单重置操作而不是表单提交操作

让我们来探索过期文件影响我们的另外一种情况。

假设我们在一个叫做app.min.js的文件中修复了一个bug，并且把它推送到了生产环境

在HTML中，该脚本是这个样子

```javascript
  <script src="assets/js/app.min.js">
```

我们的web服务器给这个js文件设置了max-age为1周（604800秒）的过期时间

```javascript
  Cache-Control: private, max-age=604800
```

在更新完该js文件之后，一些用户仍在反馈说那个bug依然存在。
那这里到底发生了什么呢？

- Bob两个星期以前访问了这个网站并且缓存一个有问题的app.min.js文件的副本，由于他的副本已经过期，所以浏览器回去请求服务器，去拿最新的已经修复了bug的版本文件。
- Mary在两天之前访问了该网站，同样缓存了一个有问题的app.min.js文件的副本，她的副本由于还没过期，所以浏览器还是会去使用缓存中的那个副本


在下一个章节中，我们将会利用一个叫做"cache busting"的技术来解决这些问题和情况

### Cache Busting

Cache busting会使一个资源文件失效，强制浏览器去服务器端重新获取数据。

我们可以通过改变文件名来命令浏览器去避开缓存，对于浏览器来说，这是一份完全新的资源，所以浏览器会去服务端请求最新的数据

Cache busting同样允许我们设置一个比较长的max-age值针对频繁改动的资源。Google推荐max-age被设为1年（[source](https://developers.google.com/speed/docs/insights/LeverageBrowserCaching)）

### 版本号
我们可以给文件名添加一个版本号

```javascript
  assets/js/app-v2.min.js
```

### 指纹（finerprinting）

我们可以基于文件内容添加一个指纹值

```javascript
  assets/js/app-d41d8cd98f00b204e9800998ecf8427e.min.js
```

### 拼接查询参数

我们可以在文件名的末尾拼接一个查询参数

```javascript
  assets/js/app.min.js?version=2
```

拼接查询参数的方式在和代理服务器交互时有明显的错误（[known issues](https://gtmetrix.com/remove-query-strings-from-static-resources.html)），所以这种方式一般不推荐使用


### 最佳实践

#### 推荐

  - 对于静态资源来说，使用Cache-Control和ETag头部来控制缓存的行为
  - 设置一个比较长的max-age值，以此来获得浏览器缓存带来的好处
  - 针对Cache busting，使用指纹值（或hash值）和版本号的方式

#### 不推荐

  - 使用html元标签去指定缓存的行为
  - 使用查询参数的方式实现Cache busting


### FAQ

##### 如何知道这个文件时来自于缓存中

打开web开发者工具，在chrome中，这个信息会显示在network面板中的Size列

##### 如何阻止去缓存一个文件

使用下面的响应头

```javascript
  Cache-Control: no-cache, no-store, must-revalidate
```

### 注意

从chrome 66版本开始，chrome已经删除了chrome://cache，原因可以参考[reason1](https://chromium.googlesource.com/chromium/src.git/+/6ebc11f6f6d112e4cca5251d4c0203e18cd79adc), [reason2](https://bugs.chromium.org/p/chromium/issues/detail?id=811956)