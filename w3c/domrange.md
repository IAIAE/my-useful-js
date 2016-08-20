#DOM Range
###边界点
一个range有两个**边界点**(boudary-points)，每个边界点用node和offset来表示。
node是边界点的容器，它也就对应DOM中的node，可以是Element\Comment\ProcessingInstruction\EntityReference\CDATASection\Document\DocumentFragment\Attr\Text类型的节点。但是边界点的node不能是DocumentType\Entity\Notation类型。






















一个range的每个边界点都有一个container（node），这些node的父元素也是这个range的container，这样，两个边界点必定有一个共有的container，我们叫做这个range的根container。

如果一个range的两个边界点有相同的node和offset，那么，我们称这个range为collapsed Range。这常常出现在插入光标的情况。

###selection和partial selection（选中和部分选中）
如果一个node或者16位字符位于range之中（怎样才位于range之中，想想就知道了）。那么我们说这个node或字符被选中了（selected）。上图中的例子，range2选中了P节点，但是range4没有选中那个文本节点。

我们说一个节点被部分选中了，如果这个节点是并且只是一个边界点的Container。

##创建一个range

	var range = document.createRange();
	
创建的range的container是document，offset是0，是一个collapsed range。
**获取边界点信息：**

	range.startContainer;
	range.startOffset;
**设定边界点信息：**

	range.setStart(node,offset)
注意node是dom类型。offset不能超出实际下标，不然报错IndexSizeError: DOM Exception xx;
**还有几个常用的设定边界点的接口：**

	range.setStartBefore(node)
	range.setStartAfter(node)
**合并range：**
	
	range.collapse();
**判断range是否是合并：**

	range.collapsed;  //只读
**设定一个range选中某节点或节点内容：**

	range.selectNode(node);
	range.selectNodeContent(node);

##判断两个边界点的位置关系

首先这样判断连个边界点的位置关系:

	range1.compareBoundaryPoints(Range.START_TO_START,range2);
这样就是range1的起始点和range2的起始点比较。
返回值为-1,0,1，分别代表小于、等于、大于的情况。
然后：比较两个边界点的算法如下：

1. 如果A点和B点有同一个container，比较他们的offset
2. 如果A点的container是B点的祖先container。比较A的offset和B的祖先container的位置。
3. 同第2点。B的情况
4. 如果以上都不是，A的container和B的container是兄弟节点或者是旁系血亲。那么比较A的container和Bcontainer在先序遍历中的先后关系。

##删除range中的内容

	range.deleteContents();

删除range中的内容后，如果一个node被range“选中”，那么这个node被删除掉；如果这个node被“部分选中”，只删除node中被选中的部分。

例如:
	
	<div>hi <p>hel|lo </p> wor|ld</div>  //两条竖线代表边界点
	<div>hi <p>hel</p>|ld</div>   //删除后的结果，range合并于竖线的地方

关于合并点：

1. 如果没有node被部分选中，直接删除，range合并于start位置
2. 如果有node被部分选中了，range删除后合并于元素之后或者之前。

##抽出内容

提取内容:
	
	range.extractContents();

提取的内容会暂时从dom节点中删除。方法返回的内容为documentFragment。
需要注意的是：如果一个node被选中，这个节点从dom中删除，直接放进返回的documentFragment中，如果一个node被部分选中，或克隆这个节点放进documentFragment中。
一定要注意的是，克隆一个节点的时候会克隆所有的attr属性，比如ID。

##克隆内容

	range.cloneContents();
	
##插入内容
	range.insertNode(node);
在range的start处插入node，同时node也可以是一个documentFragment，例如：
	
	range.insertNode(range1.extrctContents());

##包裹内容

	range.surroundContents(node);
如果range部分选择了一个非文本节点，那么此方法会抛出一个DOM异常。


##克隆range
	range.cloneRange();

###获取根container
	range.commonAncestorContainer  //只读
###获取所有文本内容
	range.toString();  //返回的内容类型为"DOMString"


##文档变化后的range情况
让文档中的节点发生了变化，那么关联的range会自动更新。
###插入
如果一个节点插入在range之前，那么range的startOffset和endOffset会发生变化，container不会变化。
如果一个节点插入的位置恰好是range的一个边界点的位置。w3c标准采取的策略是：尽量不要让range的container和offset变动。那么，在边界点的位置插入一个节点时候，会按情况选择插入range里面还是外面。看下面的例子：

	<div>|wor|ld</div>   
	//然后在start的位置插入hello，结果是
	<div>|hello wor|ld</div>   //这样保证了startOffset一直是0不变
	//或者在end的位置插入pew!，结果是
	<div>|wor|pre!ld</div>   //这样保证了start和end的位置都没变

###删除节点
删除一个节点后range的变化情况，可以想象成要删除的范围range2.deleteContents()后这个range1怎么变化。

1. 如果range1的边界点在range2之内，那么range2.deleteContents()之后，range2.会合并，那么range1的这个边界点也会合并到和range2同一点上。
2. 如果range1的边界点在range2之后，那么range2删除后可能会更新range1这个边界点的offset。
3. 如果range1的边界点在range2之前，那么range2的删除对range1的这个边界点没有影响。













