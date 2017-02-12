#响应式函数设计
##什么是响应式的程序？
这个问题有偏重应用的答案和偏重理论的答案。作为一个码农屌丝，能告诉大家的只有响应式编程到底长啥样，怎么用。
根据Wikipedia的[解释](https://en.wikipedia.org/wiki/Reactive_programming)，响应式编程是一种编程泛型，其研究领域围绕如何构建数据流以及繁衍数据变化（oriented around data flows and the propagation of change）。

这不是一个难懂的解释。在我们的代码中，一个事件监听器，应该就可以看做响应式的，例如：

```javascript
document.getElementById('input').addEventListener('change',e=>handle);
```
这就是对input框的change事件的监听，其本质就是响应式的。

既然响应式这么简单，还有什么好讲的呢？

可以这样问，但！在应用上，总有人将一个简单的事件监听玩出花儿来，一个语法糖不可怕，多了就出乎人的意料了。

我们引用most.js的代码，来看看这个响应式库完成了什么优雅的api让你觉得眼前一亮。


```javascript
import * as most from 'most'
most.fromEvent(inputNode, 'input')
    .map(e=>e.target.value)
    .observe(console.info);
```
上面的代码不足为奇，监听了input框的input事件，然后打印（console.info）出来。

不用most.js也可以实现，例如：

```javascript
inputNode.addEventListener('input', getValueAndPrint);

function getValueAndPrint(e){
    var value = e.target.value;
    console.info(value);
}
```

如果想让得到的`value`再做其他处理呢？比如全部变成大写，且保证输入不能是数字。是不是要在`getValueAndPrint`方法里面在添加新的方法调用？

一种稍微优雅点的解决方案是用函数组合：
```javascript
inputNode.addEventListener('input', _.compose(console.info, setInputValue, filterNumber, toUpperCase, getValue));
//_.compose的作用去看underscore.js的api吧。
//其余各方法的实现方法略，可以想象得到
```
是不是灵活很多？当然还有更优雅的api。那就是引入数据流的概念，一个数据流是一个对象，你可以操作它，生成一个新的流，或者和另一个流合并，就像两条合流交汇一样。

我们看看most.js来实现上面的例子。

```javascript
import * as most from 'most'
most.fromEvent(inputNode, 'input')
    .map(e=>e.target.value)   //获取value
    .map(value=>value.toUpperCase())  //变成大写
    .map(value=>value.replace(/\d/g,'')) //过滤掉数字
    .tap(setInputValue)   // 设置input框的值
    .observe(console.info);  //打印
```

第一句代码就生成了一个流

```javascript
most.fromEvent(inputNode, 'input')
```
然后接下来的map函数将这个流转换为一个新的流，新的数据流中的数据发生了变化。我们不断的map这个数据流，其中的数据不断变化，直到最后得到我们想要的数据流，这个数据流中流淌的是全部是大写的没有数字的输入内容。

用一个抽象的图来表示：

```segment
----{event}-----   //原始的数据流
----name123-----   //第一次map后
----NAME123-----   //第二次map后
----NAME--------   //第三次map后
```

最后还是回到最初的定义上：**响应式编程是一种编程泛型，其研究领域围绕如何构建数据流以及繁衍数据变化（oriented around data flows and the propagation of change）**

我们的事件监听只能算作最原始的数据流，响应式编程要做到的，是构建更加复杂当然也更加灵活的数据流，并且让其中流淌的数据更好的**“繁衍”**，两条数据流可以按各种方式合并，一条数据流也可以截断，重新开始等。就像是科幻电影中一样，我们手中的东西真的有绿色的字母在流动。


##回归现实，我们是怎么实现一条数据流的？
这里就是乏味的代码实现阶段。 : (

most.js是我接触响应式的起点，且这个库算是很轻量的了。这里就以most.js为例简要阐明一条数据流的原生实现。

###Functor
要实现一条数据流，你可以选择不同的编程泛型（命令式、面向对象，函数式等等）。函数式的编程泛型是声明式的，一开始难以理解，但是一旦掌握其规律就能达到知其一便知其百的功效。这里我们选用函数式的编程泛型来实现一条数据流。

首先是数据流的构造函数：

```javascript
const Stream = function Stream(source){
    this.__value = source;
}
```

就这样！！！一个数据流除了把数据塞进自己什么也不做，没错吧？

那么装进Stream的source是什么呢？那就要看流淌的数据是什么了。

我们来实现一个较为实用的数据流，也最容易阐释一个数据流所包含的元素。这条数据流的描述如下：

> 设计一条数据流，这个数据流每一秒产生一个1并发射出来，就像这样， start:-1-1-1-1-1-1-1-1

用setInterval实现的方案只是一个最简单的数据流，它不能“繁衍”。这在第一小节就说过了。

我们再写一个构造方法，`Periodic`将它的实例放进Stream中，这样数据流中装的就是一个Periodic对象了。


```javascript
function Periodic(duration, value){
    this.duration = duration;
    this.value = value;
}

var periodicStream = new Stream(new Periodic(1000, 1));
```

现在我们已经有了一个每1秒发射一个数字1的数据流了。

外部的api搭建好后，最重要的两个问题来了：

1. 我们怎么控制Periodic周期性的发射数据？
2. 怎样让Periodic具备繁衍的能力？

对于第一个问题，我们要实现一个自己的时序机制。对于第二个问题，要定义一系列map函数的衍生函数，只要这些函数都返回一个新的Stream对象就可以了。

###时序控制

我们知道js上只有两个时序控制接口：setTimeout和setInterval。都不是精确的时序控制。没错，要在js上做到完全精确的时序控制是不可能的，但我们可以做到尽最小偏差可能的时序控制。

###繁衍









