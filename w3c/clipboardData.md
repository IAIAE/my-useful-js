###native环境向web端的粘贴
方法：

可行性：


###web端向native环境的粘贴
方法：

可行性：


###网络资源（图片、附件）在各端之间粘贴传输办法
方法：

可行性：


###w3c提供的操作二进制文件相关接口：

- Ajax传输二进制文件，结果可为：ArrayBuffer\Blob\DOMString\Document\Text(这里添加链接)，其中ArrayBuffer和Blob为二进制格式。
- ArrayBuffer\Blob\File\DataUrl相互之间可以转换。


###js脚本中设置剪贴板的两种办法：

- ``e.clipboardData.setData(plain_unicode_string, format)``，只能设置字符型数据。
- （未实现）``e.clipboardData.items.add(File, fileType, options)``，可以设置file类型的数据。如果实现了，理论上浏览器端可以将Ajax获取到的文件保存在本地剪贴板中。见 https://html.spec.whatwg.org/multipage/interaction.html#datatransferitemlist和http://stackoverflow.com/questions/27206268/how-should-i-according-to-w3c-put-a-canvas-in-clipboard