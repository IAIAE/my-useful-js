#操作剪切板
浏览器在一个沙箱环境中运行，但也并非绝对的沙箱。在一定限制的条件下，js脚本拥有操作系统剪切板clipboard的能力。中的来说，js只能存放和读取文本数据。

### 写剪切板

js脚本中设置剪贴板的两种办法：

- ``event.clipboardData.setData(format, plain_unicode_string)``，只能设置字符型数据。
- ``event.clipboardData.items.add(File, fileType, options)``（部分实现），可以设置file类型的数据。如果实现了，理论上浏览器端可以将Ajax获取到的文件保存在本地剪贴板中。见 https://html.spec.whatwg.org/multipage/interaction.html#datatransferitemlist和http://stackoverflow.com/questions/27206268/how-should-i-according-to-w3c-put-a-canvas-in-clipboard

第一种方法例子：

```
document.addEventListener('copy',function(event){
	var clipboard = event.clipboardData;
	clipboard.setData('text/html','<img src="http://shared.ydstatic.com/dict/v2016/160525/logo@2x.png" />');
	clipboard.setData('text/plain','hello youdao world');
});
```
每种format只能写入一条数据。也就是说，如果写入两条'text/plain'的数据，最终剪切板中内容是后写入的那条。

标准给出的说法是，利用setData只能写入plain_unicode_string，如果向第二个参数传入非字符类型数据，结果是未定义的（至少不要期望能成功，我试过了，你可以试试）。

对于写操作的数据类型format，w3c给出的建议是，支持：

- text/plain
- text/uri-list
- text/csv
- text/html
- image/svg+xml
- application/xml, text/xml
- application/json
都是字符串不是么？

第二种方法例子：

```
document.addEventListener('copy',function(event){
	var clipboard = event.clipboardData;
	clipboard.items.add(new File(['hello youdao world'], 'copy.txt', {type:'text/plain'}));
});
```
这涉及到FileAPI，可以去[这里](http://www.w3.org/TR/FileAPI/#dfn-file)看。话说回来，这个接口目前即使传入File对象，剪切板的中设置的依然是string，内容为文件名称。



### 读取剪切板

第一种方法，用setData()放在剪切板的数据用getData获取，索引是format：

```
document.addEventListener('paste', function(event){
	var clipboard = event.clipboardData;
	var html = clipboard.getData('text/html');
	var text = clipboard.getData('text/plain');
});
```
第二种方法，用items.add放进剪切板或者系统级别的剪切板用遍历item的方法获取：

```
document.addEventListener('paste',function(event){
	var items = event.clipboardData.items;
	items.forEach(function(item){
		if(item.kind === 'string'){
			item.getAsString(function(str){
				console.info(str);
			});
		}else if(item.kind === 'file'){
			var blob = item.getAsFile();
		}
	});
});
```
对于能读取的数据类型，w3c官方给出的解释是：“如果一下原生类型数据存在于剪切板中，那么js就应该将之暴露出来。”（可见，w3c给出了js能够访问剪切板的支持范围）

- text/plain
- text/uri-list
- text/csv
- text/css
- text/html
- application/xhtml+xml
- image/png
- image/jpg, image/jpeg
- image/gif
- image/svg+xml
- application/xml, text/xml
- application/javascript
- application/json
- application/octet-stream

#数据类型
``event.clipboardData``的数据类型为：DataTransfer

``clipboardData.items``的类型为：DataTransferItemList


