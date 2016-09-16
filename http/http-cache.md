#与缓存相关的报文头
###request头
① 当浏览器请求一个资源时，不希望缓存此资源，会发送以下请求头：(如果你chrome打开开发者工具，并且设定不要缓存时，你的每一条http请求都会带上下面两条。注意和max-age=0的区别)

```
Cache-Control:no-cache
Progma:no-cache  //这条是为了支持HTTP1.0
```

② 当浏览器请求一个资源，并表示不需要缓存服务器过问，而是直接向服务器验证时，会发送以下请求头：

```
Cache-Control:max-age=0
```
③ 当浏览器请求一个资源，且自己有一份该资源的缓存时，会这样验证自己已有缓存的新鲜度（有没有过期）：

```
If-Modified-Since: Mon, 19 Nov 2015 08:38:01 GMT
If-None-Match: "0693f67a67cc1:0"
```
###response头
① 当服务器不希望浏览器缓存这份资源时：

```
Cache-Control:no-cache  //提醒浏览器尽量不要缓存这个资源，下次还要从服务器请求它
Cache-Control:no-store  //禁止服务器缓存它
```
② 当服务器希望浏览器缓存这个资源时：

```
Date:Mon, 19 Nov 2015 08:38:01 GMT  //这份资源是我2015-11-19 08:38:01发给你的
Cache-Control:max-age=60  //这份资源你缓存60秒，60秒后过期（相对时间）
Expire:Mon, 19 Nov 2015 08:39:01 GMT  //这份资源这个时候过期（绝对时间）
Last-Modified:Mon, 19 Nov 2014 08:39:01 GMT //这份资源最后修改日期是在2014年~
ETag:"0693f67a67cc1:0"  //这份资源的验证码，你留着有用
```
约定一下，以下内容，如果浏览器发送包含内容①的请求头，我会说“请求①”，同理，请求②③，返回200+②之类的。如果浏览器请求头不包含以上任何头，我会直接说“请求”，同理，直接说“返回200”。

#浏览器缓存策略（chrome）（重要的啦！测了好多种情况总结出来哒~）
基于上面的论述，来说说浏览器缓存资源的策略，
- 只有当返回200+②时，浏览器会缓存资源，除此之外，浏览器不会缓存发来的资源。
- 如果浏览器已经缓存了资源，但新接收到的返回是200不加②，那么已经缓存的资源会被清除掉。
- 如果一个资源已经被缓存了，那么下次请求这个资源的时候会请求③。
- 只要返回304，那么浏览器会在缓存中寻找该资源，找到了即使用，找不到无可奈何。
- 无论如何，只要返回②，浏览器都会更新缓存时间。