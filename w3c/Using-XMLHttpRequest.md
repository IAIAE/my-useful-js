#使用XMLHttpRequest
这是一篇翻译，只是提取了重要的部分。[原文链接](https://developer.mozilla.org/en/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest)

发送一个XMLHttpRequest(以下简称xhr)异步请求很简单：

```
function reqListener () {
  console.log(this.responseText);
}

var oReq = new XMLHttpRequest();
oReq.addEventListener("load", reqListener);
oReq.open("GET", "http://www.example.org/example.txt");
oReq.send();
```

## 异步请求与同步请求
默认情况下，xhr请求是异步的，可以在open方法中设置同步请求方式。

```
var xhr = new XMLHttpRequest();
xhr.open('get', url, async);  // async can be true/false, true means asynchronously, and false means synchronously.
```

## 控制返回类型
### DOM对象的解析与操作：responseXML属性
如果你的xhr请求的是一个XML文档，那么``responseXML``属性将会是一个DOM对象，你可以有以下几种方法解析和操作这个XML文档。

- 使用[XPath](https://developer.mozilla.org/en-US/docs/Web/XPath)来查找节点
- 使用[jXon](https://developer.mozilla.org/en-US/docs/Archive/JXON)将之转换为js的对象。
- 手动解析或者序列化
- 使用[XMLSerilizer](https://developer.mozilla.org/en-US/docs/Web/API/XMLSerializer)将DOM树序列化。
- 使用一些正则表达式

### 字符串格式的DOM文档
如果你没有设置``xhr.responseType = 'document'``有请求了一个html(xml)文档，那么xhr的返回将是一个纯文本的字符串。想要操作这个DOM，你的办法是：

- 构建一个document fragment。然后设置``fragment.body.innerHTML = xhr.reposneText;``，你就得到了一个DOM节点了。（这里考虑到fragment的构造函数有一定的兼容性问题，所以我想最好还是直接createElement('div')好了）；
- 或者用正则表达式得到你想要的东西。

## 处理二进制数据
有两种方法可以接收到二进制的数据。一种是老办法：

```
var oReq = new XMLHttpRequest();
oReq.open("GET", url);
// retrieve data unprocessed as a binary string
oReq.overrideMimeType("text/plain; charset=x-user-defined");
/* ... */
```
另一种是level 2 引入的新办法

```
var oReq = new XMLHttpRequest();

oReq.onload = function(e) {
  var arraybuffer = oReq.response; // not responseText
  /* ... */
}
oReq.open("GET", url);
oReq.responseType = "arraybuffer";
oReq.send();
```

## 进度
```
var req = new XMLHttpRequest();

req.addEventListener("progress", updateProgress, false);
req.addEventListener("load", transferComplete, false);
req.addEventListener("error", transferFailed, false);
req.addEventListener("abort", transferCanceled, false);

req.open();

...

// progress on transfers from the server to the client (downloads)
function updateProgress(evt) {
  if (evt.lengthComputable) {
    var percentComplete = evt.loaded / evt.total;
    ...
  } else {
    // Unable to compute progress information since the total size is unknown
  }
}

function transferComplete(evt) {
  alert("The transfer is complete.");
}

function transferFailed(evt) {
  alert("An error occurred while transferring the file.");
}

function transferCanceled(evt) {
  alert("The transfer has been canceled by the user.");
}
```



