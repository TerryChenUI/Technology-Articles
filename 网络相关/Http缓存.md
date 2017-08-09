# Http缓存
客户端和服务端通信，

## 与缓存相关的HTTP首部字段
1. 通用首部字段
![](https://github.com/TerryChenUI/Technology-Articles/blob/master/%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3/images/Http%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E5%92%8C%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87-1.jpg)

2. 请求首部字段
![](https://github.com/TerryChenUI/Technology-Articles/blob/master/%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3/images/Http%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E5%92%8C%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87-1.jpg)

3. 响应首部字段
![](https://github.com/TerryChenUI/Technology-Articles/blob/master/%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3/images/Http%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E5%92%8C%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87-1.jpg)

4. 实体首部字段
![](https://github.com/TerryChenUI/Technology-Articles/blob/master/%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3/images/Http%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E5%92%8C%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87-1.jpg)

## 缓存头部对比
|头部 |	优势和特点 | 劣势和问题
|-----|-----|-----|
|Expires| 1、HTTP 1.0 产物，可以在HTTP 1.0和1.1中使用，简单易用。2、以时刻标识失效时间。| 1、时间是由服务器发送的(UTC)，如果服务器时间和客户端时间存在不一致，可能会出现问题。2、存在版本问题，到期之前的修改客户端是不可知的。
|Cache-Control| 1、HTTP 1.1 产物，以时间间隔标识失效时间，解决了Expires服务器和客户端相对时间的问题。2、比Expires多了很多选项设置。| 1、HTTP 1.1 才有的内容，不适用于HTTP 1.0 。2、存在版本问题，到期之前的修改客户端是不可知的。
|Last-Modified|1、不存在版本问题，每次请求都会去服务器进行校验。服务器对比最后修改时间如果相同则返回304，不同返回200以及资源内容。| 1、只要资源修改，无论内容是否发生实质性的变化，都会将该资源返回客户端。例如周期性重写，这种情况下该资源包含的数据实际上一样的。2、以时刻作为标识，无法识别一秒内进行多次修改的情况。3、某些服务器不能精确的得到文件的最后修改时间。
|ETag|1、可以更加精确的判断资源是否被修改，可以识别一秒内多次修改的情况。2、不存在版本问题，每次请求都回去服务器进行校验。| 1、计算ETag值需要性能损耗。2、分布式服务器存储的情况下，计算ETag的算法如果不一样，会导致浏览器从一台服务器上获得页面内容后到另外一台服务器上进行验证时发现ETag不匹配的情况。

## 缓存机制

缓存判断的流程:
1. Expires / Cache-Control / 200(from cache)
2. Last-Modified (response header) / If-Modified-Since (request header) / 304(nochange)
3. ETag (response header) / If-None-Match (request header) / 304(no change)

![](https://github.com/TerryChenUI/Technology-Articles/blob/master/%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3/images/Http%E8%AF%B7%E6%B1%82%E6%8A%A5%E6%96%87%E5%92%8C%E5%93%8D%E5%BA%94%E6%8A%A5%E6%96%87-1.jpg)

## 缓存实践
1. Expires / Cache-Control
Expires用时刻来标识失效时间，不免收到时间同步的影响，而Cache-Control使用时间间隔很好的解决了这个问题。 但是 Cache-Control 是 HTTP1.1 才有的，不适用于 HTTP1.0，而 Expires 既适用于 HTTP1.0，也适用于 HTTP1.1，所以说在大多数情况下同时发送这两个头会是一个更好的选择，当客户端两种头都能解析的时候，会优先使用 Cache-Control。

2. Last-Modified / ETag
二者都是通过某个标识值来请求资源， 如果服务器端的资源没有变化，则自动返回 HTTP 304 （Not Changed）状态码，内容为空，这样就节省了传输数据量。而当资源发生比那话后，返回和第一次请求时类似。从而保证不向客户端重复发出资源，也保证当服务器有变化时，客户端能够得到最新的资源。其中Last-Modified使用文件最后修改作为文件标识值，它无法处理文件一秒内多次修改的情况，而且只要文件修改了哪怕文件实质内容没有修改，也会重新返回资源内容；ETag作为“被请求变量的实体值”，其完全可以解决Last-Modified头部的问题，但是其计算过程需要耗费服务器资源。

3. from-cache / 304
Expires和Cache-Control都有一个问题就是服务端作为的修改，如果还在缓存时效里，那么客户端是不会去请求服务端资源的（非刷新），这就存在一个资源版本不符的问题，而强制刷新一定会发起HTTP请求并返回资源内容，无论该内容在这段时间内是否修改过；而Last-Modified和Etag每次请求资源都会发起请求，哪怕是很久都不会有修改的资源，都至少有一次请求响应的消耗。
对于所有可缓存资源，指定一个Expires或Cache-Control max-age以及一个Last-Modified或ETag至关重要。同时使用前者和后者可以很好的相互适应。
前者不需要每次都发起一次请求来校验资源时效性，后者保证当资源未出现修改的时候不需要重新发送该资源。而在用户的不同刷新页面行为中，二者的结合也能很好的利用HTTP缓存控制特性，无论是在地址栏输入URI然后输入回车进行访问，还是点击刷新按钮，浏览器都能充分利用缓存内容，避免进行不必要的请求与数据传输。

## 结论
* 需要兼容HTTP1.0的时候需要使用Expires，不然可以考虑直接使用Cache-Control。
* 需要处理一秒内多次修改的情况，或者其他Last-Modified处理不了的情况，才使用ETag，否则使用Last-Modified。
* 对于所有可缓存资源，需要指定一个Expires或Cache-Control，同时指定Last-Modified或者Etag。
* 可以通过标识文件版本名、加长缓存时间的方式来减少304响应。


## 相关参考
* [HTTP缓存控制小结](http://imweb.io/topic/5795dcb6fb312541492eda8c)
* [浅谈Web缓存](http://www.alloyteam.com/2016/03/discussion-on-web-caching/)
* [浅谈浏览器http的缓存机制](http://www.cnblogs.com/vajoy/p/5341664.html)