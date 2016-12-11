#React中的动画
##ReactCSSTransitionGroup基本
为一个React元素添加动画，需要在意的是React元素挂载和卸载时候的动画，以及状态改变后重绘时发生的动画。

先说前者，React元素的挂载和卸载是由React来管理的，本质上是插入或者移除DOM。我们不可能改变React插入和移除DOM的方式，只能在DOM插入和移除之后为这个DOM添加动画，以达到动画效果。

React-addon列表中ReactCSSTransitionGroup这个库替我们完成了很多事情。可以利用ReactCSSTransitionGroup来管理组件挂载、卸载时候动画。如：

```javascript
<ReactCSSTransitionGroup
	transitionName="example"
	transitionEnterTimeout={500}
	transitionLeaveTimeout={300}>
	{items}
</ReactCSSTransitionGroup>
```
上面的组件中，`{item}`被一个ReactCSSTransitionGroup组件所包裹，如果item中添加了一个元素，那么ReactCSSTransitionGroup会立即给这个dom元素添加一个class:`example-enter`，然后在下一个tick后立即再添加一个class:`example-enter-active`，最后在500ms后把这两个class都删除。

所以，我们可以在样式表中事先写好：

```css
.example-enter {
  opacity: 0.01;
}

.example-enter.example-enter-active {
  opacity: 1;
  transition: opacity 500ms ease-in;
}
```
所以，当dom被加载时候，它立即变成透明的（`opacity：0.1`），然后由于立即加入了`example-enter-active`类名，所以这个dom会在500ms的时间内完全显现。最终，当这个dom完全显示后，这些class被清除掉。这就是运用ReactCSSTransitionGroup组件制作进入和离开动画的原理。

运用ReactCSSTransitionGroup还可以定义初始化时候的动画。上面的例子中，`{item}`中的元素如果最初就存在，怎么定义动画呢？ReactCSSTransitionGroup提供一个transitionAppear接口，用来定义初始化组件的出现方式。原理是一样的，在最初组件加载时添加一个`example-appear`类名，然后立即添加`example-appear-active`类名，最后在规定的时间内消除这些类名。

上面的例子中，通过添加transitionAppear属性，最后的代码如下：


```javascript
<ReactCSSTransitionGroup
	transitionName="example"
	transitionAppear={true}
	transitionAppearTimeout={500}
	transitionEnterTimeout={500}
	transitionLeaveTimeout={300}>
	{items}
</ReactCSSTransitionGroup>
```

##优化类名
ReactCSSTransitionGroup中并不是固定了类名必须是`enter`、`enter-active`之类的。可以自己设置：

```javascript
<ReactCSSTransitionGroup
  transitionName={ {
    enter: 'enter',
    enterActive: 'enterActive',
    leave: 'leave',
    leaveActive: 'leaveActive',
    appear: 'appear',
    appearActive: 'appearActive'
  } }>
  {item}
</ReactCSSTransitionGroup>
```
有了自定义类名的接口，我们可以使用css-module来进行样式的模块化，这样，就不需要将动画样式写在全局作用域中了。


```javascript
import {itemEnter, itemEnterActive} from './index.less'

<ReactCSSTransitionGroup
  transitionName={ {
    enter: {itemEnter},
    enterActive: {itemEnterActive}
  } }>
  {item}
</ReactCSSTransitionGroup>
```

