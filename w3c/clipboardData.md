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
这涉及到FileAPI，可以去[这里]()看。话说回来，这个接口目前即使传入File对象，剪切板的中设置的依然是string，内容为文件名称。





